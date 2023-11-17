# Step 1: Set Up AWS Infrastructure
  setup the networking components
  i will provision:
- 1 VPC
- 2 Public Subnets (us-east-1 and us-east-2)
- Route tables
- internet gateway
- Security Groups open to ports (8000, 22, 443, 80, 8025 and 5432)
# Step 2: Create 2 Virtual Machines 
  Setup two EC2 instances in us-east-1 and us-east-2
- Amazon Linux 2
- instance type
- Security group
- key pair
- User data, to install docker and docker-compose by bootstrapping my EC2 instances.
# Step 3: Setup a LoadBalancer, TLS/SSL Certificatte and a DNS service:
 I will provision a loadBalancer, 
- Target Group
- TLS/SSL certificate
- DNS
##### For Infrastructure Provisioning and Management
I will provision all of this Infrastructure leveraging Terraform and i will make sure to utilize Terraform feature call Modules.
Step 1 will be in a separate module as well as step 2 and 3.

# Step 4: SSH into the virtual machines
 I will ssh into the instances using gitbase or putty
 I will install git, this will enable me to clone the git repo.
 I will add EC2-user to the docker group.
 - sudo yum update
 - sudo yum install git -y
 - sudo usermod -aG docker ec2-user
I will now clone the repo into my virtual machines by running
git clone https://github.com/SilversphereInc/django-devops-arena.git
After that i can run this command
- ./run-devenv.sh 
which will build the docker images and start the application.

# Step 5: For Persistent data Storage,
 I will provision an external volume like an EBS volume
 and then map or mount it to the containers volume for data persistency because containers are ephemeral.

# Step 6: For High Availability, Secure connection and Domain name service.
I will configure the Two Virtual Machine as the Target group and then setup the Application Load balancer to distribute traffic between the Target groups.
I will also configure the ALB with the SSL/TLS certificate for a secure access to the application Using HTTPS.
I will create an DNS A record and map the ALB endpoint to the A record for the DNS service to resolve the ALB endpoint to a custom Domain name.

# Step 7: For Security,
I will make sure to enable encryption at rest as well as in Transit using KMS and SSL/TLS
I will enable backup of data by taking automatic snapshot with AWS Data Lifecycle manager.
for IAM user and roles, i will leverage the principle of least privilege that means i will only give fine grain access to services and users that need access to the application.

# Step 8: Post Deployment,
i will setup Cloudwatch which will give me insight base on health and performance of my application by monitoring and collecting logs.
I can also set up SNS for notification.



# For CICD Implementation Using Jenkins
I will utilize Jenkins, a powerful automation service that enables you to orchestrate all stages in your pipeline, 
from continuous integration all the way to continuous deployment.

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

# First Pipeline Script
stage 1: i will checkout the code
stage 2: I will confirm the terraform version (Terraform --version)
stage 3: i initialize the repository (Terraform init)
stage 4: i will check for syntax error (Terraform fmt)
stage 5: i will do a dry run (Terraform plan)
stage 5: i will create the infrastructure (Terraform apply --auto-approve)
lastly i can integrate slack to notify me if there is an error.

# Second Pipeline Script
I will configure each stage with multiple commands because I'm deploying into two VMs
stage 1: i will checkout the code
stage 2: i will use ssh to copy the Docker-compose into the both EC2 instances.
stage 3: i will create an external volume and then mount it to each container inside each Vms
stage 3: i will run Docker-compose up -d.


