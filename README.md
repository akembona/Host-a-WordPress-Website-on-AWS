![Alt text](2._Host_a_WordPress_Website_on_AWS.png)

# WordPress Website Hosting on AWS with High Availability & Scalability

This project demonstrates how to deploy and host a highly available, fault-tolerant, and scalable **WordPress** website on **Amazon Web Services (AWS)** using various managed services and DevOps best practices.

All infrastructure components were configured to ensure security, reliability, and performance, with resources organized using a multi-tier architecture pattern.

## 📁 Repository Contents

- 📄 Reference architecture diagram
- 📜 Deployment scripts for WordPress on EC2
- ⚙️ Auto Scaling Group (ASG) launch template script

---

## 🛠️ AWS Services Used

The following AWS components were used to provision the infrastructure:

1. **VPC with Subnets**
   - Created a Virtual Private Cloud (VPC) with public and private subnets spanning two Availability Zones (AZs).
   - Public subnets host NAT Gateway and Application Load Balancer.
   - Private subnets host EC2 instances (web servers) and RDS.

2. **Internet Gateway & NAT Gateway**
   - Internet Gateway allows public subnet resources to access the internet.
   - NAT Gateway enables private subnets to initiate outbound connections securely.

3. **Security Groups**
   - Implemented firewall rules to allow specific traffic to/from the EC2 instances, Load Balancer, and RDS.

4. **EC2 Instances**
   - Web servers are deployed in private subnets for added security.
   - User data scripts install and configure WordPress, Apache, PHP, and MySQL.

5. **Elastic File System (EFS)**
   - Shared file system for WordPress files, mounted to `/var/www/html`.

6. **RDS (MySQL)**
   - Managed relational database instance to store WordPress data.

7. **Application Load Balancer (ALB)**
   - Distributes incoming web traffic to multiple EC2 instances across AZs.

8. **Auto Scaling Group (ASG)**
   - Maintains desired number of EC2 instances based on traffic demand and health checks.

9. **Amazon Certificate Manager (ACM)**
   - Used to issue and manage SSL/TLS certificates for secure communication.

10. **Amazon SNS**
    - Notification system configured to alert administrators of Auto Scaling events.

11. **Route 53**
    - Domain registration and DNS management for routing to ALB.

12. **EC2 Instance Connect Endpoint**
    - Enables secure management of EC2 instances in private subnets without using bastion hosts.

---

## 📜 WordPress Installation Script (EC2 User Data)

Below is a simplified breakdown of the user data script for WordPress setup on a standalone EC2 instance:

```bash
# Switch to root user and update packages
sudo su
sudo yum update -y

# Install Apache & PHP with required extensions
sudo yum install -y httpd
sudo systemctl enable httpd && sudo systemctl start httpd

# Install PHP 8 and required extensions
sudo dnf install -y php php-cli php-mysqlnd php-xml php-json php-mbstring ...

# Install MySQL 8
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm
sudo dnf install -y mysql-community-server
sudo systemctl enable mysqld && sudo systemctl start mysqld

# Mount EFS to /var/www/html
EFS_DNS_NAME=fs-064e9505819af10a4.efs.us-east-1.amazonaws.com
sudo mkdir -p /var/www/html
sudo mount -t nfs4 -o nfsvers=4.1 ... "$EFS_DNS_NAME":/ /var/www/html

# Download and configure WordPress
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
cp -r wordpress/* /var/www/html/
cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php

# Restart Apache
sudo service httpd restart
````

---

## 🚀 Auto Scaling Group Launch Template Script

A second user data script was used in the **Launch Template** for the Auto Scaling Group. It installs Apache, PHP, MySQL, and mounts EFS, similar to the standalone script but designed to automate scaling:

```bash
# Update system packages
sudo yum update -y

# Install Apache and PHP
sudo yum install -y httpd
sudo dnf install -y php php-cli php-mysqlnd ...

# Enable and start Apache
sudo systemctl enable httpd
sudo systemctl start httpd

# Install MySQL
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm
sudo dnf install -y mysql-community-server
sudo systemctl enable mysqld && sudo systemctl start mysqld

# Mount EFS to /var/www/html
EFS_DNS_NAME=fs-02d3268559aa2a318.efs.us-east-1.amazonaws.com
echo "$EFS_DNS_NAME:/ /var/www/html nfs4 ..." >> /etc/fstab
mount -a

# Set permissions and restart Apache
chown apache:apache -R /var/www/html
sudo service httpd restart
```

---

## 🌐 Domain Name & DNS

* A domain name was registered through **Route 53**.
* DNS A and CNAME records were created to point the domain to the Application Load Balancer.

---

## 🔐 Security

* All EC2 instances reside in **private subnets**.
* **Security Groups** restrict traffic to only necessary ports (HTTP, HTTPS, SSH via EC2 Connect).
* **ACM** provides HTTPS encryption for secure data in transit.
* **IAM roles and policies** were attached to EC2 and Lambda (if applicable) for secure access to AWS services.

---

## 📡 Notifications

* **Amazon SNS** was configured to send notifications about changes or activities within the Auto Scaling Group.

---

## 📦 Version Control

* All configuration scripts and deployment files are version-controlled and stored in this GitHub repository to ensure collaboration and traceability.

---

## 🧰 How to Use This Project

> 🚨 This project assumes basic familiarity with AWS CLI, EC2, and infrastructure concepts.

1. Clone this repo and review the reference diagram.
2. Provision infrastructure using CloudFormation or Terraform (optional).
3. Launch EC2 instances using the provided user data scripts.
4. Mount EFS, connect RDS, and complete WordPress setup via browser.
5. Point your domain to the ALB via Route 53.

---

## 📎 Reference

* [WordPress Official Site](https://wordpress.org/)
* [Amazon EC2 Documentation](https://docs.aws.amazon.com/ec2/)
* [Amazon VPC Documentation](https://docs.aws.amazon.com/vpc/)
* [Amazon RDS Documentation](https://docs.aws.amazon.com/rds/)
* [Amazon EFS Documentation](https://docs.aws.amazon.com/efs/)

---

