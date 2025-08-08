# aws-vpc-webapp-with-bastion-and-alb
Real-world DevOps project: Deploy a website inside a private subnet with SSH access via Bastion and public access via Load Balancer.”

## 📁 Table of Contents
- [Architecture Overview](#architecture-overview)
- [Components Used](#components-used)
- [Deployment Steps](#deployment-steps)
- [Security Group Configuration](#security-group-configuration)
- [Accessing the Website](#accessing-the-website)
- [Key Learnings](#key-learnings)
- [Architecture Diagram](#architecture-diagram)

---

## 🧱 Architecture Overview

- Custom VPC with public and private subnets
- Bastion Host in public subnet for secure SSH
- NAT Gateway for internet access from private subnets
- Hosted a Tooplate-based static website on a private EC2 instance using Apache HTTP server, accessible only via an Application Load Balancer.
- Application Load Balancer (ALB) in public subnet
- Security Groups with least privilege access control

---

## 🧩 Components Used

| Resource Type        | Details                             |
|----------------------|--------------------------------------|
| VPC                  | Custom VPC with CIDR range           |
| Subnets              | 2 Public, 2 Private (across AZs)     |
| Bastion Host         | Public EC2 instance (SSH Gateway)    |
| Private Server       | EC2 instance hosting website         |
| Load Balancer        | ALB with listener on port 80         |
| NAT Gateway          | Enables private subnet internet access |
| Security Groups      | Granular access control between components |
| Key Pairs            | `bastion-key.pem` and `web-key.pem` |

---

## 🚀 Deployment Steps

### 1. Create Key Pair
- Create a key pair named `web-key` from AWS Console.
- Download the `.pem` file.

### 2. Copy Key to Bastion Host
```bash
scp -i bastion-key.pem web-key.pem ubuntu@<bastion-host-public-ip>:/home/ubuntu

## 🚀 Launching and Configuring the Website

### 3. Launch Private EC2 Instance

- **AMI**: Amazon Linux 2  
- **Instance Type**: t2.micro  
- **Subnet**: Private Subnet  
- **Key Pair**: `web-key`  
- **Security Group**:  
  - Allow **SSH (port 22)** from Bastion SG  
  - Allow **HTTP (port 80)** from Load Balancer SG

---

### 4. SSH to Private Instance via Bastion Host

```bash
ssh -i bastion-key.pem ubuntu@<bastion-public-ip>
chmod 400 web-key.pem
ssh -i web-key.pem ec2-user@<private-ec2-ip>

### 5. Website Setup on Private Instance
Install Apache and download a template from Tooplate:

sudo yum install httpd wget unzip -y
wget <template_download_url>
unzip <template>.zip
sudo cp -r <unzipped_folder>/* /var/www/html/
sudo systemctl start httpd
sudo systemctl enable httpd

### 6. Create Load Balancer
      1: Create Target Group
         Name: web-tg
         Protocol: HTTP
         Port: 80
         Target: Private EC2 instance

      2. Create Application Load Balancer (ALB)
          Type: Application Load Balancer
          Subnets: Public Subnets
          Security Group: Allow HTTP (port 80) from 0.0.0.0/0
          Listener: Port 80 → Target Group web-tg

### 7. Security Group Configuration
       Component	Inbound Rules
        Bastion Host	SSH (22) from your IP
        Private Instance	SSH (22) from Bastion SG
        HTTP (80) from Load Balancer SG
        Load Balancer	HTTP (80) from 0.0.0.0/0

### 8. Accessing the Website
Once the Load Balancer is active and the target instance is Healthy, open:
http://<your-load-balancer-dns>
You should see your static website (from Tooplate) hosted on the private EC2 instance.




