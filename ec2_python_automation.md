# AWS EC2 Automation with Python — Beginner Walkthrough (Step‑by‑Step)

> **Goal:** From zero to a working Python script that can **create**, **start**, **stop**, **check status**, and **terminate** an EC2 instance in AWS.
>
> **You’ll use:** Python 3, AWS CLI, and the **Boto3** library (AWS SDK for Python).
>
> **Region used in examples:** `ap-south-1` (Mumbai). You can change it if needed.

---

## 0) Read me first (safety, cost, and clarity)

- **You said “boto2.”** The correct library is **`boto3`**. (Boto2 is very old.)
- Running cloud resources can **cost money**. To stay within the **Free Tier**, use a small instance type like **`t2.micro`** or **`t3.micro`** and **terminate** it after testing.
- Never paste or share your **AWS Access Key** and **Secret Key** publicly.
- Everything below is written in **simple, copy‑pasteable steps**.

---

## 1) Prerequisites

1. **AWS Account:** Create one if you don’t have it.
2. **Windows PC (PowerShell)** — since you’re using Windows.
3. **A working internet connection**.

> Optional but recommended: keep this guide open next to your terminal so you can follow along line by line.

---

## 2) Install the tools (once)

### 2.1 Install Python 3
- In **PowerShell**:
```powershell
winget install -e --id Python.Python.3
python --version
```
You should see a version like `Python 3.x.x`.

### 2.2 Create a project folder & virtual environment
```powershell
mkdir C:\aws-ec2-python
cd C:\aws-ec2-python
py -m venv .venv
. .venv\Scripts\Activate.ps1   # activate the venv
python -m pip install --upgrade pip
```

### 2.3 Install AWS CLI v2
```powershell
winget install -e --id Amazon.AWSCLI
aws --version
```
You should see something like `aws-cli/2.x.x`.

### 2.4 Install Boto3 (AWS SDK for Python)
```powershell
pip install boto3
```

> If you like, save your dependencies:
```powershell
pip freeze > requirements.txt
```

---

## 3) Create an IAM user (first-time setup only)

> You need an **IAM user** with programmatic access to use the AWS CLI and Boto3.

1. Sign in to the AWS Console → **IAM** → **Users** → **Create user**.
2. Name: `lab-admin` (or anything you like).
3. **Attach policies:** For a short lab, you can use **AdministratorAccess** (easiest). Later, switch to least‑privilege.
4. Create an **Access key** (programmatic access). **Download the `.csv`** with the Access Key ID and Secret Access Key.

> Keep keys secret. Treat them like a password.

---

## 4) Configure AWS CLI (stores your keys locally)

In **PowerShell** (with your virtual environment activated):
```powershell
aws configure
```
Enter:
- **AWS Access Key ID**: (from your CSV)
- **AWS Secret Access Key**: (from your CSV)
- **Default region name**: `ap-south-1`  (Mumbai) — or your preferred region
- **Default output format**: `json`

> This writes credentials to `C:\Users\<You>\.aws\credentials` and config to `C:\Users\<You>\.aws\config`.

**Test the identity quickly:**
```powershell
aws sts get-caller-identity
```
You should see your AWS account and user ARN.

---

## 5) Create a working folder structure

Inside `C:\aws-ec2-python`:
```
C:\aws-ec2-python
├── .venv\                 # virtual environment
├── ec2_automation.py      # you’ll create this file now
├── requirements.txt       # optional
└── (generated) ec2_state.json  # will be created by the script
```

---

## 6) Paste this single Python script (does everything)

Create and open a new file **`ec2_automation.py`** and paste the full code below:

