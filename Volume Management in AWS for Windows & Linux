# Volume Attachment and Management in AWS (Windows & Linux)

This lab demonstrates how to **create, attach, and manage secondary volumes** in AWS EC2 instances, both on **Windows** and **Linux** servers. It covers **volume creation, mounting, resizing, and permanent mounting**.

---

##  Lab 1: Attach a Volume in **Windows Server**

### Steps:
1. **Launch an Instance**  
   Create a new EC2 Windows instance.

2. **Create and Attach a Volume**  
   - Go to **AWS Console → Elastic Block Store (EBS) → Volumes**  
   - Create a new volume (ensure it is in the same **Availability Zone** as the instance).  
   - Attach the volume to your instance.

   ```
   +------------------+        +------------------+
   |   EBS Volume     | -----> |   EC2 Instance   |
   | (Secondary Disk) |        |  Windows Server  |
   +------------------+        +------------------+
   ```

3. **Connect to Windows Server**  
   - Download the key pair from AWS.  
   - Decrypt the password and connect using **RDP (Remote Desktop Protocol)**.

4. **Open Disk Management**  
   - Press `Win + R` → type `diskmgmt.msc` → Enter.  
   - Initialize the disk:  
     - **MBR (Master Boot Record):** If `< 2TB`  
     - **GPT (GUID Partition Table):** If `> 2TB`

5. **Create and Format the Volume**  
   - Right-click → **New Simple Volume** → Assign a drive letter → Format with a filesystem.

6. **Modify and Extend Volume**  
   - To increase size → update in **AWS Console**.  
   - Extend volume inside Windows **Disk Management**.

---

### File System Types in Windows
- **NTFS** → New Technology File System (default, secure, supports large files)  
- **ExFAT** → Extended File Allocation Table (cross-platform, external drives)  
- **ReFS** → Resilient File System (high availability, auto-healing)

---

##  Lab 2: Attach a Volume in **Linux Server**

### Steps:
1. **Launch an Instance and Create a Volume**  
   - Ensure the volume is in the **same Availability Zone** as the instance.  
   - Attach the volume to the EC2 instance.

   ```
   +------------------+        +------------------+
   |   EBS Volume     | -----> |   EC2 Instance   |
   | (Secondary Disk) |        |   Linux Server   |
   +------------------+        +------------------+
   ```

2. **Login to Instance**  
   ```bash
   ssh -i "key.pem" ec2-user@<public-ip>
   ```

3. **Check Available Volumes**  
   ```bash
   lsblk        # Lists block devices
   df -hT       # Shows mounted devices with filesystem details
   ```

4. **Format the Volume**  
   ```bash
   sudo mkfs.ext4 /dev/nvme1n1
   ```

5. **Create a Mount Point and Mount Volume**  
   ```bash
   sudo mkdir /home/ec2-user/eshwari
   sudo mount /dev/nvme1n1 /home/ec2-user/eshwari
   ```

6. **Verify Mounting**  
   ```bash
   lsblk
   df -hT
   ```

   ```
   /dev/nvme1n1  --->  /home/ec2-user/eshwari
   ```

---

### Types of Linux File Systems
- **Ext2, Ext3, Ext4** → Common Linux filesystems  
- **XFS** → High performance (default in Amazon Linux 2)  
- **Btrfs, ZFS** → Advanced features  

---

##  Lab 3: Increase Volume Size (Linux)

1. **Modify Volume Size in AWS Console**  
   - Go to **Volumes → Modify Volume**.  
   - Example: Increase from `100GB → 120GB`.

2. **Check in Instance**  
   ```bash
   lsblk     # Shows updated size
   df -hT    # Old size still visible
   ```

3. **Resize Filesystem**  
   ```bash
   sudo resize2fs /dev/nvme2n1
   ```

   ```
   AWS Console Resize ---> lsblk updated ---> resize2fs ---> df -hT shows new size
   ```

---

##  Lab 4: Permanent Mounting (Linux)

By default, mounting is **temporary**. After reboot, the volume will not be mounted. To make it **permanent**, update the `fstab` file.

1. **Edit `fstab` File**  
   ```bash
   sudo vim /etc/fstab
   ```

   Add entry:
   ```bash
   /dev/nvme1n1   /home/ec2-user/vishwa   ext4   defaults,noatime   1   1
   ```

2. **Mount All Filesystems**  
   ```bash
   sudo mount -a
   ```

3. **Test by Creating Files**  
   ```bash
   sudo touch /home/ec2-user/vishwa/file{1..1000}
   sudo mkdir /home/ec2-user/vishwa/dir{1..199}
   ```

   Reboot and check if the data persists.

---

##  Mounting Types in Linux
```
+--------------------+
|  Mounting Types    |
+--------------------+
| Temporary Mounting | --> Data lost after reboot
| Permanent Mounting | --> Configured in /etc/fstab
+--------------------+
```

---

##  Summary

- **Windows:** Use `diskmgmt.msc` to manage volumes. Supports **NTFS, ExFAT, ReFS**.  
- **Linux:** Use `lsblk`, `df -hT`, `mkfs`, and `mount`. Supports **Ext4, XFS, others**.  
- **Resizing:** Modify in AWS Console, then use `resize2fs` (Linux) or Disk Management (Windows).  
- **Mounting:**  
  - **Temporary:** Manual mount (`mount /dev/...`).  
  - **Permanent:** Update `/etc/fstab`.

---

✅ This completes **Volume Management in AWS for Windows & Linux**.
