# AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM. PART 3 – REFACTORING

- Firstly, we re-arrange our files and directories from project-17 into various module sub-directory (VPC, ALB, Autoscaling, compute, EFS, RDS, Security).

![apache](https://github.com/femie15/darey/blob/main/project%201/project18/1-directories.PNG)


![apache](https://github.com/femie15/darey/blob/main/project%201/project18/2-directories%20files.PNG)

- Create a "main.tf" in the root folder, "variable.tf" in all modules. 
- Change all hard-coded script to variables in the modules.
- Adjust the root module and use a forloop to create the cirdrsubnet variation 
 
 `[for i in range(1, 8, 2) : cidrsubnet(var.vpc_cidr, 8, i)]`

- Split the lunch template and the autoscaling group in the "asg-bastion-nginx.tf" and "asg-webservers.tf" files to now include "lt-bastion-nginx.tf" and "lt-webservers.tf" 
- Create "output.tf" files in "ABL , security and VPC" to outpute data for other modules to consume.
- in the compute module, create an instance (with code) for Jenkins, Sonarqube and artifactory.

```
#  Database subnet group from the private subnets
resource "aws_db_subnet_group" "BROWT-rds" {
  name       = "browt-rds"
  subnet_ids = var.private_subnets

  tags = merge(
    var.tags,
    {
      Name = "BROWT-database"
    },
  )
}

# RDS instance with subnets group
resource "aws_db_instance" "BROWT-rds" {
  allocated_storage      = 20
  storage_type           = "gp2"
  engine                 = "mysql"
  engine_version         = "5.7"
  instance_class         = "db.t2.micro"
  name                   = "femiedb"
  username               = var.db-username
  password               = var.db-password
  parameter_group_name   = "default.mysql5.7"
  db_subnet_group_name   = aws_db_subnet_group.BROWT-rds.name
  skip_final_snapshot    = true
  vpc_security_group_ids = var.db-sg
  multi_az               = "true"
}
```

- Rename the security group file "security.tf" to "main.tf" and make the seccurity group dynamic and also create an egress rule.
- create security group rule "sg-rule.tf" and 

# Backend on S3

This is where and how operations are performed, where state snapshots are stored...

S3 backend is State Locking; used to lock your state for all operations that could write state to avoid unwanted access. DynamoDB is required for this purpose.

create "backend.tf" and add the following 

```
# Note: The bucket name may not work for you since buckets are unique globally in AWS, so you must give it a unique name.
resource "aws_s3_bucket" "terraform_state" {
  bucket = "femie-dev-terraform-bucket"
  # Enable versioning so we can see the full revision history of our state files
  versioning {
    enabled = true
  }
  # Enable server-side encryption by default
  server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
        sse_algorithm = "AES256"
      }
    }
  }
}
```  


![apache](https://github.com/femie15/darey/blob/main/project%201/project18/3-bucket.PNG)

You must be aware that Terraform stores secret data inside the state files. Passwords, and secret keys processed by resources are always stored in there. Hence, you must consider to always enable encryption. You can see how we achieved that with server_side_encryption_configuration.

create a DynamoDB table to handle locks and perform consistency checks. In previous projects, locks were handled with a local file as shown in terraform.tfstate.lock.info. Since we now have a team mindset, causing us to configure S3 as our backend to store state file, we will do the same to handle locking. Therefore, with a cloud storage database like DynamoDB, anyone running Terraform against the same infrastructure can use a central location to control a situation where Terraform is running at the same time from multiple different people.

Dynamo DB resource for locking and consistency checking:

```
resource "aws_dynamodb_table" "terraform_locks" {
  name         = "terraform-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"
  attribute {
    name = "LockID"
    type = "S"
  }
}
```

run `terraform apply` to provision resources.

Configure S3 Backend
terraform {
  backend "s3" {
    bucket         = "femie-dev-terraform-bucket"
    key            = "global/s3/terraform.tfstate"
    region         = "eu-central-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}


Create a new file and name it "output.tf" and add below code.

```
output "s3_bucket_arn" {
  value       = aws_s3_bucket.terraform_state.arn
  description = "The ARN of the S3 bucket"
}
output "dynamodb_table_name" {
  value       = aws_dynamodb_table.terraform_locks.name
  description = "The name of the DynamoDB table"
}
```

run `terraform apply` Terraform will automatically read the latest state from the S3 bucket to determine the current state of the infrastructure. Even if another engineer has applied changes, the state file will always be up to date.



















