```python
import argparse
import json
import os
import sys
from pathlib import Path

import boto3
from botocore.exceptions import ClientError

STATE_FILE = Path("ec2_state.json")
DEFAULT_REGION = os.getenv("AWS_DEFAULT_REGION", "ap-south-1")
DEFAULT_INSTANCE_TYPE = "t3.micro"  # free-tier friendly; you can also use t2.micro in many regions
DEFAULT_NAME_TAG = "python-ec2-lab"
DEFAULT_SSH_PORT = 22

# ---------- helpers ----------

def save_state(data: dict):
    with STATE_FILE.open("w", encoding="utf-8") as f:
        json.dump(data, f, indent=2)


def load_state() -> dict:
    if STATE_FILE.exists():
        with STATE_FILE.open("r", encoding="utf-8") as f:
            return json.load(f)
    return {}


def get_clients(region: str):
    ec2 = boto3.client("ec2", region_name=region)
    ssm = boto3.client("ssm", region_name=region)
    return ec2, ssm


def get_latest_al2023_ami(ssm) -> str:
    """Use SSM public parameter to fetch the latest Amazon Linux 2023 AMI (x86_64)."""
    param_name = "/aws/service/ami-amazon-linux-latest/al2023-ami-kernel-6.1-x86_64"
    resp = ssm.get_parameter(Name=param_name)
    return resp["Parameter"]["Value"]


def get_default_vpc_id(ec2) -> str:
    resp = ec2.describe_vpcs(Filters=[{"Name": "isDefault", "Values": ["true"]}])
    vpcs = resp.get("Vpcs", [])
    if not vpcs:
        raise RuntimeError("No default VPC found. Create a VPC first or specify subnet/VPC manually.")
    return vpcs[0]["VpcId"]


def get_one_default_subnet_id(ec2, vpc_id: str) -> str:
    resp = ec2.describe_subnets(Filters=[{"Name": "vpc-id", "Values": [vpc_id]}])
    subnets = resp.get("Subnets", [])
    if not subnets:
        raise RuntimeError("No subnets found in the default VPC.")
    # pick the first available subnet
    return subnets[0]["SubnetId"]


def ensure_key_pair(ec2, key_name: str) -> None:
    """Create a key pair if it doesn't exist. If it exists, reuse it (you won't get the private key again)."""
    try:
        ec2.describe_key_pairs(KeyNames=[key_name])
        print(f"Key pair '{key_name}' already exists. Reusing it.")
        return
    except ClientError as e:
        if e.response["Error"].get("Code") != "InvalidKeyPair.NotFound":
            raise
    # create new key pair
    resp = ec2.create_key_pair(KeyName=key_name)
    material = resp["KeyMaterial"]
    pem_path = Path(f"{key_name}.pem")
    pem_path.write_text(material, encoding="utf-8")
    print(f"Created key pair '{key_name}'. Private key saved to {pem_path.resolve()}.")
    print("Keep it safe. If you delete it, you cannot download it again.")


def ensure_security_group(ec2, vpc_id: str, sg_name: str, ssh_cidr: str) -> str:
    """Create (or reuse) a security group that allows SSH from your IP."""
    # Try to find existing SG by name within this VPC
    resp = ec2.describe_security_groups(
        Filters=[
            {"Name": "group-name", "Values": [sg_name]},
            {"Name": "vpc-id", "Values": [vpc_id]},
        ]
    )
    groups = resp.get("SecurityGroups", [])
    if groups:
        sg_id = groups[0]["GroupId"]
        print(f"Security Group '{sg_name}' exists: {sg_id}")
        # Try to ensure the SSH rule exists
        try:
            ec2.authorize_security_group_ingress(
                GroupId=sg_id,
                IpPermissions=[
                    {
                        "IpProtocol": "tcp",
                        "FromPort": DEFAULT_SSH_PORT,
                        "ToPort": DEFAULT_SSH_PORT,
                        "IpRanges": [{"CidrIp": ssh_cidr, "Description": "SSH from my IP"}],
                    }
                ],
            )
            print(f"Added SSH ingress for {ssh_cidr} to existing SG.")
        except ClientError as e:
            code = e.response["Error"].get("Code")
            if code in ("InvalidPermission.Duplicate", "InvalidGroup.NotFound"):
                print("SSH rule already present or group missing; continuing.")
            else:
                raise
        return sg_id

    # Create new SG
    resp = ec2.create_security_group(
        GroupName=sg_name,
        Description="SG for Python EC2 lab (SSH only)",
        VpcId=vpc_id,
    )
    sg_id = resp["GroupId"]
    # Add SSH inbound rule
    ec2.authorize_security_group_ingress(
        GroupId=sg_id,
        IpPermissions=[
            {
                "IpProtocol": "tcp",
                "FromPort": DEFAULT_SSH_PORT,
                "ToPort": DEFAULT_SSH_PORT,
                "IpRanges": [{"CidrIp": ssh_cidr, "Description": "SSH from my IP"}],
            }
        ],
    )
    print(f"Created Security Group '{sg_name}' with SSH access from {ssh_cidr} → {sg_id}")
    return sg_id


def create_instance(region: str, name: str, key_name: str, ssh_cidr: str, instance_type: str = DEFAULT_INSTANCE_TYPE):
    ec2, ssm = get_clients(region)

    # Discover defaults
    vpc_id = get_default_vpc_id(ec2)
    subnet_id = get_one_default_subnet_id(ec2, vpc_id)

    # Ensure key pair & SG
    ensure_key_pair(ec2, key_name)
    sg_id = ensure_security_group(ec2, vpc_id, f"{name}-sg", ssh_cidr)

    # Latest Amazon Linux 2023 AMI
    ami_id = get_latest_al2023_ami(ssm)
    print(f"Using AMI: {ami_id}")

    # Launch
    print("Launching EC2 instance…")
    resp = ec2.run_instances(
        ImageId=ami_id,
        InstanceType=instance_type,
        KeyName=key_name,
        SecurityGroupIds=[sg_id],
        SubnetId=subnet_id,
        MinCount=1,
        MaxCount=1,
        TagSpecifications=[
            {
                "ResourceType": "instance",
                "Tags": [
                    {"Key": "Name", "Value": name},
                    {"Key": "Owner", "Value": "python-lab"},
                ],
            }
        ],
        BlockDeviceMappings=[
            {
                "DeviceName": "/dev/xvda",
                "Ebs": {"VolumeSize": 10, "VolumeType": "gp3", "DeleteOnTermination": True},
            }
        ],
    )

    instance = resp["Instances"][0]
    instance_id = instance["InstanceId"]
    print(f"Created instance: {instance_id}. Waiting for 'running'…")

    # Wait until running
    waiter = ec2.get_waiter("instance_running")
    waiter.wait(InstanceIds=[instance_id])

    # Fetch public details
    desc = ec2.describe_instances(InstanceIds=[instance_id])
    inst = desc["Reservations"][0]["Instances"][0]
    public_ip = inst.get("PublicIpAddress")

    state = {
        "region": region,
        "name": name,
        "instance_id": instance_id,
        "key_name": key_name,
        "security_group_id": sg_id,
        "public_ip": public_ip,
    }
    save_state(state)
    print("Instance is running.")
    print(json.dumps(state, indent=2))


def start_instance(region: str, instance_id: str):
    ec2, _ = get_clients(region)
    print(f"Starting instance {instance_id}…")
    ec2.start_instances(InstanceIds=[instance_id])
    ec2.get_waiter("instance_running").wait(InstanceIds=[instance_id])
    print("Instance is running.")


def stop_instance(region: str, instance_id: str):
    ec2, _ = get_clients(region)
    print(f"Stopping instance {instance_id}…")
    ec2.stop_instances(InstanceIds=[instance_id])
    ec2.get_waiter("instance_stopped").wait(InstanceIds=[instance_id])
    print("Instance is stopped.")


def terminate_instance(region: str, instance_id: str):
    ec2, _ = get_clients(region)
    print(f"Terminating instance {instance_id}…")
    ec2.terminate_instances(InstanceIds=[instance_id])
    ec2.get_waiter("instance_terminated").wait(InstanceIds=[instance_id])
    print("Instance is terminated.")


def status_instance(region: str, instance_id: str):
    ec2, _ = get_clients(region)
    resp = ec2.describe_instances(InstanceIds=[instance_id])
    inst = resp["Reservations"][0]["Instances"][0]
    state = inst["State"]["Name"]
    public_ip = inst.get("PublicIpAddress")
    print(json.dumps({"instance_id": instance_id, "state": state, "public_ip": public_ip}, indent=2))


# ---------- CLI ----------

def main():
    parser = argparse.ArgumentParser(description="AWS EC2 automation with Python (boto3)")

    sub = parser.add_subparsers(dest="command", required=True)

    p_create = sub.add_parser("create", help="Create a new EC2 instance")
    p_create.add_argument("--region", default=DEFAULT_REGION)
    p_create.add_argument("--name", default=DEFAULT_NAME_TAG)
    p_create.add_argument("--key-name", default="ec2-lab-key")
    p_create.add_argument("--ssh-cidr", default="0.0.0.0/0", help="CIDR allowed to SSH (e.g., 1.2.3.4/32)")
    p_create.add_argument("--type", default=DEFAULT_INSTANCE_TYPE, help="Instance type (e.g., t3.micro)")

    p_start = sub.add_parser("start", help="Start an EC2 instance")
    p_start.add_argument("--region", default=DEFAULT_REGION)
    p_start.add_argument("--instance-id", default=None)

    p_stop = sub.add_parser("stop", help="Stop an EC2 instance")
    p_stop.add_argument("--region", default=DEFAULT_REGION)
    p_stop.add_argument("--instance-id", default=None)

    p_terminate = sub.add_parser("terminate", help="Terminate an EC2 instance")
    p_terminate.add_argument("--region", default=DEFAULT_REGION)
    p_terminate.add_argument("--instance-id", default=None)

    p_status = sub.add_parser("status", help="Show status of an EC2 instance")
    p_status.add_argument("--region", default=DEFAULT_REGION)
    p_status.add_argument("--instance-id", default=None)

    args = parser.parse_args()

    # Allow reading instance_id from saved state if not provided
    state = load_state()

    if args.command == "create":
        return create_instance(
            region=args.region,
            name=args.name,
            key_name=args.key_name,
            ssh_cidr=args.ssh_cidr,
            instance_type=args.type,
        )

    # commands needing instance_id
    instance_id = getattr(args, "instance_id", None) or state.get("instance_id")
    if not instance_id:
        print("No instance_id provided and none saved in ec2_state.json. Use --instance-id or run 'create' first.")
        sys.exit(1)

    region = getattr(args, "region", DEFAULT_REGION)

    if args.command == "start":
        return start_instance(region, instance_id)
    if args.command == "stop":
        return stop_instance(region, instance_id)
    if args.command == "terminate":
        return terminate_instance(region, instance_id)
    if args.command == "status":
        return status_instance(region, instance_id)


if __name__ == "__main__":
    main()
```

