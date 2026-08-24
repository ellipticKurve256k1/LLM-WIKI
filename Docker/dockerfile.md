# Dockerfile Overview

A `Dockerfile` is a text file that contains instructions for building a Docker image.

It defines the base image, application files, dependencies, environment variables, startup command, and other image configuration.

## Basic Example

```dockerfile
FROM node:22

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 8080

CMD ["npm", "start"]
```

## Common Instructions

### `FROM`

Defines the base image.

```dockerfile
FROM ubuntu:24.04
```

Every Dockerfile normally starts with a `FROM` instruction.

The base image must support the target CPU architecture, such as:

- `linux/amd64`
    
- `linux/arm64`
    

The same Dockerfile can produce different architecture-specific images depending on the build platform.

---

### `WORKDIR`

Sets the working directory inside the image.

```dockerfile
WORKDIR /app
```

Following instructions such as `COPY`, `RUN`, and `CMD` use this directory as their default location.

---

### `COPY`

Copies files from the build context into the image.

```dockerfile
COPY . .
```

The first path refers to the host build context, and the second path refers to the destination inside the image.

```dockerfile
COPY package.json /app/package.json
```

Files excluded by `.dockerignore` cannot be copied into the image.

When include multiple files, then the last argument becomes the destination of container

```dockerfile
COPY index.html package.json /app
```

---

### `RUN`

Executes a command while the image is being built.

```dockerfile
RUN apt-get update && apt-get install -y curl
```

Each `RUN` instruction creates a new image layer.

Related commands should usually be combined to reduce unnecessary layers.

```dockerfile
RUN apt-get update \
    && apt-get install -y curl git \
    && rm -rf /var/lib/apt/lists/*
```

---

### `ENV`

Defines environment variables inside the image.

```dockerfile
ENV NODE_ENV=production
```

These variables are available during later build steps and when the container runs.

Runtime values can override them.

```bash
docker run -e NODE_ENV=development my-app
```

---

### `EXPOSE`

Documents the port used by the application inside the container.

```dockerfile
EXPOSE 8080
```

`EXPOSE` does not publish the port to the host.

The port must be mapped when the container starts.

```bash
docker run -p 8080:8080 my-app
```

The syntax is:

```text
host_port:container_port
```

A protocol can also be specified.

```dockerfile
EXPOSE 8080/tcp
EXPOSE 53/udp
```

TCP is the default protocol.

---

### `CMD`

Defines the default command executed when the container starts.

```dockerfile
CMD ["npm", "start"]
```

Only the final `CMD` instruction is used.

A command provided through `docker run` overrides `CMD`.

```bash
docker run my-app npm run dev
```

The JSON array form is generally preferred.

```dockerfile
CMD ["python", "app.py"]
```

---

### `ENTRYPOINT`

Defines the main executable for the container.

```dockerfile
ENTRYPOINT ["python", "app.py"]
```

Arguments passed through `docker run` are appended to the entrypoint.

```bash
docker run my-app --debug
```

This effectively runs:

```bash
python app.py --debug
```

`ENTRYPOINT` is useful when the image should behave like a dedicated executable.

---

## `.dockerignore`

A `.dockerignore` file excludes unnecessary files from the Docker build context.

Example:

```dockerignore
node_modules
.git
.env
*.log
docker-compose.yml
compose.yml
```

This improves build performance, reduces image size, and prevents sensitive or unnecessary files from being included in the image.

Ignoring `docker-compose.yml` does not prevent Docker Compose from using it.

Docker Compose reads the file directly from the host.

```bash
docker compose up
```

The file is only excluded from the image build context.

For example, this is valid:

```dockerignore
docker-compose.yml
```

The Compose command still works, but the file is not included by:

```dockerfile
COPY . .
```

## Build Context

The final argument of `docker build` defines the build context.

```bash
docker build -t my-app .
```

Here, `.` means the current directory is sent as the build context.

Only files inside the build context can be accessed by `COPY` or `ADD`.

```text
project/
├── Dockerfile
├── .dockerignore
├── package.json
├── docker-compose.yml
└── src/
```

## Building an Image

```bash
docker build -t my-app:latest .
```

- `-t` assigns an image name and tag.
    
- `my-app` is the image name.
    
- `latest` is the image tag.
    
- `.` is the build context.
    

## Building for a Specific CPU Architecture

```bash
docker buildx build \
  --platform linux/amd64 \
  -t my-app:amd64 \
  --load .
```

For ARM64:

```bash
docker buildx build \
  --platform linux/arm64 \
  -t my-app:arm64 \
  --load .
```

A multi-platform image can also be built and pushed to a registry.

```bash
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t username/my-app:latest \
  --push .
```

The base image and all included binaries must support the selected architecture.

## Dockerfile and Docker Compose

A Dockerfile defines how to build one image.

```text
Dockerfile
    ↓
Docker image
    ↓
Container
```

A Compose file defines how one or more containers should run together.

```yaml
services:
  app:
    build: .
    ports:
      - "8080:8080"
```

The responsibilities are different:

|File|Purpose|
|---|---|
|`Dockerfile`|Defines how an image is built|
|`.dockerignore`|Excludes files from the build context|
|`compose.yml`|Defines how containers are configured and started|

## Summary

A Dockerfile is a reproducible recipe for creating a Docker image.

It usually defines:

- The base image
    
- The application working directory
    
- Files copied into the image
    
- Dependencies installed during the build
    
- Environment variables
    
- The application port
    
- The default startup command
    

`EXPOSE` documents an internal container port but does not publish it.

`.dockerignore` removes files such as `node_modules`, `.git`, `.env`, and Compose files from the build context without affecting Docker Compose itself.