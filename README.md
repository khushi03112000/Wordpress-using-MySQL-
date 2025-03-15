# Deploying WordPress with MySQL on AWS Using Docker
## Project Overview
This project demonstrates the deployment of a WordPress application and MySQL database using Docker on AWS. The WordPress instance is hosted in a **public subnet** with an **Elastic IP** for consistent access, while the MySQL database is secured in a **private subnet** to prevent external exposure.

## Architecture
1. **WordPress** - Runs inside a **Docker container** on an EC2 instance in the **public subnet**.
2. **MySQL Database** - Runs inside a **Docker container** on an EC2 instance in the **private subnet**.
3. **Networking** - Uses **security groups** and **subnet routing** to allow communication between the WordPress and MySQL containers.
4. **Elastic IP** - Ensures that WordPress is always accessible with a static public IP.
5. **Custom AMIs** - Used to speed up deployments and ensure repeatability.
6. **Startup Scripts** - Automate the setup process when new instances are launched.

## Prerequisites
- AWS account to create VPC, EC2, and Security Groups
- SSH key pair for EC2 access

## Deployment Steps

### Step 1: Create VPC and Subnets
- Create a **VPC** with a CIDR block (10.0.0.0/16).
- Create two **subnets**:
  - **Public Subnet** (10.0.0.0/20) for WordPress.
  - **Private Subnet** (10.0.128.0/20) for MySQL.
- Set up an **Internet Gateway** and route public traffic correctly.
![image](https://github.com/user-attachments/assets/fdbe0639-9dc5-4f98-b207-65c775d9be6a)
![image](https://github.com/user-attachments/assets/d950f373-c381-49d9-99c2-5684712612a9)
![image](https://github.com/user-attachments/assets/e2c68b1e-cc6d-49d2-8760-98e07d373a62)

### Step 2: Configure Security Groups
- **WordPress EC2 Security Group**
  - Allow **HTTP (80)** and **SSH (22)** from anywhere.
  - Allow **MySQL (3306)** access only from the private subnet.
- **MySQL EC2 Security Group**
  - Allow **MySQL (3306)** access only from the public subnet.

![image](https://github.com/user-attachments/assets/4e26ed2c-6b10-4270-80ad-f9903f0624ef)
![image](https://github.com/user-attachments/assets/c60cd2d8-62ca-4c8d-8a50-f760858d7eeb)


### Step 3: Launch EC2 Instances
- Launch two EC2 instances using **Ubuntu AMI**.
- Assign **Elastic IP** to the WordPress instance.(Elastic IP comes with a cost hence I have used normal IPs for demo purpose)
- Place WordPress EC2 in the **public subnet** and MySQL EC2 in the **private subnet**.

![image](https://github.com/user-attachments/assets/f3f1cedf-33af-4374-b4a0-3f180a0459d4)
![image](https://github.com/user-attachments/assets/0feffb08-4088-4995-9ec0-01573f6c99be)


### Step 4: Install Docker and Deploy Containers
#### Install Docker on Wordpress EC2 Instance. Learning : Since MySQL EC2 instance has been launched using AMI of Wordpress EC2 so it has docker and mysql docker image pre-installed.
Run the following commands on Wordpress EC2 instances:
```bash
sudo apt update -y
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
```
![image](https://github.com/user-attachments/assets/f21d8ca3-6388-40a8-bca8-cc5ad1057d2a)
![image](https://github.com/user-attachments/assets/7cb782d8-8074-42bf-8a4b-7f06637dc467)


#### Deploy MySQL Container (Private EC2)
```bash
docker run -d \
  --name mysql-container \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=wordpress \
  -e MYSQL_USER=wordpress \
  -e MYSQL_PASSWORD=password \
  -p 3306:3306 \
  mysql:latest
```
![image](https://github.com/user-attachments/assets/c348d364-e4aa-40a4-a72e-4133d6b6589b)


#### Deploy WordPress Container (Public EC2)
```bash
docker run -d \
  --name wordpress-container \
  -e WORDPRESS_DB_HOST=10.0.2.100:3306 \
  -e WORDPRESS_DB_USER=wordpress \
  -e WORDPRESS_DB_PASSWORD=password \
  -e WORDPRESS_DB_NAME=wordpress \
  -p 80:80 \
  wordpress:latest
```
![image](https://github.com/user-attachments/assets/4fbecd9d-e1c1-4590-8143-aa353391904c)

### Step 5: Verify Connection Between WordPress and MySQL
From the WordPress EC2 instance, test the MySQL connection:
```bash
mysql -u wordpress -p -h 10.0.136.111 -D wordpress
```
![image](https://github.com/user-attachments/assets/c4c0b5d7-5ec0-46fa-8ff5-633313c5b693)


### Step 6: Access WordPress
- Open the **IP** of the WordPress EC2 instance in a browser.
- Complete the WordPress setup.

📸 **Snapshot Required:** WordPress setup page in a browser.

## Automating with Custom AMIs
- Create an **AMI** from the configured EC2 instances.
- Use the AMIs to launch pre-configured WordPress and MySQL instances in future deployments.

📸 **Snapshot Required:** Screenshot of created AMIs in AWS.

## Automating with Startup Scripts
Use the following **user data** scripts when launching EC2 instances:

### WordPress Instance Script
```bash
#!/bin/bash
apt update -y
apt install docker.io -y
systemctl start docker
systemctl enable docker
docker run -d \
  --name wordpress-container \
  -e WORDPRESS_DB_HOST=10.0.2.100:3306 \
  -e WORDPRESS_DB_USER=wordpress \
  -e WORDPRESS_DB_PASSWORD=password \
  -e WORDPRESS_DB_NAME=wordpress \
  -p 80:80 \
  wordpress:latest
```

### MySQL Instance Script
```bash
#!/bin/bash
apt update -y
apt install docker.io -y
systemctl start docker
systemctl enable docker
docker run -d \
  --name mysql-container \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=wordpress \
  -e MYSQL_USER=wordpress \
  -e MYSQL_PASSWORD=password \
  -p 3306:3306 \
  mysql:latest
```
📸 **Snapshot Required:** Console output when launching instances with user data.


## Conclusion
This setup ensures a **secure**, **scalable**, and **repeatable** deployment of WordPress and MySQL using Docker on AWS. The architecture improves security by keeping the database private and enhances availability by using an Elastic IP.

📸 **Final Snapshot Required:** Screenshot of the working WordPress homepage.

---

### 🚀 **Next Steps**
- Automate the deployment with Terraform.
- Use ECS and RDS for better scalability.
- Implement monitoring using CloudWatch.



