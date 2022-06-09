# AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM PART 1

We need to re-create the project 15 with Infrastructure as code.

![apache](https://github.com/femie15/darey/blob/main/project%201/project16/archi.png)

1. Create an IAM user named "terraform" and give it only "programatic access" to AWS account and grant "AdministratorAccess permissions" to it.
2. Configure programmatic access from the workstation to connect to AWS using "secret access key and access key ID" of the new user, and a <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/guide/quickstart.html">Python SDK (boto3)</a>. (Python 3.6 or higher is recommended on your workstation), Use "AWS CLI with AWS configure" for easier authentication.
3. Create a S3 bucket and name it "femie13-dev-terraform-bucket"

![apache](https://github.com/femie15/darey/blob/main/project%201/project16/1-s3.PNG)

To confirm the boto3 set up, on the terminal, run `python` , this will cause the terminal to expect python code, then enter the script below to fetch the list of S3 buckets.

```
import boto3
s3 = boto3.resource('s3')
for bucket in s3.buckets.all():
    print(bucket.name)
```

![apache](https://github.com/femie15/darey/blob/main/project%201/project16/2.PNG)

## VPC | SUBNETS | SECURITY GROUPS

Create a folder called "PBL" and in it, create a file called "main.tf", in it paste the snippet below and set provider to "aws" and region to the specific region closer to our users (this should be noted in other configurations).

```
provider "aws" {
  region = "eu-east-1"
}

# Create VPC
resource "aws_vpc" "main" {
  cidr_block                     = "172.16.0.0/16"
  enable_dns_support             = "true"
  enable_dns_hostnames           = "true"
  enable_classiclink             = "false"
  enable_classiclink_dns_support = "false"
}
```

### Plugins

To download necessary plugins used by providers(in our case) and provisioners for Terraform to work, we need to run the command `terraform init` this downloads some files (".terraform.lock.hcl") and folders (".terraform" where is stores plugins).

![apache](https://github.com/femie15/darey/blob/main/project%201/project16/3-terainit.PNG)

From our "main.tf" file we defined a VPC resource to be created, let's view the resource terraform plan to create by running `terraform plan`

![apache](https://github.com/femie15/darey/blob/main/project%201/project16/4-plan.PNG)

During the "terraform plan" and "terraform apply" a temporary file called "terraform.tfstate.lock.info" This is what Terraform uses to track, who is running its code against the infrastructure at any point in time. This is very important for teams working on the same Terraform repository at the same time. The lock prevents a user from executing Terraform configuration against the same infrastructure when another user is doing the same – it allows to avoid duplicates and conflicts.

"terraform.tfstate" is also created. This is how Terraform keeps itself up to date with the exact state of the infrastructure. It reads this file to know what already exists, what should be added, or destroyed based on the entire terraform code that is being developed.

![apache](https://github.com/femie15/darey/blob/main/project%201/project16/5-temp.PNG)

### Subnets resource section

Create the first 2 (of 6) public subnets. in the two AZs

```
# Create public subnets1
    resource "aws_subnet" "public1" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.0.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "us-east-1a"

}

# Create public subnet2
    resource "aws_subnet" "public2" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.1.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "us-east-1b"
}
```

Note: some values are still hard-coded and multiple resource blocks are present (these will be fixed later).

run `terraform fmt` to format the arrangement of our code well and run `terraform validate` to validate, (these are not assurance that the code will apply/plan successfully).

![apache](https://github.com/femie15/darey/blob/main/project%201/project16/6-createVpc.PNG)

now run `terraform plan` to view what we are creating and `terraform apply` to create the resources.

### Fixing The Problems By Code Refactoring

Variables: We can make our work neater and easy to read and debug by setting variables and calling them where we had hardcoded.

```
variable "region" {
        default = "eu-central-1"
    }

    variable "vpc_cidr" {
        default = "172.16.0.0/16"
    }

    variable "enable_dns_support" {
        default = "true"
    }

    variable "enable_dns_hostnames" {
        default ="true" 
    }

    variable "enable_classiclink" {
        default = "false"
    }

    variable "enable_classiclink_dns_support" {
        default = "false"
    }

    provider "aws" {
    region = var.region
    }

    # Create VPC
    resource "aws_vpc" "main" {
    cidr_block                     = var.vpc_cidr
    enable_dns_support             = var.enable_dns_support 
    enable_dns_hostnames           = var.enable_dns_support
    enable_classiclink             = var.enable_classiclink
    enable_classiclink_dns_support = var.enable_classiclink

    }
```
    
### Fixing multiple resource blocks

We now introduce "loops" and "Data Sources" to fix specifying multiple resource blocks.

fetch Availability zones from AWS and specify it with terraform.

```
# Get list of availability zones
        data "aws_availability_zones" "available" {
        state = "available"
        }
```

### making cidr_block dynamic using function cidrsubnet()

We will need to introduce a count argument in the subnet block which tells us how many times we want an iteration loop to occur (2 in this case).Therefore, Terraform will invoke a loop to create 2 subnets.

The data resource will return a list object that contains a list of AZs. Internally, Terraform will receive the data like this ["eu-central-1a", "eu-central-1b"] indexed [0,1,2,3, etc)

```
# Create public subnet1
resource "aws_subnet" "public" { 
    count                   = 2
    vpc_id                  = aws_vpc.main.id
    cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
    map_public_ip_on_launch = true
    availability_zone       = data.aws_availability_zones.available.names[count.index]

}
```

"cidrsubnet" function works like an algorithm to dynamically create a subnet CIDR per AZ. Regardless of the number of subnets created, it takes care of the cidr value per subnet. Its parameters are cidrsubnet(prefix, newbits, netnum).

- The prefix parameter must be given in CIDR notation, same as for VPC.
- The newbits parameter is the number of additional bits with which to extend the prefix. For example, if given a prefix ending with /16 and a newbits value of 4, the resulting subnet address will have length /20
- The netnum parameter is a whole number that can be represented as a binary integer with no more than newbits binary digits, which will be used to populate the additional bits added to the prefix

- run `terraform console` to see how the above works.
- run `cidrsubnet("172.16.0.0/16", 4, 0)`

![apache](https://github.com/femie15/darey/blob/main/project%201/project16/7-cidrsubnet.PNG)

See the output by changing the numbers and see what happens. (To get out of the console, type exit)

### removing hard coded count value.

Since the data resource returns all the AZs within a region, it makes sense to count the number of AZs returned and pass that number to the count argument using length() function.

Since "data.aws_availability_zones.available.names" returns a list like ["eu-central-1a", "eu-central-1b", "eu-central-1c"] we can pass it into a lenght function and get number of the AZs. e.g. terraform sees it as 'length(["eu-central-1a", "eu-central-1b", "eu-central-1c"])'. The problem here is that all availability zones will be listed and we only need 2. we then need to declare a variable.

```
variable "preferred_number_of_public_subnets" {
  default = 2
}
```

We need to replace the "count" argument with a condition; 

1. Terraform needs to check first if there is a desired number of subnets. 
2. Otherwise, use the data returned by the lenght function

```
count  = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets
```

The final code looks like this;

```
variable "region" {
  default = "us-east-1"
}

variable "vpc_cidr" {
  default = "172.16.0.0/16"
}

variable "enable_dns_support" {
  default = "true"
}

variable "enable_dns_hostnames" {
  default = "true"
}

variable "enable_classiclink" {
  default = "false"
}

variable "enable_classiclink_dns_support" {
  default = "false"
}

variable "preferred_number_of_public_subnets" {
  default = 2
}


provider "aws" {
  region = var.region
}

# Create VPC
resource "aws_vpc" "main" {
  cidr_block                     = var.vpc_cidr
  enable_dns_support             = var.enable_dns_support
  enable_dns_hostnames           = var.enable_dns_support
  enable_classiclink             = var.enable_classiclink
  enable_classiclink_dns_support = var.enable_classiclink

}

# Get list of availability zones
data "aws_availability_zones" "available" {
  state = "available"
}

# Create public subnet1
resource "aws_subnet" "public" {
  count                   = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets
  vpc_id                  = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 4, count.index)
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]

}
```

If we change the value of "preferred_number_of_public_subnets" variable to "null" 6 subnets get created.

![apache](https://github.com/femie15/darey/blob/main/project%201/project16/8-nullAllSubnet.PNG)


![apache](https://github.com/femie15/darey/blob/main/project%201/project16/9-tree.PNG)

### variables.tf & terraform.tfvars

We can now make our code neater by separating variables in a separate file and provide non default values to each of them.

1. Create a new file and name it "variables.tf"
2. Copy all the variable declarations into the new file.
3. Create another file, name it terraform.tfvars
4. Set values for each of the variables.

run `terraform plan`, `terraform apply --auto-approve`

![apache](https://github.com/femie15/darey/blob/main/project%201/project16/10-success.PNG)

Done !!!
