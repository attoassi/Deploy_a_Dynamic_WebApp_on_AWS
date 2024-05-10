# Dynamic Web Application deployment on AWS

This project guidelines the deployment and hosting of a dynamic web application on AWS, utilizing various services and components to ensure high availability, scalability, security, and fault tolerance.

# Architecture Overview

The web application is hosted on EC2 instances within a Virtual Private Cloud (VPC) configured with public and private subnets spanning two Availability Zones. The infrastructure leverages the following AWS resources:

- **Virtual Private Cloud (VPC)**: A logically isolated section of the AWS cloud where AWS resources are launched.
- **Internet Gateway**: Enables communication between the VPC instances and the internet.
- **Security Groups**: Act as virtual firewalls to control inbound and outbound traffic.
- **Availability Zones**: Multiple Availability Zones are used to increase reliability and fault tolerance.
- **Public Subnets**: Host infrastructure components like the NAT Gateway and Application Load Balancer.
- **Private Subnets**: Host the web servers (EC2 instances) and data base (RDS instances) for enhanced security.
- **NAT Gateway**: Allows instances in private subnets to access the internet.
- **EC2 Instance Connect Endpoint**: Enables secure connections to EC2 instances within both public and private subnets.
- **Application Load Balancer**: Distributes incoming web traffic across multiple EC2 instances in an Auto Scaling group.
- **Auto Scaling Group**: Automatically manages the EC2 instances hosting the website, ensuring availability, scalability,  fault tolerance, and elasticity.
- **RDS instances**: used for deploying the data base
- **AWS Certificate Manager**: Secures application communications with SSL/TLS certificates.
- **Simple Notification Service (SNS)**: Sends notifications about activities within the Auto Scaling Group.
- **CloudWatch**:CloudWatch to monitor key metric (CPU , memory and disk space) and CloudWatch logs for logging 
- **Route 53**: Provides DNS services for registering and managing the website's domain name.
- **S3 Bucket**: Stores the application code and assets.

# Deployment script

#!/bin/bash

# This command updates all the packages on the server to their latest versions
sudo yum update -y

# This series of commands installs the Apache web server, enables it to start on boot, and then starts the server immediately
sudo yum install -y httpd
sudo systemctl enable httpd 
sudo systemctl start httpd

# This command installs PHP along with several necessary extensions for the application to run
sudo dnf install -y \
php \
php-pdo \
php-openssl \
php-mbstring \
php-exif \
php-fileinfo \
php-xml \
php-ctype \
php-json \
php-tokenizer \
php-curl \
php-cli \
php-fpm \
php-mysqlnd \
php-bcmath \
php-gd \
php-cgi \
php-gettext \
php-intl \
php-zip

## These commands Installs MySQL version 8
# Install the MySQL Community repository
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm 

# Install the MySQL server
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server 

# Start and enable the MySQL server
sudo systemctl start mysqld
sudo systemctl enable mysqld

# This command enables the 'mod_rewrite' module in Apache on an EC2 Linux instance. It allows the use of .htaccess files for URL rewriting and other directives in the '/var/www/html' directory
sudo sed -i '/<Directory "\/var\/www\/html">/,/<\/Directory>/ s/AllowOverride None/AllowOverride All/' /etc/httpd/conf/httpd.conf

# Environment Veriable
S3_BUCKET_NAME=atto-shopwise-project-webfiles 

# This command downloads the contents of the specified S3 bucket to the '/var/www/html' directory on the EC2 instance
sudo aws s3 sync s3://"$S3_BUCKET_NAME" /var/www/html

# This command changes the current working directory to '/var/www/html', which is the standard directory for hosting web pages on a Unix-based server
cd /var/www/html

# This command is used to extract the contents of the application code zip file that was previously downloaded from the S3 bucket
sudo unzip shopwise.zip

# This command recursively copies all files, including hidden ones, from the 'shopwise' directory to the '/var/www/html/'
sudo cp -R shopwise/. /var/www/html/

# This command permanently deletes the 'shopwise' directory and the 'shopwise.zip' file.
sudo rm -rf shopwise shopwise.zip

# This command set permissions 777 for the '/var/www/html' directory and the 'storage/' directory
sudo chmod -R 777 /var/www/html
sudo chmod -R 777 storage/

# This command will open th vi editor and allow you to edit the .env file to add your database credentials 
sudo vi .env

# This command will restart the Apache server
sudo service httpd restart

## Usage

1. Clone the GitHub repository to your local machine.
2. Follow the instructions in the repository's documentation to set up the required AWS resources.
3. Deploy the website code and assets to the appropriate S3 bucket.
4. Access the website using the registered domain name.