> This **one file** handles everything: create, start, stop, status, terminate. It also writes a local `ec2_state.json` so you don’t have to copy the instance-id around.

---

## 7) How to run it (copy‑paste commands)

Make sure your virtual environment is active and you’re in `C:\aws-ec2-python`:

### 7.1 Create a new instance
```powershell
python .\ec2_automation.py create --name my-first-ec2 --key-name my-ec2-key --ssh-cidr 0.0.0.0/0
```
- `--ssh-cidr 0.0.0.0/0` means **anyone** can try to SSH; this is **not secure**.
- For better security, replace it with your **public IP with /32**, for example `49.207.12.34/32`.
- The script will create a **key pair** file named `my-ec2-key.pem` and a **security group** named `my-first-ec2-sg`.
- It waits until the instance is **running** and prints the **public IP**.

A file named **`ec2_state.json`** will be created which stores your instance details.

### 7.2 Check status
```powershell
python .\ec2_automation.py status
```
(Reads the instance-id from `ec2_state.json` by default.)

### 7.3 Stop the instance (to save money)
```powershell
python .\ec2_automation.py stop
```

### 7.4 Start it again later
```powershell
python .\ec2_automation.py start
```

### 7.5 Terminate (permanently delete) the instance
```powershell
python .\ec2_automation.py terminate
```
> This will **delete** the instance and its root volume. You cannot undo this.

