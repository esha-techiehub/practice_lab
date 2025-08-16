
# End-to-End Lab Activity: Using Key Pairs Across AWS Regions

This lab explains how to create, convert, and use key pairs across multiple AWS regions using **PuTTYgen** and **AWS Management Console** / **AWS CLI**.

---

## 1. Create a Key Pair in Mumbai Region
1. Log in to the **AWS Management Console**.
2. Navigate to **EC2 > Key Pairs**.
3. Click **Create key pair**.
4. Choose:
   - Name: `mumbai-key`
   - Key pair type: `RSA`
   - Private key file format: `.pem`
5. Download the `.pem` file.

---

## 2. Convert `.pem` to `.ppk` using PuTTYgen
Since Windows PuTTY needs `.ppk`, convert:
1. Open **PuTTYgen**.
2. Click **Load** and select the `mumbai-key.pem` file.
3. Click **Save private key** → Save as `mumbai-key.ppk`.

Now you have both:
- `mumbai-key.pem` → For Linux/Mac or AWS CLI
- `mumbai-key.ppk` → For PuTTY

---

## 3. Extract Public Key from `.pem`
Run this command in PowerShell (with OpenSSL installed) or Git Bash:

```bash
openssl rsa -in mumbai-key.pem -pubout -out mumbai-key.pub
```

This creates `mumbai-key.pub`.

---

## 4. Import Public Key into Another AWS Region (e.g., Singapore)
1. Log in to AWS Console.
2. Switch to **Singapore region**.
3. Go to **EC2 > Key Pairs > Import key pair**.
4. Give the key a name, e.g., `mumbai-key-sg`.
5. Copy the contents of `mumbai-key.pub` and paste into the **Public Key** field.
6. Click **Import**.

✅ Now the same key is available in Singapore.

---

## 5. Launch EC2 Instance in Singapore with Imported Key
1. Launch a new EC2 instance in **Singapore region**.
2. Select the **imported key pair** (`mumbai-key-sg`).
3. Finish the setup and launch the instance.

---

## 6. Connect to the Instance
- **Linux/Mac/WSL:**

```bash
ssh -i "mumbai-key.pem" ec2-user@<Public-IP>
```

- **Windows (PuTTY):**
   - Load `mumbai-key.ppk` in PuTTY under **Connection > SSH > Auth**.
   - Enter the public IP and connect.

---

## 7. Validate Access
If successful, you can log in to both **Mumbai** and **Singapore** instances using the **same private key** (`.pem` or `.ppk`).

---

## 🔑 Key Notes
- Only **public key** needs to be imported to other regions.
- **Private key never leaves your local machine**.
- Use `.pem` for Linux/CLI, `.ppk` for PuTTY (Windows).

---

## ✅ Lab Completed!
You can now securely use **one private key** to manage instances in multiple AWS regions.
