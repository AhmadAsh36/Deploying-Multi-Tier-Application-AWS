![Alt text](AWS project.png )


---

# Deploying a Multi-Tier Web Application on AWS

## Overview

This project demonstrates how to deploy a highly available, fault-tolerant, and scalable multi-tier web application on AWS using EC2 instances, Elastic Load Balancer (ELB), security groups, auto-scaling, and other AWS services. The multi-tier architecture consists of a frontend hosted on Apache Tomcat, a backend with MySQL, Memcached, and RabbitMQ, and an S3 bucket for storing the application artifacts.

The infrastructure is automated using EC2 user-data scripts for setting up services like RabbitMQ, MySQL, Memcached, and Tomcat.

## Architecture

The application is designed with the following tiers:

1. **Frontend Tier**:
   - **Tomcat**: Serves the web application via HTTP(S).
   - **ELB**: Distributes traffic to Tomcat instances.
   
2. **Backend Tier**:
   - **MySQL**: Database server.
   - **Memcached**: Caching layer for improved performance.
   - **RabbitMQ**: Message queue service.

## Resources Utilized

- **EC2 Instances**: For running MySQL, Memcached, RabbitMQ, and Tomcat.
- **ELB (Elastic Load Balancer)**: Distributes incoming traffic to Tomcat instances.
- **S3 Bucket**: Stores the application artifacts.
- **IAM**: Provides secure access to S3 bucket and EC2 instances.
- **Route 53**: Maps domain names to application endpoints.

## Steps to Deploy the Application

### 1. **Create Security Groups**
   - **ELB Security Group**: Allow inbound HTTPS traffic.
   - **Apache Tomcat Security Group**: Allow inbound traffic on port 8080, but only from the ELB.
   - **Backend Services Security Group**: Allow access for MySQL, Memcached, and RabbitMQ services.

### 2. **Create Key Pairs for EC2 Instances**
   - Generate key pairs for SSH access to the EC2 instances.

### 3. **Launch EC2 Instances**
   - Launch EC2 instances for MySQL, Memcached, RabbitMQ, and Apache Tomcat.
   - Assign the relevant security groups and configure instance roles.

### 4. **Run User-Data Scripts**
   - Attach the following user-data scripts to the EC2 instances to automate the setup of required services.

#### RabbitMQ Setup Script
```bash
#!/bin/bash
## Install RabbitMQ
rpm --import 'https://github.com/rabbitmq/signing-keys/releases/download/3.0/rabbitmq-release-signing-key.asc'
rpm --import 'https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-erlang.E495BB49CC4BBE5B.key'
rpm --import 'https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-server.9F4587F226208342.key'
curl -o /etc/yum.repos.d/rabbitmq.repo https://raw.githubusercontent.com/hkhcoder/vprofile-project/aws-LiftAndShift/al2023rmq.repo
dnf update -y
dnf install -y erlang rabbitmq-server
systemctl enable rabbitmq-server
systemctl start rabbitmq-server
sudo sh -c 'echo "[{rabbit, [{loopback_users, []}]}]." > /etc/rabbitmq/rabbitmq.config'
sudo rabbitmqctl add_user test test
sudo rabbitmqctl set_user_tags test administrator
sudo systemctl restart rabbitmq-server
```

#### MySQL Setup Script
```bash
#!/bin/bash
DATABASE_PASS='admin123'
sudo yum update -y
sudo dnf install mariadb105-server -y
sudo systemctl start mariadb
sudo systemctl enable mariadb
git clone -b main https://github.com/hkhcoder/vprofile-project.git
sudo mysqladmin -u root password "$DATABASE_PASS"
sudo mysql -u root -p"$DATABASE_PASS" -e "ALTER USER 'root'@'localhost' IDENTIFIED BY '$DATABASE_PASS'"
sudo mysql -u root -p"$DATABASE_PASS" -e "create database accounts"
sudo mysql -u root -p"$DATABASE_PASS" -e "grant all privileges on accounts.* TO 'admin'@'localhost' identified by 'admin123'"
sudo mysql -u root -p"$DATABASE_PASS" accounts < /tmp/vprofile-project/src/main/resources/db_backup.sql
```

#### Memcached Setup Script
```bash
#!/bin/bash
sudo dnf install memcached -y
sudo systemctl start memcached
sudo systemctl enable memcached
sed -i 's/127.0.0.1/0.0.0.0/g' /etc/sysconfig/memcached
sudo systemctl restart memcached
sudo systemctl enable firewalld
firewall-cmd --add-port=11211/tcp --permanent
firewall-cmd --add-port=11111/udp --permanent
sudo systemctl restart firewalld
sudo memcached -p 11211 -U 11111 -u memcached -d
```

#### Tomcat Setup Script
```bash
#!/bin/bash
sudo apt update
sudo apt upgrade -y
sudo apt install openjdk-11-jdk -y
sudo apt install tomcat9 tomcat9-admin tomcat9-docs tomcat9-common git -y
```

### 5. **Validate Scripts**
   - SSH into the EC2 instances and verify that the services (RabbitMQ, MySQL, Memcached, Tomcat) are running correctly.

### 6. **Update Route 53 for Domain Mapping**
   - Update IP-to-name mapping in Route 53 for the application domain name.

### 7. **Build the Application Artifact**
   - Build the artifact of the web application from the source code repository.

### 8. **S3 Bucket Setup**
   - Create an IAM user with permissions to upload and access the S3 bucket.
   - Upload the application artifact to the S3 bucket.

### 9. **IAM Role Setup**
   - Create IAM roles to allow EC2 instances to fetch the artifact from the S3 bucket.

### 10. **Configure ELB**
   - Set up an ELB to distribute traffic to Tomcat EC2 instances.
   - Configure the ELB to use HTTPS with Amazon Certificate Manager for SSL termination.

### 11. **DNS Configuration in GoDaddy**
   - Map the ELB endpoints to the website domain name in GoDaddy’s DNS settings.

### 12. **Enable Auto Scaling**
   - Set up an auto-scaling group to ensure the EC2 instances scale dynamically based on traffic load.

## Final Architecture Diagram

- [Insert reference diagram here, showing the multi-tier architecture with EC2 instances for Tomcat, MySQL, RabbitMQ, Memcached, ELB, Route 53, and S3]

## Conclusion

This project demonstrates how to create a robust, scalable, and highly available multi-tier web application on AWS. By utilizing EC2, ELB, IAM, S3, and Route 53, we ensured the application was both secure and scalable. The use of auto-scaling and proper security configurations guarantees fault tolerance and optimal performance.


---

This README provides a comprehensive guide for deploying a multi-tier web application on AWS.
