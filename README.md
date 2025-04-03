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


## **9. Deploy an Instance (VM) in Openstack Horizon**

1️⃣ **Open Openstack Dashboard**

  - Open your browser and go to:
  ```bash
  http://your-ec2-public-ip/dashboard
  ```
  - **Login Credentials**:
    - **Username**: `admin`
    - **Password**: `SuperSecret`

2️⃣ **Create a New Key Pair**

1. Go to: `Project` → `Compute` → `Key Pairs`
2. Click: `Create Key Pair`
3. Enter a Name: Example: `my-key`
4. Key Type: Select `SSH Key (RSA)`
5. Click: `Create Key Pair`
6. Download the Private Key (`.pem` file) and store it safely!
    - Example: `my-key.pem`

![7](https://github.com/user-attachments/assets/88f6c6a2-bc04-4e29-b37e-f2c8ed00791d)

3️⃣ **Upload an Image (OS for the VM)**

1. Go to: `Project` → `Compute` → `Images`
2. Click: `Create Image`
3. Enter Details:
    - Name: `Ubuntu-22.04`
    - Image Source: `Upload Image`
    - Format: `QCOW2`
    - URL (for Ubuntu 22.04):
    ```bash
    https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-amd64.img
    ```
4. Click: `Create Image`
5. Wait for the image to upload.

![8](https://github.com/user-attachments/assets/40cbae48-aebc-4611-8311-977b99e29487)

4️⃣ **Create a Flavor (VM Size)**

1. Go to: `Admin` → `Compute` → `Flavors`
2. Click: `Create Flavor`
3. Set the Following:
    - Name: `small`
    - vCPUs: `1`
    - RAM: `2048` MB (2GB)
    - Disk: `10` GB
4. Click: `Create Flavor`

![9](https://github.com/user-attachments/assets/576a9810-29a3-405a-89ec-326fb98f4c6a)

5️⃣ **Set Up Networking**

1. Go to: `Project` → `Network` → `Networks`
2. Click: `Create Network`
3. Enter Details:
    - Network Name: `private-net`
    - Subnet Name: `private-subnet`
    - Network Address: `192.168.1.0/24`
4. Click: `Create`

Then, set up a router:
1. Go to: `Project` → `Network` → `Routers`
2. Click: `Create Router`
3. Set Name: `my-router`
4. Click: `Create Router`

Now `delete` the default `router` and go back to `Networks` and delete the `private network` as well.

![10](https://github.com/user-attachments/assets/1d607160-813d-464b-832e-a8743f464f72)

6️⃣ **Launch an Instance (VM)**

1. Go to: `Project` → `Compute` → `Instances`
2. Click: `Launch Instance`
3. Enter Instance Details:
    - Instance Name: `my-instance`
    - Flavor: Select `small` (1 vCPU, 2GB RAM)
    - Image: Select your uploaded image (e.g., `Ubuntu-22.04`)
    - Network: Select `private-net`
    - Key Pair: Select `my-key`
4. Click: `Launch Instance`

![11](https://github.com/user-attachments/assets/9251dad9-2565-4e9d-92b0-c5b959eeff3b)

---
## **10. Conclusion**

You now have a fully functional OpenStack environment on AWS EC2. This setup is ideal for:
- Development and testing
- Learning OpenStack
- Proof-of-concept deployments

For production environments, consider:
- Multi-node deployment
- High availability configuration
- Enhanced security measures

**References:**
- [DevStack Documentation](https://docs.openstack.org/devstack/latest/)
- [OpenStack CLI Reference](https://docs.openstack.org/python-openstackclient/latest/cli/)
