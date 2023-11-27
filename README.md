# Step 1: Set Up AWS Infrastructure
  setup the networking components
  I will provision:
- 1 VPC
- 2 Public Subnets us-east-1 
- 2 private Subnet us-east-2
- 2 Route tables
- 1 internet gateway
- 3 Security Groups
   - ALB Security Group: open port 443(HTTPS), Destination internet.
   - ECS Task Security Group: open ports: 8000 and 8025 >> Destination, ALB Sg.
   - RDS PostreSQL Security Group: open port 5432 >> Destination, ECS Task Sg.
# Create an RDS PostgreSQL Database
 - Provision a RDS postgres database in AWS RDS service which is a manage service that is highly available out of the box.
 - Configure a multi AZ deployment for high availability and fault tolerant.
 - Ensure to update the database environmental variables in the .env.example file.
 - I will use a private subnet to deploy the database
 - Attach the RDS PostgreSQL Security group to the database which only allow traffic from the ECS task security group.
 - Ensure to store the database credential in SSM parameter store.
# Create an ECR Repo, Build the Docker images and push it to ECR
 - Create an ECR repo to store the docker images
 - Install AWS CLI, docker and docker compose on my local machine.
 - Clone the repo to my local machine and the build the images
 - Push the  docker images to ECR
  
# Step 2: Create AWS ECS Cluster and Task Definition and ECS service 
  - Setup ECS Cluster to manage the EC2 instances that will run the docker container.
  - I will setup Auto Scaling group in the cluster to launch and terminal instances base on a predefined rule.
    - ECS Cluster Role
    - AMI
    - instance type
    - Desire Capacity (minimum - 2, maximum - 4)
    - Security group 
    - VPC
    - Subnets
    - Enable CloudWatch container insight to collect matric and logs.
# Task definition
 - Create Task definition for the Django app and the mailhog server.
 - Utilize the specific docker image for each container in the task definition.
 - Create a task definition for each of the container by using the specific docker image inside the ECR Repo.
# ECS Service
- I will create a service for the cluster
- launch type is EC2 
- I will specify the Task definition that i created above
- Rolling or blue/green Deployment
- I will configure the load balance for my service
- i will enable health check
once the service is created, the task will be deploy.

# Step 3: Setup a LoadBalancer, TLS/SSL Certificate and a DNS service:
 I will configure the ALB to distribute incoming traffic across the ECS tasks
 I will attach the ALB security group and also configure the TLS/SSL certificate to enable secure HTTPS traffic.
- TLS/SSL certificate
- DNS
##### For Infrastructure Provisioning and Management
I will provision all of this Infrastructure leveraging Terraform and i will make sure to utilize Terraform feature call Modules.
Step 1 will be in a separate module as well as step 2 and 3.
# Step 7: For Security,
 - Enable encryption at rest as well as in Transit using KMS and SSL/TLS
 - Enable backup of data by taking automatic snapshot with AWS Data Lifecycle manager.
 - for IAM users and roles, i will leverage the principle of least privilege that means i will only give fine grain access to services and users that need access to the application.


# Step 8: Post Deployment,
i will setup Cloudwatch which will give me insight base on health and performance of my application by monitoring and collecting logs.
i will develop a custom CloudWatch Alarms for proactive monitoring.
I can also set up SNS for notification.


# For CICD Implementation Using Jenkins 
I will utilize Jenkins, a powerful automation service that enables you to orchestrate all stages in your pipeline, 
from continuous integration all the way to continuous deployment.
#### NOTE, i will use the Terraform template that i mention in the infrastructure setup
# Step 1
i will create a new Virtual Machine and ssh into the machine
i will install:
- Jenkins,
- Git,
- Docker,
- Docker-compose,
- AWS CLI,
- Terraform
I will create 2 jenkins pipeline, one to automate infrastructure provisioning and the second for automatic build and deployment of the docker images.
to configure the jenkins, i will login to the service using the public ip of my virtual machine and the default jenkins port number which is 8080.
i will setup the necessary plugin inside my dashboard. then i will create a pipeline job.
I will also use Jenkins shared libraries to manage my pipeline code.
# First Pipeline Script
- stage 1: Checkout the code
- stage 2: Confirm the terraform version (Terraform --version)
- stage 3: initialize the repository (Terraform init)
- stage 4: To check for syntax error (Terraform fmt)
- stage 5: I will do a dry run (Terraform plan)
- stage 5: Provision the infrastructure (Terraform apply --auto-approve)
- lastly i can integrate slack to notify me if there is an error.
# Second Pipeline Script
- Stage 1: check out the code into the Jenkins machine
- Stage 2: Build the docker images
- Stage 3: Authenticate and push docker images to ECR
- Stage 4: Update the ECS Task definition with the latest image  `aws ecs register-task-definition`
- Stage 5: Update the ECS service with the latest task definition `aws ecs update-service`
- Stage 6: Deploy the application with AWS CLI command, `aws ecs run-task`
post deployment, integrate slack to notify me if there is an error.



# Follow up Questions and Answers

1. Explain the open port choices for a production environment.
The open port choices for a production environment are carefully selected to minimize the attack surface and protect the application from unauthorized access.
 In this solution, the open ports are:
- Port 443 (HTTPS): This port is used for secure communication between the ALB and the client browsers. HTTPS encrypts all traffic, protecting sensitive data from eavesdropping and tampering.

- Ports 8000 and 8025: These ports are used for internal communication between the ALB and the ECS tasks running the Django application. These ports are only accessible from within the VPC, ensuring that they are not exposed to the public internet.

- Port 5432: This port is used for communication between the ECS tasks running the Django application and the RDS PostgreSQL database. This port is only accessible from the ECS task security group, ensuring that only authorized containers can access the database.

2. Where is the database running and how will the data stay in sync?

The database is running on an RDS PostgreSQL instance in a multi-AZ deployment. This means that the database is replicated across multiple Availability Zones, ensuring that it remains available even if one Availability Zone experiences an outage. The RDS instance also provides automatic backups, ensuring that data loss is minimized in case of accidental deletion or corruption.

3. Explain the choice to use Docker on EC2 vs ECS?

In the first solution, I used an EC2 instance because it was a simple and small deployment. One advantage of using EC2 over ECS is the ability to have fine-grained access to the underlying infrastructure. This becomes crucial if you have strict compliance regulations that require the installation of specific licenses in your environment.

If the deployment were more complex, I would have opted for ECS, which is a container orchestration tool and managed service providing capabilities such as auto-scaling, auto-healing, and support for load balancing. I have used ECS for the new solution.

4. How will the docker containers persist between restarts of the EC2 instances?

The Docker containers will persist between restarts of the EC2 instances using the ECS task definition and service configuration. The ECS task definition defines the containers that should be running, and the ECS service manages the lifecycle of those containers. When an EC2 instance restarts, the ECS service will automatically relaunch the containers defined in the task definition.

5. In your opinion, would this setup be considered high availability?

The first solution was partially available, but the new solution would be considered high availability. The use of a multi-AZ RDS database instance, an Auto Scaling group for the ECS cluster, and health checks for the ALB and ECS tasks provide a resilient and fault-tolerant infrastructure. The CI/CD pipeline further enhances high availability by automating the deployment process, ensuring that the application can be quickly restored in case of a failure.




