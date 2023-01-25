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
## 4 Create ECS-SecurityGroup
```t
# AWS EC2 Security Group Terraform module
# Security Group for ECS Application

module "lb_ecs_application_sg" {
  source  = "terraform-aws-modules/security-group/aws"
  version = "4.16.2"

  name        = "ecs-web-sg"
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
```t
resource "aws_ecs_task_definition" "webapp" {
  family                   = "webserver"
  network_mode             = "awsvpc"
  requires_compatibilities = ["FARGATE", "EC2"]
  cpu                      = 1024
  memory                   = 2048

  container_definitions = <<DEFINITION
  [
    {
        "name": "webserver",
        "image": "nginx:alpine",
        "cpu": 1024,
        "memory": 2048,
        "portMappings" : [
            {
                "containerPort": 80,
                "hostPort": 80
            }
        ]
    }
  ]
  DEFINITION
```
## 6 Create ECS Cluster
```t
 resource "aws_ecs_cluster" "webapp_ecs_cluster" {
    name = "webapp-ecs-cluster"
}

resource "aws_ecs_service" "webapp_service" {
  name = "forest"
  cluster = aws_ecs_cluster.webapp_ecs_cluster.id
  task_definition = aws_ecs_task_definition.webapp.arn
  desired_count = var.app_count
  launch_type = "FARGATE"

  network_configuration {
    assign_public_ip = true
    security_groups = [ module.lb_ecs_application_sg.security_group_id ]
    subnets = data.aws_subnet_ids.public_subnets.ids
  }
}

```
## 7 Deploy infraestructure
```t
terraform apply -auto-approve
```
## 8 Go to AWS Console -> ECS -> Task definitions -> Task ID and copy Public IP Address
## 9 Using a web browser, paste the public IP address and verify that the default NGinx page appears
## 10 Clean Resources
```t
terraform destroy -auto-approve
```

