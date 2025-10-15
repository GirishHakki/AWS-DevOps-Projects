
# 🏗️ AWS VPC Project — Secure Network Setup with EC2, NAT Gateway & S3 Access

![AWS](https://img.shields.io/badge/AWS-Cloud-orange?logo=amazonaws)
![VPC](https://img.shields.io/badge/VPC-Networking-blue)
![DevOps](https://img.shields.io/badge/DevOps-Engineer-green)
![EC2](https://img.shields.io/badge/EC2-Instance-yellow)
![S3](https://img.shields.io/badge/S3-Storage-red)
![Made with ❤️ on AWS](https://img.shields.io/badge/Made%20with%20%E2%9D%A4%EF%B8%8F-on%20AWS-orange)

---

## 🔹 Overview
In this project, I created a **custom Virtual Private Cloud (VPC)** in AWS to understand how internal networking works in a cloud environment.  
The setup includes **Public and Private Subnets**, **Internet Gateway**, **NAT Gateway**, **VPC Endpoints**, and **IAM Roles** to enable secure connectivity between EC2 instances and AWS S3.

---

## 🧭 Architecture

**Architecture Flow:**  
**User → JumpServer (Public Subnet) → NAT Gateway → Private Subnet (DBServer) → S3 via VPC Endpoint**

---

## 🖼️ Architecture Diagram

![Architecture Diagram](A_digital_diagram_illustrates_an_AWS_(Amazon_Web_S.png)

---

## ⚙️ Step-by-Step Implementation

### **Step 1: Create VPC**
- Go to **VPC → Your VPCs → Create VPC**
- **Name:** MumbaiVPC  
- **IPv4 CIDR block:** `192.168.0.0/16`  
- **IPv6 CIDR block:** None  
- **Tenancy:** Default  
- Click **Create VPC**

---

### **Step 2: Create Internet Gateway**
- Go to **Internet Gateways → Create Internet Gateway**
- **Name:** MumbaiIGW  
- Click **Create**
- Select it → **Actions → Attach to VPC → MumbaiVPC**

---

### **Step 3: Create Subnets**
- Go to **Subnets → Create Subnet**
- **VPC:** MumbaiVPC  

**Subnet 1:**  
- Name: Public Subnet  
- AZ: ap-south-1a  
- IPv4 CIDR: 192.168.0.0/24  

**Subnet 2:**  
- Name: Private Subnet  
- AZ: ap-south-1b  
- IPv4 CIDR: 192.168.2.0/24  

Click **Create Subnets**

---

### **Step 4: Create NAT Gateway**
- Go to **NAT Gateways → Create NAT Gateway**
- **Name:** MumbaiNAT  
- **Subnet:** Public Subnet  
- **Connectivity Type:** Public  
- **Elastic IP:** Allocate new Elastic IP  
- Click **Create NAT Gateway**

---

### **Step 5: Create Route Tables**
- **Public RT**
  - Associate with **MumbaiVPC**
  - Add Route → `0.0.0.0/0` → **Internet Gateway (MumbaiIGW)**
  - Subnet Association → **Public Subnet**

- **Private RT**
  - Associate with **MumbaiVPC**
  - Add Route → `0.0.0.0/0` → **NAT Gateway (MumbaiNAT)**
  - Subnet Association → **Private Subnet**

---

### **Step 6: Create Security Group**
- Go to **Security Groups → Create Security Group**
- **Name:** MumbaiSG  
- **VPC:** MumbaiVPC  
- Add inbound rule:
  - Type: SSH → Source: My IP  
- Click **Create Security Group**

Now modify it:
- Add another inbound rule:
  - Type: All Traffic → Source: MumbaiSG (self-reference)
- Save changes

---

### **Step 7: Launch EC2 Instances**

#### **JumpServer (Public Subnet)**
- AMI: **Red Hat Enterprise Linux**
- Type: **t2.micro (Free Tier)**
- Key Pair: `reyazsir.ppk`
- Network:
  - VPC: MumbaiVPC  
  - Subnet: Public Subnet  
  - Auto-assign Public IP: Enabled  
  - Security Group: MumbaiSG  
- Click **Launch**

#### **DBServer (Private Subnet)**
- AMI: **Amazon Linux 2023 (kernel 6.1)**  
- Type: **t2.micro**
- Key Pair: `reyazsir.ppk`
- Network:
  - VPC: MumbaiVPC  
  - Subnet: Private Subnet  
  - Auto-assign Public IP: Disabled  
  - Security Group: MumbaiSG  
- Click **Launch**

---

### **Step 8: Connect EC2 Instances**

#### Login to JumpServer
```bash
ssh -i "reyazsir.pem" ec2-user@<JumpServer_Public_IP>
sudo -s

From JumpServer → Connect to DBServer
vi reyazsir.pem   # create file
# paste private key content
:wq!
chmod 400 reyazsir.pem
ssh -i "reyazsir.pem" ec2-user@192.168.2.64
sudo -s

---

From JumpServer → Connect to DBServer
```bash 
vi your.pem   # create file
# paste private key content
:wq!
chmod 400 reyazsir.pem
ssh -i "your.pem" ec2-user@192.168.2.64
sudo -s

```

---

### Step 9: Configure IAM Role & Attach to DBServer

Go to IAM → Roles → Create Role
Trusted entity: AWS Service → EC2
Permissions: AmazonS3FullAccess
Role name: myvpcrole
Attach Role:
EC2 → Select DBServer → Actions → Security → Modify IAM Role → myvpcrole

---

Step 10: Access S3 from DBServer

Verify AWS CLI:
```bash
aws --version


Create a new S3 bucket:

aws s3 mb s3://your-vpce-role --region ap-south-1
aws s3 ls --region ap-south-1
```

---

### Step 11: Create VPC Endpoint for S3

Go to VPC → Endpoints → Create Endpoint
Name: S3-VPCE
Service Type: AWS Services
Service: S3 (Gateway)
VPC: MumbaiVPC
Route Table: Private RT
Click Create Endpoint
Now your DBServer in the Private Subnet can access S3 without using the Internet Gateway or NAT Gateway.


















