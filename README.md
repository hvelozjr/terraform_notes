## 2021 Terraform Notes: Linux Academy


## Terraform Init (Initializing the Working Directory)



* Initializes the working directory that contains your Terraform code
	* `terraform  init`

* Write the Code

* Plan the Code
	* `terraform plan`

* Apply the Code: Deploys the instructions and statement in the code. Updates the deployment and state tracking mechanism file, the `state file.`
	* `terraform apply`

* Looks at the recorded and `destroy` all resources tracked by the `state file.`
	* `terraform destory`


## Resource Addressing in Terraform: Understanding Terraform Code

Providers

```
provider "aws" {
	region = us-east-1"
}
```


###### Resources

1. `Resource`: Reserved Keyword
2. `aws_instance`: Resource provided by Terraform provider
3. `web`: User-provided arbitrary resource name
4. {Resources config argument}
5. To access the resources: `aws_instance.web`

```
resource "aws_instance" "web" {
	ami				= "ami-a1b2c3d4"
	instance_type	= "t2.micro"
}


```

###### Data

1. `Data`: Reserved Keyword
2. `aws_instance`: Resource provided by Terraform provider
3. `my-vm`: User-provided arbitary resource name
4. {Data source argument}
5. To access data: `data.aws_instance.my-vm`

```
data "aws_instance" "my-vm" {
	instance_id "i-1234567890"
}
```



## Download and Install Terraform

