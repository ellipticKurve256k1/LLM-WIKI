## aws_instance

```sh
resource "aws_instance" "instance_name" {
	count = 0
	ami = "ami"
	instance_type = "t3.micro"
	key_name = "keyname"
	subnet_id = "subnet_id"
	vpc_security_group_ids = []
	associate_public_ip_address = true

	user_data = file("${path.module}/init.sh")
	
	root_block_device { # default os level volume
		volume_size = 20
		volume_type = "gp3"
		delete_on_termination = true
		encrypted = true
		
		tags = {
			Name = "default-block" # ebs is separated resource
		}
	}
	
	tags = {
		Name = "tag-name"
		Publicity = "Public"
	}
}
```

### addtional volume

```sh
resource "aws_instance" "instance_name" {
	ebs_block_device {
		device_name = "/dev/sdf"
		volume_size = 50
		volume_type = "gp3"
		delete_on_termination = true
		encrypted = true
		
		tags = {
			Name = "ebs_name"
		}
	}
}
```

## aws_ebs_volume

```sh
resource "aws_ebs_volume" "volume_name" {
	availability_zone = aws_instance.instance_name.availability_zone
	size = 30
	type = "gp3"
	encrypted = true
	
	tags = {
		Name = "volume_name"
	}
}
```

### aws_volume_attachment

```sh
resource "aws_volume_attachment" "name" {
	device_name = "/dev/sdf"
	volume_id = aws_ebs_volume.volume_name.id
	instance_id = aws_instance.instance_name.id
}
```

## aws_subnet

```sh
resource "aws_subnet" "subnet_name" {
	vpc_id = "vpc_id"	
	cidr_block = "172.31.0.0/24"
	
	tags = {
		Name = "public_subnet"
	}
}
```

## Security Group (SG)

1. input ingress / egress (inline rule)

```sh
resource "aws_security_group" "security_group_name" {
	name = "${project_name}_web_sg"
	description = "description"
	vpc_id = 
	
	ingress {
		description = 
		from_port = 22
		to_port = 22
		protocol = "tcp"
		cidr_blocks = ["172.31.0.0/24"] # source ip
	}
	
	ingress {
		description = "allow http"
		from_port = 80
		to_port = 80
		protocol = "tcp"
		cidr_blocks = ["0.0.0.0/0"]
	}
	
	egress {
		description = "Allow all outbound"
		from_port = 0
		to_port = 0
		protocol = "-1"
		cidr_blocks = ["0.0.0.0/0"] # destination
	}
	
	tags = {
		Name = "Name"
	}
}
```

2. declare SG first, ingress / egress in separate part **(recommended)**
```sh
resource "aws_security_group" "security_group_name" {
	name = "name"
	description = "description"
	vpc_id = aws_vpc.main.id
	
	tags = {
		Name = "Name"
	}
}
```

```sh
resource "aws_vpc_security_group_ingress_rule" "security_group_name" {
	security_group_id = aws_security_group.security_group_name.id
	description = "description"
	ip_protocol = "tcp" #-1: all protocols
	from_port = 80
	to_port = 80
	cidr_ipv4 = "0.0.0.0/0"
}
```

```sh
resource "aws_vpc_security_group_egress_rule" "name" {
	security_group_id = aws.security_group.security_group_name.id
	ip_protocol = "-1"
	cidr_ipv4 = "0.0.0.0/0"
	
	description = "description"
}
```

## Key Pair
1. make the ssh key first in locally, refer [[ssh-key-management-commands]]
2. declare through `resource` block

```sh
resource "aws_key_pair" "key_name" {
	key_name = "my_key_name"
	public_key = file("~/.ssh/key.pub")
}
```