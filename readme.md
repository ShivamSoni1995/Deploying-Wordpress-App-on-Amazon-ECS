# WordPress Deployment on AWS ECS Fargate 

 ## Description 

 This repository contains configuration and guidance for deploying a scalable and highly available WordPress instance using AWS Elastic Container Service (ECS) with the Fargate launch type. It leverages AWS Application Load Balancer (ALB) for traffic distribution and AWS Relational Database Service (RDS) for the MySQL database backend. This setup follows best practices for containerized deployments on AWS, simplifying management by using serverless container infrastructure. 

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

 * **AWS ECS (Fargate):** Hosts the WordPress container without needing to manage underlying EC2 instances. 
 * **AWS Application Load Balancer (ALB):** Distributes incoming web traffic across the running WordPress container tasks. 
 * **AWS RDS (MySQL):** Provides a managed MySQL database for WordPress data storage. 
 * **AWS VPC:** Networking infrastructure to host the resources. 
 * **AWS Secrets Manager (Optional but Recommended):** For managing database credentials securely. 

 ## Prerequisites 

 * An AWS Account. 
 * AWS CLI (optional, for command-line operations). 
 * Git installed locally. 
 * Familiarity with AWS Console, ECS, RDS, and VPC concepts. 

 ## Deployment Steps 

 Follow these steps to deploy the WordPress application: 

 ### 1. Create ECS Cluster 

 * Create an ECS Cluster using the AWS Management Console or AWS CLI. 
 * Choose the **AWS Fargate** compute option. 

 ### 2. Create RDS MySQL Instance 

 * Launch an RDS instance using the MySQL engine. 
 * **Important:**     * Deploy the RDS instance within the same VPC as your ECS Cluster. 
     * Configure the RDS instance security group to allow inbound traffic on port 3306 (MySQL default) from the ECS Service's security group. 
     * Disable public access for security. 
     * Note down the database endpoint, username, and password (consider using AWS Secrets Manager for the password). 
     * Specify an initial database name during creation. 

 ### 3. Create ECS Task Definition 

 * Define how the WordPress container should run. 
 * Use the public `wordpress` Docker image or specify your own image (e.g., from ECR). 
 * Specify CPU and Memory requirements (e.g., 1 vCPU, 3 GB Memory). 
 * Configure port mapping (e.g., map container port 80 to host port 80). 
 * Set the Network Mode to `awsvpc` (required for Fargate). 
 * Assign an `ecsTaskExecutionRole` to allow ECS to pull images and log to CloudWatch. 
 * You can create this using the AWS Console UI or by providing a JSON definition (see example below). 

 ### 4. Create ECS Service 

 * Within your ECS Cluster, create a new Service. 
 * Select the **Fargate** launch type. 
 * Choose the Task Definition created in the previous step. 
 * Set the desired number of tasks (e.g., 2 for high availability). 
 * Configure networking: 
     * Select your VPC. 
     * Choose subnets across multiple Availability Zones for high availability. 
     * Create or select a security group that allows inbound traffic on port 80 from the ALB. 
 * Configure an Application Load Balancer (ALB): 
     * Create a new ALB or select an existing one. 
     * Set up a listener for HTTP on port 80. 
     * Create a target group that points to the IP addresses of the tasks, using port 80 and the HTTP protocol. Ensure health checks are configured appropriately. 
 * Review and create the service. It may take a few minutes for the tasks to start and register with the ALB. 

 ### 5. Connect WordPress to Database 

 * Once the service is running, access the WordPress setup page using the ALB's DNS name. 
 * When prompted, enter the database details gathered during the RDS setup: 
     * Database Name 
     * Username 
     * Password (retrieved from Secrets Manager if used) 
     * Database Host (the RDS instance endpoint) 
 * Run the WordPress installation. 

 ## Accessing the Application 

 After completing the WordPress setup, you can access your site using the DNS name of the Application Load Balancer. You can also map a custom domain name to the ALB DNS name using Route 53 or your DNS provider. 

 ## Configuration Details 

 ### Task Definition Example 

 Here is an example JSON structure for the Task Definition (remember to replace `<AWS_ACCOUNT_ID>` with your actual AWS Account ID): 

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