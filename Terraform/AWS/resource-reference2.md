## Internet gateway

```sh
resource "aws_internet_gateway" "primary_igw" {
	vpc_id = vpcid
	
	tags = {
		Name = "primary_igw"
	}
}
```

## Route Table

1. declare a route table
```sh
resource "aws_route_table" "main_rt" {
	vpc_id = aws_vpc.vpc_name.id
	
	tags = {
		Name = "primary_rt"
	}
}
```

2. declare a route for destination
```sh
resource "aws_route" "public_internet" {
	route_table_id = aws_route_table.main_rt.id
	destination_cidr_block = "0.0.0.0/0"
	gateway_id = aws_internet_gateway.primary_igw.id
}
```

### IMPORT
- when importing `aws_route`, since there is no `route_id` in AWS, the combination of route_table_id and destination should be used separated by `_`

```sh #example
import {
	to = aws_route.public_internet
	id = "rtb-0123456789abcdef0_0.0.0.0/0"
}
```

3. Associate to subnet
```sh
resource "aws_route_table_association" "public_assoc" {
	subnet_id = aws_subnet.public_subnet_name.id
	route_table_id = aws_route_table.mainr_rt.id
}
```

### IMPORT
- when importing route association, use `/` to establish connection between subnet_id and route_table_id
```sh
import {
	to = aws_route_table_association.public_assoc
	id = 'subnet-id/rtb-id'
}
```
