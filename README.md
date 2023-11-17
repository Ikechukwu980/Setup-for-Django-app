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
# Step 3: Setup a LoadBalancer, TLS/SSL Certifiacte and a DNS service:
 I will provision a loadBalancer, 
- Target Group
- TLS/SSL certificate
- DNS

I will provision all of this Infrastructure levegring Terraform and i will make sure to utilize Terraform feature call Modules.
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
which will build the docker images and start the applicaition.

# Step 5: For Persistent data Storage,
 I will provision an external volume like an EBS volume
 and then map or mount it to the containers volume for data persistency because containers are ephemeral.

# Step 6: For High Availability,
I will configure the Two Virtual Subnets as the Target group and then setup the Application Load balancer to distribute traffic between the Target groups.
I will also configurte the ALB with the SSL/TLS certifiacte for a secure access to the application Using HTTPS.

# Step 7: For Security,
I will make sure to enable encription at rest as well as in Transit using KMS and SSL/TLS
I will enable backup of data by taking automatic snapshot with AWS Data Lifecycle manager.
for IAM user and roles, i will levegrage the principle of least privilege that means i will only give fine grain access to services and users that need access to the application.

# Step 8: Post Deployment,
i will setup Cloudwatch which will give me insight base on health and performance of my application by monitoring and collecting logs.
I can also set up SNS for notification


