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
data "aws_availability_zones" "available_zones" {
  state = "available"
}


## 4 Create LB-SecurityGroup
## 5 Create LB-Resource
## 6 Create LB-Outputs
## 7 Create ECS Task Definition
## 8 Creates ECS Cluster
## 9 Terraform apply command 
## 10 Validate infrastructure
