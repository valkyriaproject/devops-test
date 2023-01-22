# Create Service in AWS ECS
## 1 Define Provider block
```t
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~>4.0"
    }
  }
}

provider "aws" {
  region = "eu-west-1"
  access_key = "<your_aws_access_key>"
  secret_key = "<your_aws_secret_key>"
  
  default_tags {
    tags = {
      Deployed = "Solution Deployed by Terraform"
    }
  }
}  
```
## 2 Define global variables
```t
# AWS Region
variable "region" {
  type = string
  default = "eu-west-1"
  description = "Region of the VPC"
}  

variable "app_count" {
  type = number
  default = 1
}
```
## 3 Retrieve VPC data
```t
data "aws_availability_zones" "region_available_zones" {
  state = "available"
}

data "aws_vpc" "account_vpc" {
    tags = {
       Name = "App-vpc"
    }
}

data "aws_subnet_ids" "public_subnets" {
   vpc_id = data.aws_vpc.account_vpc.id
   
   tags = {
     "Tier" = "Public"
   }
}

data "aws_subnet_ids" "private_subnets" {
   vpc_id = data.aws_vpc.account_vpc.id
   
   tags = {
     "Tier" = "Private"
   }
}

output "account_vpc_id" {
  value = data.aws_vpc.account_vpc.id
}

output "public_subnets_ids" {
  value = data.aws_subnet_ids.public_subnets.ids
}

output "private_subnets_ids" {
  value = data.aws_subnet_ids.private_subnets.ids
}


```
## 4 Create LB-SecurityGroup
```t
# AWS EC2 Security Group Terraform module
# Security Group for Load Balancer Web Application

# AWS EC2 Security Group Terraform Module
# Security Group for Load Balancer Web application

module "lb_web_application_sg" {
  source  = "terraform-aws-modules/security-group/aws"
  version = "4.16.2"

  name        = "lb-web-sg"
  description = "Security Group with HTTP port open for everywhere"
  vpc_id      = data.aws_vpc.account_vpc.id

  # Ingress rules
  ingress_rules = ["http-80-tcp"]
  ingress_cidr_blocks = ["0.0.0.0/0"]

  # Egress rules
  egress_rules = ["all-all"]

  tags = {
    Name = "lb-web-application"
  }
}


```
## 5 Create ECS Task Definition
## 6 Creates ECS Cluster
## 7 Create LB-Resource
## 8 Create LB-Outputs
## 9 Terraform apply command 
## 10 Validate infrastructure
