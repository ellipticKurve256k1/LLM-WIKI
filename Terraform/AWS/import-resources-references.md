## using `import command`

1. declare `resource` in the tf file.

```sh
resource "aws_subnet" "subnet_name" {
	# can be empty now, but needs to be filled later
}
```

2. import command

```sh
terraform import aws_subnet.subnet_name id
```

3. compile the source 

```sh
terraform plan
terraform apply
```

## Using `import {}`

1. declare `resource` in the tf file.

```sh
resource "aws_subnet" "subnet_name" {
	# can be empty now, but needs to be filled later
}
```

2. declare in the file

```sh
import {
	to = aws_subnet.subnet_name
	id = "id" 
}
```
- when importing key, input **"key-name"** intead of "pairid"
	