
# Automated Infrastructure Setup with Terraform



### Prerequisites:

 - Install Terraform
 - AWS CLI configured with necessary permissions


### Terraform Configuration:
### 1. Create a Terraform configuration file (main.tf):


```bash
  provider "aws" {
  region = "us-west-2"
}

resource "aws_vpc" "main_vpc" {
cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "public_subnet" {
vpc_id            = aws_vpc.main_vpc.id
cidr_block        = "10.0.1.0/24"
map_public_ip_on_launch = true
}

resource "aws_subnet" "private_subnet" {
vpc_id            = aws_vpc.main_vpc.id
cidr_block        = "10.0.2.0/24"
}

resource "aws_internet_gateway" "igw" {
vpc_id = aws_vpc.main_vpc.id
}

resource "aws_route_table" "public_rt" {
vpc_id = aws_vpc.main_vpc.id
}

resource "aws_route" "public_route" {
route_table_id         = aws_route_table.public_rt.id
destination_cidr_block = "0.0.0.0/0"
gateway_id             = aws_internet_gateway.igw.id
}

resource "aws_route_table_association" "public_assoc" {
subnet_id      = aws_subnet.public_subnet.id
route_table_id = aws_route_table.public_rt.id
}

resource "aws_security_group" "web_sg" {
vpc_id = aws_vpc.main_vpc.id

  ingress {
from_port   = 80
to_port     = 80
    protocol    = "tcp"
cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
from_port   = 443
to_port     = 443
    protocol    = "tcp"
cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
from_port   = 0
to_port     = 0
    protocol    = "-1"
cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "web_instance" {
ami           = "ami-0c55b159cbfafe1f0" # Amazon Linux 2 AMI
instance_type = "t2.micro"
subnet_id     = aws_subnet.public_subnet.id
security_groups = [aws_security_group.web_sg.name]

user_data = <<-EOF
#!/bin/bash
              yum install -y httpd
systemctl start httpd
systemctl enable httpd
            EOF

  tags = {
    Name = "WebServerInstance"
  }
}

```


### 2. Initialize Terraform and apply the configuration:
Run the following command to initialize Terraform. This will download the necessary provider plugins
```bash
terraform init
```
Run the following command to apply the configuration to provision the resources
```bash
terraform apply
```
# CI/CD Pipeline Setup using Jenkins
### Prerequisites:
- Jenkins server set up
- Jenkins plugins: Git, Pipeline
### Jenkins Pipeline Configuration:

### 1. Create a Jenkinsfile in your Git repository:
```bash
pipeline {
    agent any

    stages {
stage('Clone repository') {
            steps {
                git 'https://github.com/your-repository.git'
            }
        }
        stage('Build') {
            steps {
                // Your build steps
sh 'make build'
            }
        }
        stage('Test') {
            steps {
                // Your test steps
sh 'make test'
            }
        }
        stage('Deploy') {
            steps {
sshagent(['your-ssh-credentials-id']) {
sh '''
scp -o StrictHostKeyChecking=no -r * ec2-user@your-ec2-instance-public-ip:/var/www/html
                    '''
                }
            }
        }
    }
    post {
        always {
cleanWs()
        }
    }
}
```
### 2.	Configure Jenkins Job:
- Create a new pipeline job in Jenkins.
- Point the job to your repository containing the Jenkinsfile.

# Monitoring and Logging with AWS CloudWatch

### Prerequisites:
- AWS CLI configured
- IAM Role with CloudWatch permissions attached to the EC2 instance

### Setup CloudWatch Monitoring and Alerts:
### 1.	Create a CloudWatch Alarm for CPU usage:

awscloudwatch put-metric-alarm --alarm-name "HighCPUUsage" --metric-name CPUUtilization --namespace 

AWS/EC2 --statistic Average --period 300 --threshold 80 --comparison-operator GreaterThanOrEqualToThreshold --dimensions "Name=InstanceId,Value=i-xxxxxxxxxx" --evaluation-periods 1 --alarm-actions arn:aws:sns:us-west-2:123456789012:MyTopic

### 2. Subscribe to the SNS topic to receive email alerts:

awssns subscribe --topic-arn arn:aws:sns:us-west-2:123456789012:MyTopic --protocol email --notification-endpoint your-email@example.com
