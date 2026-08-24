## `variables.tf`

- just a convention to store all `variable` block 

for instance:
```sh
# default field is not mandatory
variable "region" {
	type = string
	default = "ap-northeast-1"
}

variable "ingress" {
	type = map(string)
	default = {
		home = "10.10.10.10/32"
		vpn = "10.20.10.10/32"
	}
}
```

- must have `filename.tfvars` that actually defines the real value of each variable defined in `variables.tf`.
	- refer [[basic-compile]] for usage of variable for manual variable input in runtime.
- `filename.tfvars`
```sh
region = "ap-northeast-1"
```

## `terraform.tfvars`

- autoloads the values of variables 
- or `file_name.auto.tfvars` also auto import the variables