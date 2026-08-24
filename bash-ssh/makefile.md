# Makefile

A **Makefile** is a file used by the `make` command to automate repetitive tasks such as building, running, testing, or cleaning a project.

A Makefile usually contains:

- **Target**: The task name
    
- **Dependencies**: Files or tasks required before running the target
    
- **Recipe**: Shell commands executed by the target
    

## Basic Syntax

```makefile
target: dependencies
	command
```

> The command line must begin with a **tab**, not spaces.

## Example

```makefile
build:
	docker build -t my-app .

run:
	docker run --rm -p 8080:8080 my-app

clean:
	docker rmi my-app
```

Run a target with:

```sh
make build
make run
make clean
```

## Variables

Variables can be defined and reused inside a Makefile.

```makefile
IMAGE_NAME = my-app
PORT = 8080

build:
	docker build -t $(IMAGE_NAME) .

run:
	docker run --rm -p $(PORT):$(PORT) $(IMAGE_NAME)
```

Variables can also be overridden from the command line:

```sh
make run PORT=3000
```

## `.PHONY`

Use `.PHONY` for targets that are task names rather than actual files.

```makefile
.PHONY: build run clean
```

This prevents conflicts when a file with the same name as a target exists.

## Example for a Shell Script

```makefile
.PHONY: setup

setup:
	chmod +x docker_setup.sh
	./docker_setup.sh
```

Run it with:

```sh
make setup
```

The Makefile itself does not usually require a separate download. On Linux and macOS, you only need the `make` command installed.