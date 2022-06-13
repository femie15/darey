# AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM. PART 2

### Create 4 private subnets

Things to note;

1. Use variables or length() function to determine the number of AZs
2. Use variables and cidrsubnet() function to allocate vpc_cidr for subnets
3. Keep variables and resources in separate files for better code structure and readability
4. Tags all the resources you have created so far. (Explore how to use format() and count functions to automatically tag subnets with its respective number)

### Reasons to use tags

- Resources are much better organized in ‘virtual’ groups
- They can be easily filtered and searched from console or programmatically
- Billing team can easily generate reports and determine how much each part of infrastructure costs how much (by department, by type, by environment, etc.)
- You can easily determine resources that are not being used and take actions accordingly
- If there are different teams in the organisation using the same account, tagging can help differentiate who owns which resources.

```
# Create private subnet
resource "aws_subnet" "private" {
  count                   = var.preferred_number_of_private_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_private_subnets
  vpc_id                  = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 8, count.index + 2)
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]

  tags = merge(
    var.tags,
    {
      Name = format("%s-PrivateSubnet-%s!", var.name, count.index)
    },
  )
}
```

we can do the same refactoring for public subnet. in the variable.tf file we add the following

```
variable "name" {
  type =string
  default = "BR"
}

variable "tags" {
  description = "Tags mapping for resource naming"
  type =map(string)
  default = {}
}
```

in the terraform .tfvars file, we have this

```
tags = {
  Enviroment      = "development" 
  Owner-Email     = "myemail@domain.com"
  Managed-By      = "Terraform"
  Billing-Account = "1234567890"
}
```

### Internet Gateways

in a new file called "internet_gateway.tf"

```
resource "aws_internet_gateway" "ig" {
  vpc_id = aws_vpc.main.id

  tags = merge(
    var.tags,
    {
      Name = format("%s-%s!", aws_vpc.main.id,"IG")
    } 
  )
}
```

The format() function helps to dynamically generate a unique name for this resource? The first part of the %s takes the interpolated value of aws_vpc.main.id while the second %s appends a literal string IG and finally an exclamation mark is added in the end

### NAT Gateways

We need to create an Elastic IP first for the NAT Gateway, we will require the use of "depends_on" to indicate that the Internet Gateway resource must be available before the Nat gateway is created. 
Although Terraform does a good job to manage dependencies, but in some cases, it is good to be explicit.

Elastic IP and Nat Gateway

```
resource "aws_eip" "nat_eip" {
  vpc        = true
  depends_on = [aws_internet_gateway.ig]

  tags = merge(
    var.tags,
    {
      Name = format("%s-EIP", var.name)
    },
  )
}

resource "aws_nat_gateway" "nat" {
  allocation_id = aws_eip.nat_eip.id
  subnet_id     = element(aws_subnet.public.*.id, 0)
  depends_on    = [aws_internet_gateway.ig]

  tags = merge(
    var.tags,
    {
      Name = format("%s-Nat", var.name)
    },
  )
}
```

in the variable.tf file, we need to specify for var.environment

```
variable "environment" {
  description = "Infrastructure environment"
  type =string
  default = "BR"
}
```

and in terraform.tfvars... `enviroment = "dev" `

### AWS ROUTES

Create a file called "route_tables.tf" for the route table resources.

```
# create private route table
resource "aws_route_table" "private-rtb" {
  vpc_id = aws_vpc.main.id

  tags = merge(
    var.tags,
    {
      Name = format("%s-Private-Route-Table-%s!", var.name, var.environment)
    },
  )
}

# associate all private subnets to the private route table
resource "aws_route_table_association" "private-subnets-assoc" {
  count          = length(aws_subnet.private[*].id)
  subnet_id      = element(aws_subnet.private[*].id, count.index)
  route_table_id = aws_route_table.private-rtb.id
}

# create route table for the public subnets
resource "aws_route_table" "public-rtb" {
  vpc_id = aws_vpc.main.id

  tags = merge(
    var.tags,
    {
      Name = format("%s-Public-Route-Table-$s!", var.name , var.environment)
    },
  )
}

# create route for the public route table and attach the internet gateway
resource "aws_route" "public-rtb-route" {
  route_table_id         = aws_route_table.public-rtb.id
  destination_cidr_block = "0.0.0.0/0"
  gateway_id             = aws_internet_gateway.ig.id
}

# associate all public subnets to the public route table
resource "aws_route_table_association" "public-subnets-assoc" {
  count          = length(aws_subnet.public[*].id)
  subnet_id      = element(aws_subnet.public[*].id, count.index)
  route_table_id = aws_route_table.public-rtb.id
}
```
