* [Download Terraform](https://www.terraform.io/downloads.html)


## Installing Terraform and Terraform Providers

1. vim `providers.tf` Described below.
2. export TF_log: `debug command.`
3. terraform init
4. terraform plan


```
/******************************************************************************
 * Provider Config
 *****************************************************************************/
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 2.65"
    }
    local = {
      source  = "hashicorp/local"
      version = "~> 1.4.0"
    }
  }

  required_version = ">= 0.13"
}
```

## Terraform State: The Concept

* Resources Tracking! Terraform state: `terraform.tfstate`
* They are stored in flat files, by default name `terraform.tfstate`
* Never lose your Terraform State file!

## Terraform Variables and Outputs

* Terraform Variables
* File: `terraform.tfvars`

```terrform
variable "my-var" {}
```
###### or

1. `variable`: Reserved Keyword
2. `my-var`: User provided variable name
3. {Variable config arguments such as type of variable and default value}
4. To access data: `var.my-var`

```
variable "my-var" {
	description = "My Test Variables"
	Type		= string
	default		= "Hello"
}
```

###### Terraform Validation Feature (Optional)
```
variable "my-var" {
	description = "My Test Variables"
	Type		= string
	default		= "Hello"
	validation {
		condition 		= length(var.my-var) > 4
		  error_message = "The string must be more than 4 characters"
	}
}
```

###### Using Sensitive Parameters

* By using Sensitive Parameters you can block variables form showing up when terraform is being executed.

```
variable "my-var" {
	description = "My Test Variables"
	Type		= string
	default		= "Hello"
	sensitive = true <<- Boolean
}
```

###### Variables Contraints

1. `Base Types`
	* string
	* number
	* bool
2. `Complex Types`
	* list, set, map, object, tuple

###### Simple String Type
```
variables "image_id" {
	type    = "string"
	default = "Hello"
}
```

###### Simple List Type

* The [] in the variable indicates its a list, and thus you can add additional configuration arguments.

```
variables "availability_zone_names" {
	type    = list(string)
	default = ["us-west-1a"]
}
```

###### Complex List of Objects

* The first half of the variable defines the block.
* The second part of the varibale sets the defaults.
* There is an order in that variables are executed. You may need to look that up.

```
variables "docker_ports" {
	type    = list(object)({
		internal = number
		external = number
		protocol = string
	}))
	default = [
		{
			internal = 8300
			external = 8300
			protocol = "tcp"
		}
	]
}
```


###### Output Values

* `output`: Reserved Keyword Value
* `instance_type`: User provided variable name
* `value`: `aws_instance.my-vm.private_ip`
* {Varibales config and arguments}
* Value is `manditory` and needs to be there.
* You can assign any value can also use other variables in the value descriptor.
* Output variables values are shown in the shell after running `teraform apply`.

```
output "instance_ip" {
	description    = "VM's Private IP"
	value = aws_instance.my-vm.private_ip
}
```

## Terraform Provisioners: When to Use Them

* When to use Provisioners
* The difference between a create and destroy provisioner is that a destory has a `when.`
* These provisioners appear to be run by
	* terraform apply
	* terraform destroy

```
resource "null_resource" "dummy_resource" {
			Provisioner "local-exec" {
						Command = "echo '0' > status.txt"
			}
			provisoner "local-exec" {
						when 		= destroy
						command = "echo '1' > status.txt"
			}
}
```


###### Example
```
resource "aws_instance" "ec2-virtual-machine" {
	ami 													= "ami"
	instance_type									= t2.micro
	key_name											= KP
	associate_public_ip_address 	= true
	vpc_security_group_ids				= [aws_sg.id]
	subnet_id											= aws_subnet.subnet.id
	provisioner "local-exec" {
		command = "aws ec2 wait instance-status-ok --region us-east-1 --instance-ids ${self.id}"
	}

}
Note: self.id will grab the data from it's actual resource or self.
```

## Terraform State Command

* `terraform state list`: List out all resources tracked by the Terraform State file
* `terraform state rm`: Delete a resource from the Terraform State file
* `terraform state show`: Show details of a resource tracked in the Terraform State file

## Persisting Terraform State in AWS S3

1. `backend`: s3
2. `region`: "us-east-1"
3. `key`: The name of the state file we want to assign.
4. `bucket`: The name of the s3 bucket. Needs to pre-exist.
5. Create s3 bucket via aws cli: `aws create-bucket --bucket myawesomes3bucket3344`, you will need to have setup terminal credentials before hand.
6. Copy statefile locally
	* `aws s3 cp s3://myawesomes3bucket3344/terraform.tfstate .`

```
terraform {
	backend "s3" {
		region = "us-east-1"
		key    = "terraform.tfstate"
		bucket = "myawesomes3bucket3344"
	}
}
```

## Accessing and Using Terraform Modules

* `module`: Reserved Keyword
* `source`: Module source (mandatory).
* `version` Module version.
* `region` Input Parameter(s) for module
* Other allowed Parameters:
	* count
	* for_each
	* providers
	* depends_on (dependencies)

```
module "my-vpc-module" {
	source = "./module/vpc"
	version = "0.0.5"
	region = var.region
}
```

###### Use Ouputs in main

* Accessing module outputs in your
* Module ouput 	
	* `module.my-vpc-module.subnet_id`

```
resource "aws_instance" "my-vpc-module" {
	......# other arguments
	subnet_id = module.my-vpc-module.subnet_id
}
```

###### Invoking module output in main

* Inside your module
* Invoke: `module.my-vpc-module.ip_address`

```
output "ip_address" {
	value = aws_instance.private_ip
}
```

## Terraform Built-in Functions

* You can only use the function provided by terraform
* terraform console (Interactive console for Terraform)
* [Terraform Functions](https://www.terraform.io/docs/language/functions/index.html)


```
variable "project-name" {
	type = string
	default = "prod"
}

resource "aws_vpc" "my-vps"
	cidr_block = "10.0.0.0/16"
	tags = {
		Name = join("-",["terraform", var.project-name])
	}

```

## Terraform Type Constraints (Collections & Structural)

###### Collections example:

* Collection type allow multiple values of one primitive type to be grouped together
* Constructors for these Collections include:
		* list(type)
		* map(type)
		* set(type)

```
variable "trainning" {
	Type			= list(string)
	defaults	= ["ACG", "LA"]
}
```

###### Structural example:

* Structural types allow multiple values of different primitive types to be grouped together.
* Constructors for these Collections include
		* Object(type)
		* tuple(type)
		* set(type)

```
variable "instructor" {
	type = object({ # Object type Contains several var within it
		name = string # mixed primitive types
		age = number  # mixed primitive types 		
		})
}
```

###### any example:

* any is a placeholder for primitive type yet to be decided.
* Actual type will be determined at runtime
* Terraform will reconize all values as numbers in one variable.

```
variable "data" {
	type = list(any)
	default = [1, 42, 7]
}
```

## Terraform Dynamic Blocks

* The config block you're trying to replicate
* Complex variable to iterate over
* The nested "content" block defines the body of each generated block, using the variable you provide.

```
resource "aws_security_group" "my-sg" {
	name = 'my-aws-security-group'
	vpc-id = aws_vpc.my-vpc.id
	dynamic "ingress" { # The config block you're trying to replicate
		for_each = var.rules # Complex variable to iterate over
		content {
			from_port		= ingress.value["port"] # The nested "content"
			to_port		= ingress.value["port"] # The nested "content"
			protocol		= ingress.value["proto"] # The nested "content"
			cidr_blocks		= ingress.value["cidr"] # The nested "content"
		}
	}
}
```
###### complex variable passed to the block above
```
variable "rules" {
	default = [
		{  
			 port   			= 80
		   proto  			= "tcp"
		   cidr_block 	= ["0.0.0.0/0"]
		 },
		 {
			 port 				= 22
			 proto  			= "tcp"
			 cidr_blocks 	= ["1.2.3.4/32"]
		 }
	]
}
```

## Terraform fmt, taint, and import Commands

###### fmt

* Formats Terraform code for readability
* Helps in keeping code consistent
* Safe to run any run

###### fmt Command Syntax

 * `terraform fmt`

###### Terraform fmt Scenario:

* Before pushing your code to version control.
* After upgrading Terraform or its Modules.
* Anytime you have made changes to code.

###### taint

* Taints a resource, forcing it to be destroyed and recreated.
* Modifies the state file, which causes the recreation workflow.
* Tainting a resource may cause other resources to be modified.

###### taint Command Syntax

* `terraform taint 	RESOURCE_ADDRESS`

###### Terraform taint Scenario:

* To cause provisioners to run.
* Replace misbehaving resources forcefully.
* To mimic side effects of recreation not modeled by any attributes of the resource.

###### import

* Maps existing resource to Terraform using an "ID"
* "ID" is dependent on the underlying vendor, for example to import an AWS EC2 intance you'll need to provide its instance ID.
* Importing the same resource to multiple Terraform resources can cause unknown behavior and is not recommended.

###### import Command Syntax

* `terraform import Resource_Address ID`

###### Terraform import Scenario:

* When you need to work with an existing resources.
* Not allowed to create new resources.
* When you're not in control of creation process of infrastructure.

###### Terraform Block

* The terrafrom provide is greater than 0.13.0

```
terraform {
	required_version = ">=0.13.0"
	required_providers {
		aws = ">=3.0.0"
	}
}
```

## Terraform Workspaces

* The Terraform Workspaces are alternate state files within the same working directory.
* Terraform starts with a single workspace that is always called default. It cannot be deleted.

```
terraform workspace new <WORKSPACE-NAME> Creates new WS

terraform workspace select <WORKSPACE-NAME> Select WS

```

###### terraform workspace variable

* ``${terraform.workspace}``

###### example one:
```
resource "aws_instance" "example" {
	count = terraform.workspace == "default" ? 5 : 1
	#...other arguments
}
```
###### example two:
```
resource "aws_s3_bucket" "bucket" {
	bucket = "myxyzbucket-${terraform.workspace}"
	acl 	 = "private"
}
```
###### example three:
```
region = terraform == "default" ? "us-east-2" : "us-west-1"

Name = "${terraform.workspace}-ec2"

```

## Debugging Terraform

* TF_LOG is an environment variable for enabling verbose logging in Terraform. By Default, it will send logs to stderr (standard error output)
* Can be sent to the following level: TRACE, DEBUG, INFO, ERROR
* TRACE is the most verbose level of logging and the most reliable one.
* To persist logged output, use the TF_LOG_PATH environment variable.
* Setting logging environment variable for Terraform on Linux:

```
export TF_LOG=TRACE

export TF_LOG_PATH-./terraform.log
```

## Other resources

1. https://app.linuxacademy.com/hands-on-labs/1c87bd80-5dd4-4b91-aec2-91f70c854cea?redirect_uri=https:%2F%2Fapp.linuxacademy.com%2Fsearch%3Fquery%3Dterraorm

2. https://app.linuxacademy.com/hands-on-labs/24cb7511-50f6-4cfb-8c24-396d2634282e?redirect_uri=https:%2F%2Fapp.linuxacademy.com%2Fsearch%3Fquery%3Dterraorm

3. https://app.linuxacademy.com/hands-on-labs/77824b8d-9b1a-4f4c-9dc9-f94d35668556?redirect_uri=https:%2F%2Fapp.linuxacademy.com%2Fsearch%3Fquery%3Dterraorm

4. https://learn.hashicorp.com/tutorials/terraform/state-import

5. https://learn.hashicorp.com/tutorials/terraform/associate-review?in=terraform/certification
