- basic commands

```sh
aws configure (list)
aws ec2 describe-instances
aws ec2 describe-vpcs
aws ec2 describe-subnets
	--subnet-ids $subnet_id
aws ec2 describe-route-tables
aws ec2 describe-security-groups
aws ec2 describe-internet-gateways
aws ec2 describe-nat-gateways
aws ec2 describe-network-interfaces
aws ec2 describe-addresses
aws ec2 describe-key-pairs
aws ec2 describe-images
aws sts get-caller-identity # view current identity
```

- common additional options
	- `--output`
		- `table`
	- `--query`