---

## 8) SSH into the instance (optional)

If you opened SSH and want to try logging in from Windows (PowerShell):

1. Note the **public IP** from `status` output.
2. The default username for Amazon Linux 2023 is `ec2-user`.
3. Use the `.pem` key you created.

```powershell
# In the folder where my-ec2-key.pem exists
ssh -i .\my-ec2-key.pem ec2-user@<PUBLIC_IP>
```
If you face a permissions warning about the key on Windows, you can still proceed; on Linux/macOS you’d run `chmod 400 my-ec2-key.pem`.

---

## 9) Common errors & quick fixes

- **AuthFailure / UnrecognizedClientException**: Your keys are wrong or not configured. Run `aws configure` again.
- **UnauthorizedOperation**: Your IAM user doesn’t have required permissions. For the lab, attach `AdministratorAccess`.
- **OptInRequired**: The region is not enabled for your account. Switch to a common region like `ap-south-1`.
- **InvalidKeyPair.NotFound**: The `--key-name` you passed does not exist yet. Run `create` so the script can create it.
- **Timeout waiting for running/stopped**: Check your AWS Console to see the real state; try again.
- **SG already has this rule**: Safe to ignore (the script tries to ensure rules exist).

---

## 10) Clean up checklist

- If you’re done, run `terminate` to avoid charges.
- Delete **unused** security groups or key pairs if you created many.
- You can also delete `ec2_state.json` locally; it only stores IDs, not secrets.

---

## 11) What you learned

- Installing and configuring **Python**, **AWS CLI**, and **Boto3**.
- Programmatically **creating** an EC2 instance with the latest Amazon Linux 2023 AMI.
- Managing lifecycle: **start**, **stop**, **status**, **terminate** using **waiters** for safe state transitions.
- Saving and reusing state locally so beginners don’t have to copy instance IDs around.

---

## 12) Next steps (nice add‑ons)

- Add **CloudWatch** to start/stop on schedule (e.g., nights, weekends).
- Tag resources with your **name** and **purpose** for easy tracking.
- Replace broad SSH access with your exact IP CIDR (e.g., `x.y.z.w/32`).
- Create a small **Streamlit** or **Typer** CLI UI around this script.
- Try using **CloudFormation** or **Terraform** later for full IaC.

---

### 📌 Mini‑FAQ

- **Why Boto3?** It’s AWS’s official Python SDK; stable and well‑documented.
- **Can I use another AMI?** Yes—replace the AMI ID in the script or fetch a different SSM parameter.
- **Do I need a VPC?** The script uses your **default VPC** and a subnet in it.
- **What if there’s no default VPC?** Create one in the console or pass specific networking arguments (advanced).

---

That’s it! Follow the steps from **Section 2 → 7** in order, and you’ll have your first automated EC2 lifecycle working end‑to‑end.

