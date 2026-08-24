---
title: cross note command referecne
type: reference
tags: [terminal, commands, reference]
priority: 2
finished: true
created_date: 2026-04-19
---

# Bash commands

## Commands

### Tab Navigation

- Move to previous open tab

```text
cmd + shift + [
```

- Move to next open tab

```text
cmd + shift + ]
```

- Select the tab to open manually

```text
cmd + $n
```

### [[linux-change-user | File permission]]

- list-hidden-files-with-ls-long-format

```sh
ls -al 
```

- identify-the-current-user-with-whoami

```sh
whoami
```

- change-directory-ownership-with-chown

```sh
sudo chown -R $(whoami) node_modules
```

- change owner of `node_modules` for both user and group

```sh
sudo chown -R $(whoami):$(whoami) node_modules
```

- ssh-commands|SSH Commands

```sh
chmod 600 ~/.ssh/id_ed25519
```

- go to root directory

```sh
cd /
```

- go to home directory

```sh
cd ~
```

### Networking And HTTP

- find-port-listener-with-lsof

```sh
lsof -i :3000
```

- inspect-http-requests-with-curl-verbose

```sh
curl -v https://example.com
```

- return header only

```sh
curl -I https://example.com
```

### Tail command
- as default, it displays the last 10 lines of specified file

#### basic use
```sh
tail file
```

#### streamline the update
```sh
tail -F file
```

#### display n last lines
```sh
tail -n 5 file
```

### sed command

#### basic structure
```sh
sed -i 's/old/new/g' file.txt # 's' for substitute
sed -i '' 's/old/new/g' file.txt # only for MacOS # extra '' is needed
sed -i "s/"$var"/new/g" file.txt # use "" around at the end to use variable
```

- substitute old with new
- `-i`: actually edit the file
- `/g': replace everything with 'new', otherwise, only the first element will be replaced
	- `/3g`: apply up to 3rd occurences

#### replace space
```sh
sed -i 's/ /_/g' # replace every space with '_'
```

```sh
sed -i -E 's/ +/_/g' # replace multiple spaces with '_'
```

- `-E`: use Extended regular expression
	- `/ +/`: more than one space
	
#### pattern
- `^`:  start of line  
- `.*`: anything after / before this point

```sh
sed -i 's/^test.*/8080/g' file
```

- Find any line starting with "test", match the rest of that line, and replace it with "8080" directly inside the file.

### grep

- it is to find the 'content' of inside of the specified file, but can be filter out from stdout using pipe

	```sh
	ls -l | grep "test" # find contains `test` string from the std output of `ls- l`
	```

- but it can also serve as primary search command

```sh
grep [option] [search text] [file]
```

```sh
grep $(option) "test" test.txt # find "test" in test.txt
```

#### options

- i: case insensitive
- q: quite mode
	- does not make stdout, the result can be found echo `$?`
		1. `0`: found
		2. `1`: not found
		3. `2`: error
- n: display the line numbers 

### find

`find` is used to search for files and directories based on conditions such as name, type, size, time, permissions, and owner.

```sh
find [path] [conditions] [actions]
```

example: 

```sh
find . -name "*.log"
```

1. Search file by name

```sh
find . -name "*.log"
find . -name "*error*"
find . -iname "error" # case insensitive
find . -name "error" # only find the exact match of "error"
```

2. Search file by type

```sh
find . -type f # find only regular files
find . -type d # find only directories
find . -type f -name "*.log" # common usage example 
```

3. Search file by size

- find files larger than 100 MB

```sh
find . -type f -size +100M
```

- find files smaller than 10 MB

```sh
find . -type f -size -10M
```

- find files around 1GB

```sh
find . -type f -size 1G
```

### date

```sh
"$(date)"
```

### tee

1. `tee` reads from **stdin** and writes the output to **both stdout and a file**.

basic structure
```sh
command | tee file.txt #automatically create the `file.txt` file
```

example
```sh
echo "hello" | tee file.txt
```
It prints `hello` on the terminal and also saves it to `output.txt`.

2. Use `-a` to **append** instead of overwriting, `tee` command basically overwrites.

example
```sh
echo "another log" | tee -a log.tx
```

3. Useful with `sudo` when writing to protected files
```sh
echo "config=value" | sudo tee /etc/app.conf
```

4. use here document to save multiple files
4.1
```sh
tee ouput.txt<<EOF
great
hahaha
EOF
```
4.2
```sh
cat <<EOF | tee output.txt
test 
test2 
test3 
EOF
```
