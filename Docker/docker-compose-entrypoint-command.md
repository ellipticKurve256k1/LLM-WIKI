# Docker Compose: `entrypoint` and `command`

## `entrypoint`

`entrypoint` defines the main executable that runs when the container starts.

```yaml
services:
  app:
    image: python:3.11
    entrypoint: python3
```

## `command`

`command` overrides the image's default `CMD`.

If an `ENTRYPOINT` exists, `command` is passed as arguments to it.

```yaml
services:
  app:
    image: python:3.11
    entrypoint: python3
    command: app.py
```

Final command:

```bash
python3 app.py
```

## Command Priority

```text
Image CMD
→ overridden by Compose command
→ overridden again by arguments after docker compose run
```

Example:

```yaml
services:
  aws:
    image: amazon/aws-cli
    command: sts get-caller-identity
```

```bash
docker compose run --rm aws s3 ls
```

Final command:

```bash
aws s3 ls
```

## Overriding the Entry Point

```bash
docker compose run --rm --entrypoint sh aws -c "ls"
```

This replaces the original entry point with `sh`.

Final command:

```bash
sh -c "ls"
```

## Basic Rule

```text
entrypoint = executable
command    = default command or arguments
```