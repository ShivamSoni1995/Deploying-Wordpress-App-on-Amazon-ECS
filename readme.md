
# WordPress Deployment on AWS ECS Fargate

## Description

This repository contains configuration and guidance for deploying a scalable and highly available WordPress instance using AWS Elastic Container Service (ECS) with the Fargate launch type[cite: 1, 4, 5, 40]. It leverages AWS Application Load Balancer (ALB) for traffic distribution and AWS Relational Database Service (RDS) for the MySQL database backend[cite: 18, 19, 47, 52]. This setup follows best practices for containerized deployments on AWS, simplifying management by using serverless container infrastructure[cite: 5, 71, 73].

## Table of Contents

* [Architecture](#architecture)
* [Prerequisites](#prerequisites)
* [Deployment Steps](#deployment-steps)
    * [1. Create ECS Cluster](#1-create-ecs-cluster)
    * [2. Create RDS MySQL Instance](#2-create-rds-mysql-instance)
    * [3. Create ECS Task Definition](#3-create-ecs-task-definition)
    * [4. Create ECS Service](#4-create-ecs-service)
    * [5. Connect WordPress to Database](#5-connect-wordpress-to-database)
* [Accessing the Application](#accessing-the-application)
* [Configuration Details](#configuration-details)
    * [Task Definition Example](#task-definition-example)
    * [Networking](#networking)
* [Contributing](#contributing)
* [License](#license)
* [Acknowledgements](#acknowledgements)

## Architecture

The deployment utilizes the following AWS services:

* **AWS ECS (Fargate):** Hosts the WordPress container without needing to manage underlying EC2 instances[cite: 3, 14, 40].
* **AWS Application Load Balancer (ALB):** Distributes incoming web traffic across the running WordPress container tasks[cite: 18, 47].
* **AWS RDS (MySQL):** Provides a managed MySQL database for WordPress data storage[cite: 19, 52, 55].
* **AWS VPC:** Networking infrastructure to host the resources[cite: 44, 52].
* **AWS Secrets Manager (Optional but Recommended):** For managing database credentials securely[cite: 56, 66].

## Prerequisites

* An AWS Account.
* AWS CLI (optional, for command-line operations).
* Git installed locally.
* Familiarity with AWS Console, ECS, RDS, and VPC concepts.

## Deployment Steps

Follow these steps to deploy the WordPress application:

### 1. Create ECS Cluster

* Create an ECS Cluster using the AWS Management Console or AWS CLI.
* Choose the **AWS Fargate** compute option[cite: 16].

### 2. Create RDS MySQL Instance

* Launch an RDS instance using the MySQL engine[cite: 55].
* **Important:**
    * Deploy the RDS instance within the same VPC as your ECS Cluster[cite: 52].
    * Configure the RDS instance security group to allow inbound traffic on port 3306 (MySQL default) from the ECS Service's security group[cite: 53, 61].
    * Disable public access for security[cite: 59].
    * Note down the database endpoint, username, and password (consider using AWS Secrets Manager for the password)[cite: 56, 65, 66].
    * Specify an initial database name during creation[cite: 63].

### 3. Create ECS Task Definition

* Define how the WordPress container should run[cite: 8, 26, 27].
* Use the public `wordpress` Docker image or specify your own image (e.g., from ECR)[cite: 17, 29].
* Specify CPU and Memory requirements (e.g., 1 vCPU, 3 GB Memory)[cite: 33].
* Configure port mapping (e.g., map container port 80 to host port 80)[cite: 33].
* Set the Network Mode to `awsvpc` (required for Fargate)[cite: 33].
* Assign an `ecsTaskExecutionRole` to allow ECS to pull images and log to CloudWatch[cite: 33, 34, 37].
* You can create this using the AWS Console UI or by providing a JSON definition (see example below)[cite: 28, 33].

### 4. Create ECS Service

* Within your ECS Cluster, create a new Service[cite: 24].
* Select the **Fargate** launch type[cite: 40].
* Choose the Task Definition created in the previous step.
* Set the desired number of tasks (e.g., 2 for high availability)[cite: 42].
* Configure networking:
    * Select your VPC[cite: 44].
    * Choose subnets across multiple Availability Zones for high availability[cite: 44].
    * Create or select a security group that allows inbound traffic on port 80 from the ALB[cite: 45, 46].
* Configure an Application Load Balancer (ALB):
    * Create a new ALB or select an existing one.
    * Set up a listener for HTTP on port 80[cite: 47].
    * Create a target group that points to the IP addresses of the tasks, using port 80 and the HTTP protocol. Ensure health checks are configured appropriately.
* Review and create the service. It may take a few minutes for the tasks to start and register with the ALB[cite: 48].

### 5. Connect WordPress to Database

* Once the service is running, access the WordPress setup page using the ALB's DNS name[cite: 49, 50].
* When prompted, enter the database details gathered during the RDS setup:
    * Database Name [cite: 68]
    * Username [cite: 68]
    * Password (retrieved from Secrets Manager if used) [cite: 66, 68]
    * Database Host (the RDS instance endpoint) [cite: 68]
* Run the WordPress installation[cite: 68].

## Accessing the Application

After completing the WordPress setup, you can access your site using the DNS name of the Application Load Balancer[cite: 49, 69, 70]. You can also map a custom domain name to the ALB DNS name using Route 53 or your DNS provider[cite: 21].

## Configuration Details

### Task Definition Example

Here is an example JSON structure for the Task Definition (remember to replace `<AWS_ACCOUNT_ID>` with your actual AWS Account ID)[cite: 33]:

```json
{
  "requiresCompatibilities": [
    "FARGATE"
  ],
  "family": "ecs-demo-wordpress",
  "containerDefinitions": [
    {
      "name": "ecs-demo-wordpress",
      "image": "wordpress",
      "essential": true,
      "portMappings": [
        {
          "name": "http",
          "containerPort": 80,
          "hostPort": 80,
          "protocol": "tcp",
          "appProtocol": "http"
        }
      ]
    }
  ],
  "volumes": [],
  "networkMode": "awsvpc",
  "memory": "3 GB",
  "cpu": "1 vCPU",
  "executionRoleArn": "arn:aws:iam::<AWS_ACCOUNT_ID>:role/ecsTaskExecutionRole"
}
```

### Networking

* **Security Groups:**
    * ALB Security Group: Allow inbound traffic on port 80 (HTTP) or 443 (HTTPS) from the internet (0.0.0.0/0).
    * ECS Service Security Group: Allow inbound traffic on port 80 from the ALB Security Group[cite: 46]. Allow outbound traffic as needed.
    * RDS Security Group: Allow inbound traffic on port 3306 from the ECS Service Security Group[cite: 53, 61].

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.
(Add more specific contribution guidelines if applicable).

## License

(Specify the license under which this project is shared, e.g., MIT, Apache 2.0, or leave as placeholder).

## Acknowledgements

Based on the guide "Streamlining Containerized Deployments: A Comprehensive Guide to AWS ECS and WordPress Integration" by Shivam Soni[cite: 1].
```

Remember to replace placeholders, review the security group rules for your specific needs, and add any specific configuration details relevant to your repository's contents (like CI/CD pipeline information, environment variable setup, etc.)[cite: 74].
