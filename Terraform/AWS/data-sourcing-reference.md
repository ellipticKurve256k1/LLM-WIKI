## fetch existing setup through data source

```sh

data "aws_subnets" "public" {
   filter {
      name = "tag:Name"
	  values=["public"]
   }
    filter {
	  name = "cidr-block"
	  values = ["172.31.0.0/24"]
   }
}

data "aws_security_group" "public" {
   filter {
      name = "group-name"
      values = "public"
   }
}

data "aws_vpc" "primary" {
	filter {
		name = "tag:Name"
		values = ["default"]
	}
}
```

## launch instance

```sh
resource "aws_instance" "server" {
   count = 0
   ami = "ami-0126975fb247bf2e7"
   instance_type = "t3.micro"
   key_name = "keyname"
   vpc_security_group_ids = [data.aws_security_group.default.id]
   subnet_id = data.aws_subnets.public.ids[0]
   user_data = file("${path.module}/setup.sh")
   associate_public_ip_address = true
}
```

- `${path.module}`: path to current terraform module located