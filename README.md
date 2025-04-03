# **Deploying OpenStack on an EC2 Instance - Comprehensive Guide**

This document provides a complete, step-by-step guide for setting up **OpenStack** on an **AWS EC2 instance** using DevStack, including service verification and VM deployment.

---

## **Table of Contents**
1. [Prerequisites](#prerequisites)
2. [Connecting to EC2 Instance](#1-connecting-to-the-ec2-instance)
3. [System Setup](#2-update-system-and-install-dependencies)
4. [DevStack Installation](#3-clone-the-devstack-repository)
5. [Configuration](#4-create-a-devstack-configuration-file)
6. [OpenStack Deployment](#5-install-openstack-using-devstack)
7. [Accessing Horizon](#6-access-the-openstack-horizon-dashboard)
8. [Service Verification](#7-verify-openstack-services)
9. [Troubleshooting](#8-troubleshooting-and-service-management)
10. [VM Deployment](#9-deploying-a-virtual-machine)
11. [Conclusion](#10-conclusion)

---

## **Prerequisites**

### **1. AWS EC2 Instance Requirements**
- **AMI:** Ubuntu 22.04 LTS (Recommended)
- **Instance Type:** `t2.large` (Minimum 2 vCPUs, 8GB RAM)
- **Storage:** At least **30GB** (Default 8GB is insufficient)
- **Key Pair:** Existing or newly created SSH key
- **Security Group Rules:**
  - **SSH (22):** Restricted to your IP
  - **HTTP (80) & HTTPS (443):** Open for Horizon dashboard

### **2. Software Requirements**
- **Windows:** PuTTY + PuTTYgen for SSH
- **Linux/macOS:** Native terminal with SSH
- **Git & Python 3** (Installed automatically in setup)

![EC2 Instance Setup](https://github.com/user-attachments/assets/ea0ae67d-551b-4d5d-9d6f-65cdeb8492ee)

---

## **1. Connecting to the EC2 Instance**

### **Windows (PuTTY)**
```markdown
1. Convert PEM to PPK using PuTTYgen
2. In PuTTY:
   - Host: EC2_Public_IP
   - Connection > SSH > Auth: Load PPK file
3. Login as `ubuntu`
```

### **Linux/macOS (Terminal)**
```bash
chmod 400 your-key.pem
ssh -i your-key.pem ubuntu@EC2_Public_IP
```

---

## **2. Update System and Install Dependencies**
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git python3-dev python3-pip net-tools
```

---

## **3. Clone DevStack Repository**
```bash
git clone https://opendev.org/openstack/devstack.git
cd devstack
```

![image](https://github.com/user-attachments/assets/a832551c-d86d-4a7c-9e32-6998b75c7846)

---

## **4. Create DevStack Configuration**
```bash
nano local.conf
```
Paste:
```ini
[[local|localrc]]
ADMIN_PASSWORD=SuperSecret
DATABASE_PASSWORD=$ADMIN_PASSWORD
RABBIT_PASSWORD=$ADMIN_PASSWORD
SERVICE_PASSWORD=$ADMIN_PASSWORD

# Set Host IP (Detects Automatically)
HOST_IP=$(hostname -I | awk '{print $1}')

# Enable Only Essential Services
disable_service tempest
disable_service cinder
disable_service swift
disable_service heat

# Use Latest OpenStack Release
USE_PYTHON3=True
GIT_BASE=https://opendev.org


```

![image](https://github.com/user-attachments/assets/2493e32d-7b8a-42c2-af0b-0e08c249738e)

---

## **5. Install OpenStack Using DevStack**
```bash
./stack.sh
```
*Takes 30-45 minutes*

![image](https://github.com/user-attachments/assets/a0a40b44-e7d3-4005-b076-756a8e9281ee)

---

## **6. Access Horizon Dashboard**
URL: `http://EC2_Public_IP/dashboard`  

![image](https://github.com/user-attachments/assets/085d9919-8cf3-40d3-a4bb-18d5a429507a)

**Credentials:**
- Username: `admin`
- Password: `SuperSecret`

![image](https://github.com/user-attachments/assets/1399443d-4df3-4adf-b726-c8b2f08e6a72)

![image](https://github.com/user-attachments/assets/0741a952-b3a0-4c98-9deb-81566af855a5)

---

## **7. Verify OpenStack Services**
```bash
source openrc admin admin
openstack service list
```

### **Individual Service Checks**
| Service  | Verification Command |
|----------|----------------------|
| Keystone | `openstack token issue` |
| Nova     | `openstack compute service list` |
| Neutron  | `openstack network agent list` |
| Glance   | `openstack image list` |

![image](https://github.com/user-attachments/assets/bc6c8902-e8bc-453a-8848-14564446f381)

![image](https://github.com/user-attachments/assets/7ef9c56a-4ca8-4df2-b382-767b60f65dcc)

![image](https://github.com/user-attachments/assets/fac9a2c4-7734-4f1a-86e2-62c134af4faf)


---

## **8. Troubleshooting and Service Management**

### **Common Issues**
1. **Service Down**
   ```bash
   sudo systemctl restart devstack@SERVICE_NAME
   ```
2. **Horizon Not Accessible**
   ```bash
   sudo systemctl restart apache2
   ```

### **Log Locations**
- DevStack logs: `/opt/stack/logs`
- Service logs: `/var/log/SERVICE_NAME`

---


# **9. Deploying a Virtual Machine (VM) in OpenStack Horizon**

This section provides a structured, step-by-step guide to deploying a virtual machine (VM) in OpenStack using the Horizon dashboard. Follow these instructions carefully to ensure a successful deployment.

---

## **Step 1: Access the OpenStack Horizon Dashboard**
1. Open a web browser and navigate to:
   ```bash
   http://<EC2_Public_IP>/dashboard
   ```
2. Log in using the following credentials:
   - **Username:** `admin`  
   - **Password:** `SuperSecret` (or the one you set in `local.conf`)

---

## **Step 2: Create a Key Pair for SSH Access**
A key pair is required to securely access your VM via SSH.

### **Steps:**
1. Navigate to:  
   **Project → Compute → Key Pairs**
2. Click **Create Key Pair**.
3. Enter the following details:
   - **Name:** `my-key`  
   - **Key Type:** `SSH Key (RSA)`  
4. Click **Create Key Pair**.
5. The private key (`my-key.pem`) will automatically download.  
   **⚠️ Store this securely—it cannot be retrieved later!**

![Key Pair Creation](https://github.com/user-attachments/assets/88f6c6a2-bc04-4e29-b37e-f2c8ed00791d)

---

## **Step 3: Upload a Cloud Image (OS for the VM)**
OpenStack requires a base OS image to launch VMs. We'll use Ubuntu 22.04.

### **Steps:**
1. Navigate to:  
   **Project → Compute → Images**
2. Click **Create Image**.
3. Fill in the details:
   - **Name:** `Ubuntu-22.04`  
   - **Image Source:** `URL`  
   - **Format:** `QCOW2`  
   - **URL:**  
     ```bash
     https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-amd64.img
     ```
4. Click **Create Image**.
5. Wait for the image to upload (status changes to **Active**).

![Uploading Cloud Image](https://github.com/user-attachments/assets/40cbae48-aebc-4611-8311-977b99e29487)

---

## **Step 4: Define a Flavor (VM Resource Allocation)**
A **flavor** determines the CPU, RAM, and disk allocation for your VM.

### **Steps:**
1. Navigate to:  
   **Admin → Compute → Flavors**
2. Click **Create Flavor**.
3. Configure the following:
   - **Name:** `small`  
   - **vCPUs:** `1`  
   - **RAM (MB):** `2048` (2GB)  
   - **Root Disk (GB):** `10`  
4. Click **Create Flavor**.

![Creating a Flavor](https://github.com/user-attachments/assets/576a9810-29a3-405a-89ec-326fb98f4c6a)

---

## **Step 5: Configure Networking**
A network must be set up for the VM to communicate.

### **A. Create a Private Network**
1. Navigate to:  
   **Project → Network → Networks**
2. Click **Create Network**.
3. Enter:
   - **Network Name:** `private-net`  
   - **Subnet Name:** `private-subnet`  
   - **Network Address:** `192.168.1.0/24`  
4. Click **Create**.

### **B. Set Up a Router**
1. Navigate to:  
   **Project → Network → Routers**
2. Click **Create Router**.
3. Enter:
   - **Router Name:** `my-router`  
4. Click **Create Router**.

> **Note:** Delete the default router and network if they exist.

![Network Configuration](https://github.com/user-attachments/assets/1d607160-813d-464b-832e-a8743f464f72)

---

## **Step 6: Launch the Virtual Machine**
Now, deploy a VM using the uploaded image, key pair, and network.

### **Steps:**
1. Navigate to:  
   **Project → Compute → Instances**
2. Click **Launch Instance**.
3. Configure the VM:
   - **Instance Name:** `my-instance`  
   - **Flavor:** `small` (1 vCPU, 2GB RAM)  
   - **Image:** `Ubuntu-22.04`  
   - **Network:** `private-net`  
   - **Key Pair:** `my-key`  
4. Click **Launch Instance**.

![Launching a VM](https://github.com/user-attachments/assets/9251dad9-2565-4e9d-92b0-c5b959eeff3b)

---

## **Step 7: Access the VM via SSH**
Once the VM is **Active**, you can SSH into it.

### **Steps:**
1. Retrieve the VM’s **IP Address** from the **Instances** tab.
2. Use the downloaded private key (`my-key.pem`) to SSH:
   ```bash
   chmod 400 my-key.pem
   ssh -i my-key.pem ubuntu@<VM_IP>
   ```

---

## **Troubleshooting**
| Issue | Solution |
|-------|----------|
| **SSH Connection Fails** | Ensure the security group allows SSH (port 22). |
| **VM Stuck in "Spawning"** | Check `/var/log/nova/nova-compute.log` for errors. |
| **No Internet in VM** | Verify the router has an external gateway. |

---

## **Conclusion**
You have successfully deployed a VM in OpenStack Horizon. This setup is ideal for:
- **Testing cloud workloads**  
- **Learning OpenStack**  
- **Developing cloud-native applications**  

For production, consider:
- **Multi-node clusters**  
- **Load balancing & auto-scaling**  
- **Persistent storage (Cinder)**  

**Next Steps:**  
Explore [OpenStack CLI](https://docs.openstack.org/python-openstackclient/latest/) for advanced automation. 🚀
