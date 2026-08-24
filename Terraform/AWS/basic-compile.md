1. `terraform validate`
	- to check the grammar errors
2. `terraform plan`
	- to display what would be the results be like 
	- `terraform plan -var-file="var_file.tfvars"`
	- `terraform plan -var="variable_name=value"`
3. `terraform apply`
	`-auto-approve` option can be used without typinng "yes"
	`-var="env=prod" -var="instance_type=t3.small`
	`-var-file="var_file.tfvars"`
4. `terraform init`
5. `terraform fmt`
6. `terraform state list`
7. `terraform state rm ${}`
	- delete from the terraform state
8. `terraform state show ${}`
	- display details