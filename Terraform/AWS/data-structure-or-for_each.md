## map

```sh
locals {
	cidrs = {
		home = "1.1.1.1/32"
		vpn = "2.2.2.2/32"
	}
}
```

## for_each

Iterates over a list or map: as it runs, each `data` or `resource` instance is treated independently, with its values computed separately for each element.

```sh
resource "aws_vpc_security_group_ingress_rule" "ingress_rule_test" {
	for_each = local.cidrs 
	ip_protocol = "tcp"
	from_port = 22
	to_port = 22
	cidr_ipv4 = each.value
}
```