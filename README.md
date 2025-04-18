# 🚀 **Deploying OpenStack on an EC2 Instance - The Ultimate Guide** 🌩️

Welcome to the **most exciting** guide on setting up OpenStack on AWS EC2! Whether you're a cloud enthusiast, a DevOps engineer, or just curious, this guide will make your OpenStack journey **smooth and fun**. Let’s turn that EC2 instance into a powerful private cloud! ☁️⚡

---

## **✨ Table of Contents** ✨
1. [🎯 **Prerequisites**](#-prerequisites)
2. [🔌 **Connecting to EC2 Instance**](#-connecting-to-the-ec2-instance)
3. [⚙️ **System Setup**](#️-update-system-and-install-dependencies)
4. [📥 **DevStack Installation**](#-clone-the-devstack-repository)
5. [🔧 **Configuration**](#-create-a-devstack-configuration-file)
6. [🚀 **OpenStack Deployment**](#-install-openstack-using-devstack)
7. [🖥️ **Accessing Horizon**](#️-access-the-openstack-horizon-dashboard)
8. [✅ **Service Verification**](#-verify-openstack-services)
9. [🛠️ **Troubleshooting**](#️-troubleshooting-and-service-management)
10. [🖥️ **Deploying a VM**](#️-deploying-a-virtual-machine)
11. [🎉 **Conclusion**](#-conclusion)

---

## **🎯 Prerequisites**
Before we dive in, let’s make sure you have everything ready!

### **☁️ AWS EC2 Instance Requirements**
- **AMI:** Ubuntu 22.04 LTS (Best for stability)  
- **Instance Type:** `t2.large` (2 vCPUs, 8GB RAM)  
- **Storage:** **30GB+** (Default 8GB won’t cut it!)  
- **Key Pair:** Your SSH key (Keep it safe! 🔑)  
- **Security Group Rules:**  
  - **SSH (22)** → Your IP only  
  - **HTTP (80) & HTTPS (443)** → Open for Horizon  

![EC2 Setup](https://github.com/user-attachments/assets/ea0ae67d-551b-4d5d-9d6f-65cdeb8492ee)

### **💻 Local Machine Setup**
- **Windows:** PuTTY + PuTTYgen  
- **Linux/macOS:** Built-in terminal (Just `ssh` away!)  

---

## **🔌 Step 1: Connecting to the EC2 Instance**
Time to jump into your cloud machine!  

### **For Windows (PuTTY Users)**
```markdown
1. Convert `.pem` to `.ppk` using PuTTYgen  
2. Open PuTTY → Enter:  
   - Host: `EC2_Public_IP`  
   - Load your `.ppk` key  
3. Login as `ubuntu`  
```

### **For Linux/macOS (Terminal Pros)**
```bash
chmod 400 your-key.pem  # Lock it down!
ssh -i your-key.pem ubuntu@EC2_Public_IP
```
**💡 Pro Tip:** Use `tmux` or `screen` to avoid disconnections!  

---

## **⚙️ Step 2: Update & Install Dependencies**
Let’s get the system ready for OpenStack magic!  

```bash
sudo apt update && sudo apt upgrade -y  # Fresh & updated!
sudo apt install -y git python3-dev python3-pip net-tools  # Essentials!
```

---

## **📥 Step 3: Clone DevStack Repository**
DevStack = Fastest way to deploy OpenStack!  

```bash
git clone https://opendev.org/openstack/devstack.git
cd devstack  # Let’s get into the action!
```

![DevStack Clone](https://github.com/user-attachments/assets/a832551c-d86d-4a7c-9e32-6998b75c7846)

---

## **🔧 Step 4: Configure DevStack**
Let’s customize OpenStack before installation!  

```bash
nano local.conf  # Time for some config magic!
```
Paste this **optimized** config:  
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

![Editing Config](https://github.com/user-attachments/assets/2493e32d-7b8a-42c2-af0b-0e08c249738e)

---

## **🚀 Step 5: Install OpenStack!**
Hold tight—this will take **30-45 mins**! ☕  

```bash
./stack.sh  # Let the magic begin!
```

![OpenStack Installation](https://github.com/user-attachments/assets/a0a40b44-e7d3-4005-b076-756a8e9281ee)

---

## **🖥️ Step 6: Access Horizon Dashboard**
Your OpenStack GUI is ready!  

🌐 **URL:** `http://EC2_Public_IP/dashboard`  

![image](https://github.com/user-attachments/assets/085d9919-8cf3-40d3-a4bb-18d5a429507a)

🔑 **Login:**  
- **Username:** `admin`  
- **Password:** `SuperSecret`  

![Horizon Dashboard](https://github.com/user-attachments/assets/1399443d-4df3-4adf-b726-c8b2f08e6a72)

![image](https://github.com/user-attachments/assets/0741a952-b3a0-4c98-9deb-81566af855a5)
---

## **✅ Step 7: Verify Services**
Let’s make sure everything’s running smoothly!  

```bash
source openrc admin admin  # Load OpenStack credentials
openstack service list  # Check all services
```

### **🔍 Quick Service Checks**
| Service  | Command |
|----------|---------|
| **Keystone** | `openstack token issue` |
| **Nova** | `openstack compute service list` |
| **Neutron** | `openstack network agent list` |
| **Glance** | `openstack image list` |

![image](https://github.com/user-attachments/assets/bc6c8902-e8bc-453a-8848-14564446f381)

![image](https://github.com/user-attachments/assets/7ef9c56a-4ca8-4df2-b382-767b60f65dcc)

![image](https://github.com/user-attachments/assets/fac9a2c4-7734-4f1a-86e2-62c134af4faf)
---

## **🛠️ Step 8: Troubleshooting**
Hit a snag? Let’s fix it!  

### **Common Issues & Fixes**
| Problem | Solution |
|---------|----------|
| **Horizon not loading** | `sudo systemctl restart apache2` |
| **Nova service down** | `sudo systemctl restart devstack@n-api` |
| **Networking issues** | Check `/var/log/neutron/*.log` |

### **📂 Log Locations**
- **DevStack logs:** `/opt/stack/logs`  
- **Service logs:** `/var/log/[service_name]`  

---

# **🖥️ Step 9: Deploying a VM in OpenStack Horizon** 🎮
Time to launch your **first cloud VM**! Follow these steps carefully.  

---

## **🔑 Step 1: Create a Key Pair**
You’ll need this to SSH into your VM!  

1. Go to **Project → Compute → Key Pairs**  
2. Click **Create Key Pair**  
   - **Name:** `test-key`  
   - **Key Type:** `SSH Key (RSA)`  
3. **Download** the `.pem` file (Keep it safe!)  

![image](https://github.com/user-attachments/assets/a16c9f3f-b32f-4ae1-8980-195ae1956340)

---

## **🖼️ Step 2: Upload a Cloud Image**
We’ll use **Ubuntu 22.04** for the VM.  

1. Navigate to **Project → Compute → Images**  
2. Click **Create Image**  
   - **Name:** `Ubuntu-22.04`  
   - **Image Source:** `URL`  
   - **URL:**  
     ```bash
     https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-amd64.img
     ```
3. Wait for **Active** status  

![image](https://github.com/user-attachments/assets/3bb64c90-c086-4e5d-ae3a-2251934fdd6e)

---

## **⚡ Step 3: Define a Flavor**
A **flavor** = VM size (CPU, RAM, Disk).  

1. Go to **Admin → Compute → Flavors**  
2. Click **Create Flavor**  
   - **Name:** `small`  
   - **vCPUs:** `1`  
   - **RAM:** `2048 MB`  
   - **Disk:** `10 GB`  

![image](https://github.com/user-attachments/assets/6a7e2ebc-e8fc-49d0-9113-9ec4de57c025)


---

## **🌐 Step 4: Set Up Networking**
A VM needs a network!  

### **A. Create a Private Network**
1. **Project → Network → Networks**  
2. **Create Network**  
   - **Name:** `private-net`
   - **Subnet Name:** `private-subnet`
   - **Subnet Address:** `192.168.1.0/24`  

### **B. Add a Router**
1. **Project → Network → Routers**  
2. **Create Router** → Name: `my-router`

Now `delete` the default `router` and go back to `Networks` and delete the `private network` as well.

![image](https://github.com/user-attachments/assets/721f1e81-7603-48aa-998a-c55a387547c2)


---

## **🚀 Step 5: Launch the VM!**
Time for liftoff!  

1. Go to **Project → Compute → Instances**  
2. Click **Launch Instance**  
   - **Name:** `my-instance`  
   - **Flavor:** `small`  
   - **Image:** `Ubuntu-22.04`  
   - **Network:** `private-net`  
   - **Key Pair:** `test-key`  

![image](https://github.com/user-attachments/assets/75794a8a-fb53-44de-a0fd-a64b4945edcd)


---

## **🔐 Step 6: SSH into Your VM**
Once the VM is **Active**, connect via:  

```bash
chmod 400 test-key.pem
ssh -i my-key.pem ubuntu@<VM_IP>
```
**🎉 Congrats! You’re in your OpenStack VM!**  

---

## **🎉 Conclusion**
You’ve just **built your own private cloud** on AWS! 🎉  

### **What’s Next?**
- **Scale up:** Add more nodes!  
- **Automate:** Use Terraform/Ansible!  
- **Experiment:** Try Kubernetes on OpenStack!  

**Happy Cloud Building!** ☁️🚀  

**References:**
- [DevStack Documentation](https://docs.openstack.org/devstack/latest/)
- [OpenStack CLI Reference](https://docs.openstack.org/python-openstackclient/latest/cli/)
---

**💬 Got questions?**  
Drop them below! Let’s make cloud computing **fun and easy**! 😃
