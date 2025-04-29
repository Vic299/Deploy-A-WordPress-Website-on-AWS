
# 🖥️ Host a WordPress Website on AWS

This project provides a secure, scalable, and highly available infrastructure to host a WordPress website using Amazon Web Services (AWS). The solution is built with best practices in mind, leveraging services such as EC2, RDS, EFS, Auto Scaling, and more.

---

## 📌 Architecture Overview

This architecture consists of:

- **Amazon EC2** instances for hosting WordPress in private subnets
- **Amazon RDS (MySQL)** with Multi-AZ for high availability
- **Amazon EFS** for shared storage between EC2 instances
- **Application Load Balancer (ALB)** for routing traffic
- **Amazon Route 53** for DNS management
- **Auto Scaling Group** for scalability
- **Public and Private Subnets** across multiple Availability Zones
- **IAM, NAT Gateways, and Security Groups** for secure access and traffic flow

---

## 🚀 Deployment Guide

### ✅ Prerequisites

Before you begin, ensure the following are set up:

- AWS CLI configured
- A VPC with public/private subnets and an Internet Gateway
- EC2 key pair
- Security groups allowing HTTP (80), HTTPS (443), and SSH (22) as needed
- EFS file system and mount targets in each AZ
- RDS MySQL instance (Multi-AZ preferred)

---

### 🛠️ Option 1: Manual EC2 Setup

Use the following script to manually configure WordPress on an EC2 instance.

#### Script: `Install wordpress.sh`

```bash
# Switch to root and update system
sudo su
sudo yum update -y

# Mount EFS
EFS_DNS_NAME=fs-xxxxxxx.efs.us-east-1.amazonaws.com
sudo mkdir -p /var/www/html
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport "$EFS_DNS_NAME":/ /var/www/html

# Install Apache and PHP
sudo yum install -y httpd
sudo systemctl enable httpd
sudo systemctl start httpd

sudo dnf install -y php php-cli php-cgi php-curl php-mbstring php-gd php-mysqlnd php-gettext php-json php-xml php-fpm php-intl php-zip php-bcmath php-ctype php-fileinfo php-openssl php-pdo php-tokenizer

# Install MySQL
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf install -y mysql-community-server
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Set permissions
sudo usermod -a -G apache ec2-user
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
sudo find /var/www -type f -exec sudo chmod 0664 {} \;
sudo chown apache:apache -R /var/www/html

# Download WordPress
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
sudo cp -r wordpress/* /var/www/html/

# Configure WordPress
sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
sudo vi /var/www/html/wp-config.php  # Add DB credentials here

# Restart Apache
sudo service httpd restart
```

---

### ⚙️ Option 2: Launch Template Script

Use the following script in your EC2 Launch Template for automated provisioning.

#### Script: `launch-template-script-install-wordpress.sh`

```bash
#!/bin/bash
sudo yum update -y

# Install Apache and PHP
sudo yum install -y httpd
sudo systemctl enable httpd
sudo systemctl start httpd

sudo dnf install -y php php-cli php-cgi php-curl php-mbstring php-gd php-mysqlnd php-gettext php-json php-xml php-fpm php-intl php-zip php-bcmath php-ctype php-fileinfo php-openssl php-pdo php-tokenizer

# Install MySQL
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf install -y mysql-community-server
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Mount EFS
EFS_DNS_NAME=fs-xxxxxxx.efs.us-east-1.amazonaws.com
echo "$EFS_DNS_NAME:/ /var/www/html nfs4 nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 0 0" >> /etc/fstab
mount -a

# Set permissions
chown apache:apache -R /var/www/html

# Restart Apache
sudo service httpd restart
```

> Replace `fs-xxxxxxx.efs.us-east-1.amazonaws.com` with your actual EFS DNS name.

---

## 🌐 Access Your Site

Once deployed, access your WordPress site at:

```
http://<Application-Load-Balancer-DNS-Name>
```

Proceed with the web-based WordPress installation and set up your admin account.

---

## 🧪 Testing Checklist

- ✅ Load Balancer routes traffic correctly
- ✅ WordPress install page is accessible
- ✅ wp-config.php is correctly configured with DB credentials
- ✅ EC2 instances auto-scale as expected
- ✅ EFS is mounted and shared across instances
- ✅ RDS failover works (if Multi-AZ enabled)

---

## 📚 Resources

- [WordPress Official Site](https://wordpress.org)
- [AWS EC2](https://aws.amazon.com/ec2/)
- [Amazon RDS](https://aws.amazon.com/rds/)
- [Amazon EFS](https://aws.amazon.com/efs/)
- [Route 53](https://aws.amazon.com/route53/)
- [Auto Scaling](https://aws.amazon.com/autoscaling/)

---

