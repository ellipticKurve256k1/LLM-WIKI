## Configuration Block

```hcl
terraform {
   required_providers {
      aws = {
	     source = "hashicorp/aws"
		 version = ""
      }
   }
}

provider "aws" {
   region = "ap-northeast-1"
}
```

## output

```sh
output "output_name" {
  value = "value" 
}
```

```sh
output "ip_addr" {
	value = aws_instance.instnace_name.public_ip
}
```

## variable

```sh
variable "region" {
  type    = string
  default = "ap-northeast-1"
}
```

```sh
variable "cidrs" {
	type = map(string)
	default = {
		home = "10.10.10.10/32"
		vpn = "10.20.10.10/32"
	}
}
```

### variable usage

```sh
var.region
```

## locals

```sh
locals {
	test = "Test"
}
```

### local usage

```sh
local.test
```