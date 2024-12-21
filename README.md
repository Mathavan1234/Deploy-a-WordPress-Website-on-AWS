![Alt text](/Deploy-a-WordPress-Website-on-AWS.png)


# Hosting a WordPress Website on AWS

This project demonstrates how to host a WordPress website on AWS by utilizing a robust and scalable architecture. The implementation leverages AWS resources such as VPC, EC2 instances, Load Balancers, Auto Scaling Groups, and managed services for improved reliability, fault tolerance, and scalability.

## Key Features

1. **Virtual Private Cloud (VPC):** Configured with public and private subnets across two availability zones to ensure high availability and security.
2. **Internet Gateway:** Enabled connectivity between the VPC and the Internet.
3. **Security Groups:** Acted as a firewall to control inbound and outbound traffic.
4. **Multi-AZ Architecture:** Enhanced fault tolerance by leveraging two Availability Zones.
5. **Public Subnets:** Deployed infrastructure components like the NAT Gateway and Application Load Balancer.
6. **Private Subnets:** Hosted web servers for enhanced security.
7. **NAT Gateway:** Provided instances in private subnets with Internet access.
8. **EC2 Instances:** Hosted the WordPress website.
9. **Application Load Balancer:** Distributed traffic evenly to an Auto Scaling Group of EC2 instances.
10. **Auto Scaling Group:** Managed EC2 instances automatically to ensure scalability and availability.
11. **Amazon Certificate Manager:** Secured application communications with SSL/TLS certificates.
12. **Simple Notification Service (SNS):** Alerted about activities in the Auto Scaling Group.
13. **Amazon Route 53:** Managed the domain name and DNS records.
14. **Elastic File System (EFS):** Provided shared file storage for web servers.
15. **Relational Database Service (RDS):** Hosted the WordPress database for improved reliability.
16. **Version Control:** Stored deployment scripts and configuration files on GitHub for collaboration.

## Project Architecture

The following components were configured and deployed:

1. **VPC Configuration:**
   - Created public and private subnets across two Availability Zones.
   - Configured routing tables for Internet Gateway and NAT Gateway.

2. **Web Server Deployment:**
   - Deployed EC2 instances in private subnets.
   - Installed necessary software, including Apache, PHP, MySQL, and WordPress files.

3. **Load Balancer and Auto Scaling:**
   - Configured an Application Load Balancer to handle incoming traffic.
   - Set up an Auto Scaling Group to dynamically adjust the number of EC2 instances.

4. **Database Configuration:**
   - Used Amazon RDS to host the WordPress database.
   - Configured secure connectivity between the database and web servers.

5. **Storage:**
   - Mounted EFS to share files across EC2 instances.

6. **Domain Management:**
   - Registered a domain using Route 53.
   - Created DNS records to point the domain to the Load Balancer.

## Deployment Scripts and Resources

- **WordPress Installation Script:** See `Deploy-Script-Wordpress.sh` for the script used to install and configure WordPress.
- **Auto Scaling Launch Template Script:** The script to be added to the Auto Scaling launch template can be found in `EC2-Launch-Template.sh`.

## WordPress Installation Script

```bash
#!/bin/bash
# Switch to root user
sudo su

# Update the software packages on the EC2 instance
sudo yum update -y

# Create the web root directory
sudo mkdir -p /var/www/html

# Set the EFS DNS name (replace with your EFS DNS)
EFS_DNS_NAME=fs-0865fb2a73d784996.efs.ap-southeast-2.amazonaws.com

# Mount the EFS to the html directory
sudo mount -t nfs4 -o \
nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport \
"$EFS_DNS_NAME":/ /var/www/html

# Install Apache web server
sudo yum install -y httpd
sudo systemctl enable httpd
sudo systemctl start httpd

# Install PHP 8 and necessary extensions
sudo dnf install -y \
php \
php-cli \
php-cgi \
php-curl \
php-mbstring \
php-gd \
php-mysqlnd \
php-gettext \
php-json \
php-xml \
php-fpm \
php-intl \
php-zip \
php-bcmath \
php-ctype \
php-fileinfo \
php-openssl \
php-pdo \
php-tokenizer

# Install MySQL 8 community repository
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm

# Install MySQL server
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server

# Start and enable MySQL server
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Set permissions
sudo usermod -a -G apache ec2-user
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
sudo find /var/www -type f -exec sudo chmod 0664 {} \;
sudo chown apache:apache -R /var/www/html

# Download WordPress files
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
sudo cp -r wordpress/* /var/www/html/

# Create the wp-config.php file
sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php

# Edit the wp-config.php file with your database credentials
sudo vi /var/www/html/wp-config.php

# Restart the web server
sudo systemctl restart httpd
```

## Auto Scaling Launch Template Script

```bash
#!/bin/bash
# Update the software packages on the EC2 instance
sudo yum update -y

# Install Apache web server
sudo yum install -y httpd
sudo systemctl enable httpd
sudo systemctl start httpd

# Install PHP 8 and necessary extensions
sudo dnf install -y \
php \
php-cli \
php-cgi \
php-curl \
php-mbstring \
php-gd \
php-mysqlnd \
php-gettext \
php-json \
php-xml \
php-fpm \
php-intl \
php-zip \
php-bcmath \
php-ctype \
php-fileinfo \
php-openssl \
php-pdo \
php-tokenizer

# Install MySQL 8 community repository
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm

# Install MySQL server
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server

# Start and enable MySQL server
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Set the EFS DNS name (replace with your EFS DNS)
EFS_DNS_NAME=fs-0865fb2a73d784996.efs.ap-southeast-2.amazonaws.com

# Mount the EFS to the html directory via fstab
echo "$EFS_DNS_NAME:/ /var/www/html nfs4 \
nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 0 0" | sudo tee -a /etc/fstab

sudo mount -a

# Set permissions
sudo chown apache:apache -R /var/www/html

# Restart the web server
sudo systemctl restart httpd
```


## GitHub Repository

All deployment scripts, configuration files, and the reference diagram are stored in the GitHub repository for this project.

## Notifications

- Enabled SNS (Simple Notification System) for real-time alerts about scaling activities.

## Conclusion

This project demonstrates the deployment of a scalable and secure WordPress website on AWS using industry-standard best practices for DevOps and cloud architecture. The implementation ensures high availability, fault tolerance, and performance, making it suitable for production environments.

