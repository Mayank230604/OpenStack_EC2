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
#disable_service cinder
disable_service swift
disable_service heat

# Use Latest OpenStack Release
USE_PYTHON3=True
GIT_BASE=https://opendev.org

enable_service cinder
enable_service c-api
enable_service c-vol
enable_service c-sch
enable_service c-bak  # If you want volume backups

```

![Screenshot from 2025-03-29 02-03-20](https://github.com/user-attachments/assets/d3afe8b9-c1b6-4998-a48e-30bd2a509461)

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
**Credentials:**
- Username: `admin`
- Password: `SuperSecret`

![Horizon Login](https://github.com/user-attachments/assets/92629a6b-4ab1-4833-9bb3-de18b532fd0d)

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

![Service Verification](https://github.com/user-attachments/assets/4864989c-a066-4a55-a86e-b23841f5f273)

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

## **9. Deploying a Virtual Machine**

### **Step-by-Step VM Creation**
1. **Create Key Pair**
   - Navigate: Project → Compute → Key Pairs
   - Create & Download `.pem` file

2. **Upload Cloud Image**
   ```bash
   wget https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-amd64.img
   openstack image create --file jammy-server-cloudimg-amd64.img "Ubuntu-22.04"
   ```

3. **Create Network**
   ```bash
   openstack network create private-net
   openstack subnet create --network private-net --subnet-range 192.168.1.0/24 private-subnet
   ```

4. **Launch Instance**
   ```bash
   openstack server create --image Ubuntu-22.04 --flavor m1.small --key-name my-key --network private-net my-instance
   ```

![VM Creation](https://github.com/user-attachments/assets/9251dad9-2565-4e9d-92b0-c5b959eeff3b)

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
