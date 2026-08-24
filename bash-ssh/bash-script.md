## shebang

```sh
#!/usr/bin/env bash
```

## Variable

```sh
num=5
echo "$num"
```

- make sure variable is inside of double quotes

## function

```sh
my_function() {
num=$(($1 + $2)) #arguments
echo "$num"
}

my_function 20 30
```

## for

```sh
for ((i = 0; i < 5; i++)); do
echo "$i"
done
```

- loop through current dir

```sh
for i in ./*; do
   if [[ -f "$i" ]]; then
      echo "$i is regular file"
   elif [[ ! -e "$i" ]]
      echo "$i does not exist"
   else
      echo "$i is not regular file"
   fi
done
```

## if

```sh
if [[ ]]; then
elif
else
fi
```

- `[[]]`: string comparison 
	- in case numerical comparison is needed use following:
		- eq
		- ne
		- gt
		- ge
		- lt
		- le
- `(())`: numerical comparison
	- can use `>` `==`
- command can be used directly
	 ```sh
	 if terraform plan; then
	 fi
	 ```

### Operator
- `-e`: Checks if a file exists
- `-d`: Checks if a directory exists
- `-f`: Checks if a file is a regular file
- `-s`: Checks if a file exist and size > 0
- `-z`: Checks if a file exist and size == 0

## Array

```sh
arr=(1 3 5) 

# iterate every element
echo "${arr[@]}"

# count the number of elements
echo "${#arr[@]}"

# accessing nth element of array
echo "${arr[2]}"

for ((i = 0; i < ${#arr[@]}; i++)); do
echo "element: ${arr[$i]}"
done
```

## command substitution

Command substitution runs a command and replaces it with the command's output.

```sh
eval "$(echo pwd)" #eval runs the result string of `$(echo pwd)`.

eval "$(ssh-agent)"
```

## writing to external file

### multiple lines (here-document)
```sh
cat > $filename <<EOF 
Host
  Hostname 10.20.30.40
  User $user
  IdentityFile $dir
EOF
```

### single line
```sh
echo "text" > $filename
```

- '>' for overwrite, '>>' for adding new lines at the end.