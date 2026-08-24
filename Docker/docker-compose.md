# Docker Compose Basics

## 1. Basic Declaration

```yaml
services:
  server:
    image: nginx:alpine
    container_name: local_nginx
    working_dir: /app
    ports:
      - "8080:80"
    volumes:
      - .:/app/dev
      - node_modules:/app/dev/node_modules
    restart: unless-stopped
    depends_on:
      - backend
    command: nginx -g "daemon off;"

volumes:
  node_modules:
```

> YAML indentation must use spaces, not tabs.

Also note that `nginx:alpine` does not include Node.js or npm by default.  
Therefore, `command: npm run dev` should normally be used with a Node.js image such as:

```yaml
image: node:22-alpine
```

---

# 2. Structure of `services`

Under `services`:

1. Define a service name.
    
2. Declare the service configuration using Compose properties.
    

```yaml
services:
  app:
    image: node:22-alpine
```

Here, `app` is the service name.

Most Docker Compose commands target the **service name**, not the manually assigned container name.

---

# 3. Common Service Properties

## `image`

Specifies the Docker image used to create the container.

```yaml
image: nginx:alpine
```

---

## `container_name`

Assigns a fixed name to the container.

```yaml
container_name: local_nginx
```

This property is optional.

In Compose commands, you should normally use the service name:

```bash
docker compose exec server sh
```

Not the container name:

```bash
docker compose exec local_nginx sh
```

However, regular Docker commands use the container name:

```bash
docker exec -it local_nginx sh
```

---

## `working_dir`

Sets the working directory inside the container.

The path must be an absolute container path.

```yaml
working_dir: /app
```

Commands executed in the container will start from this directory.

---

## `ports`

Maps a host port to a container port.

```yaml
ports:
  - "8080:80"
```

Format:

```text
HOST_PORT:CONTAINER_PORT
```

In this example:

- The host listens on port `8080`.
    
- Traffic is forwarded to port `80` inside the container.
    

The application must actually listen on the specified container port.

---

## `depends_on`

Defines the service startup order.

```yaml
depends_on:
  - backend
```

This means Compose starts `backend` before starting the current service.

However, `depends_on` only controls startup order. It does not guarantee that the dependency is fully initialized or ready to accept connections.

For readiness checks, use a health check or retry logic in the application.

---

## `restart`

Defines the container restart policy.

```yaml
restart: unless-stopped
```

Common values include:

```yaml
restart: "no"
restart: always
restart: on-failure
restart: unless-stopped
```

`unless-stopped` means the container restarts automatically unless it was manually stopped.

---

## `command`

Overrides the image's default command (aka CMD).

```yaml
command: npm run dev
```

Use `sh -c` when the command contains shell syntax such as:

- `&&`
    
- `||`
    
- pipes
    
- variable expansion
    
- redirection
    

```yaml
command: sh -c "npm install && npm run dev"
```

The `-c` option tells the shell to interpret the following string as a shell command.

For production or repeatable development environments, dependencies are normally installed during the image build:

```dockerfile
RUN npm ci
```

For temporary development workflows, they can also be installed separately:

```bash
docker compose run --rm app npm install
docker compose up
```

---

# 4. Volumes

Volumes connect host files or Docker-managed storage to paths inside containers.

There are two commonly used types:

1. Bind mounts
    
2. Named volumes
    

---

## 4.1 Bind Mount

A bind mount connects an existing host path directly to a container path.

```yaml
volumes:
  - .:/app
```

Format:

```text
HOST_PATH:CONTAINER_PATH
```

Example:

```yaml
volumes:
  - ./src:/app/src
```

Changes made in `./src` on the host are immediately visible in `/app/src` inside the container.

When mounting a single file, specify the filename on both sides:

```yaml
volumes:
  - ./nginx.conf:/etc/nginx/nginx.conf
```

The source and target should both represent files.

---

## 4.2 Named Volume

A named volume is storage managed by Docker.

```yaml
services:
  app:
    volumes:
      - node_modules:/app/node_modules

volumes:
  node_modules:
```

Here:

- `node_modules` is the Docker-managed volume name.
    
- `/app/node_modules` is the mounted directory inside the container.
    
- Files written to `/app/node_modules` are stored in the named volume.
    
- The data remains after the container is recreated.
    

A named volume is useful for:

- Database data
    
- Dependency directories
    
- Persistent application data
    
- Avoiding host-container filesystem conflicts
    

---

# 5. Volume Long Syntax

The long syntax makes the volume type and options more explicit.

## Bind Mount

```yaml
services:
  app:
    volumes:
      - type: bind
        source: .
        target: /app
        read_only: true
```

Properties:

- `type`: mount type
    
- `source`: path on the host
    
- `target`: absolute path inside the container
    
- `read_only`: prevents the container from modifying the mounted data
    

---

## Named Volume

```yaml
services:
  app:
    volumes:
      - type: volume
        source: data-test
        target: /app/test

volumes:
  data-test:
```

`data-test` must be declared in the top-level `volumes` section.

### Mount external named volume

```yaml
services: 
  app:
    volumes:
      - type: volume
        source: data-test
        target: /app/test
        
volumes:
  data-test:
    external: true
    name: project_name_data-test
```

- `project name` is mostly the folder name that has `docker-compose.yml` configuration file.
- or can be searched `Volume Name` using `docker volume ls` 

---

## Using the Home Directory

Use `${HOME}` when referencing the user's home directory.

```yaml
services:
  terraform:
    volumes:
      - type: bind
        source: ${HOME}/.aws
        target: /root/.aws
        read_only: true
```

Equivalent short syntax:

```yaml
volumes:
  - ${HOME}/.aws:/root/.aws:ro
```

This is commonly used to provide AWS credentials to a container.
**${Home} refers to host OS's $HOME.** 

when set **target dir**, use /root instead, and always use **absolute path**

---

# 6. Starting Services

## `docker compose up`

Creates and starts the services declared in `compose.yaml` or `docker-compose.yml`.

```bash
docker compose up
```

Run in detached mode:

```bash
docker compose up -d
```

Detached mode runs the containers in the background.

Start a specific service:

```bash
docker compose up -d app
```

Compose may also start services required by `depends_on`.

---

# 7. Stopping Services

## `docker compose down`

Stops and removes Compose containers and networks.

```bash
docker compose down
```

Remove named volumes declared by the Compose project as well:

```bash
docker compose down -v
```

Be careful with `-v`, especially when the volumes contain database or application data.

---

# 8. Running Commands in a Service

Compose commands normally target the declared service name.

```yaml
services:
  app:
    image: node:22-alpine
```

The target name is `app`.

---

## `docker compose run`

Creates a new one-off container from a service definition and runs a command inside it.

```bash
docker compose run app npm install
```

Recommended form:

```bash
docker compose run --rm app npm install
```

`--rm` removes the one-off container after the command finishes.

Without `--rm`, the stopped container may remain visible in:

```bash
docker ps -a
```

Typical uses include:

- Installing dependencies
    
- Running migrations
    
- Running Terraform commands
    
- Running tests
    
- Starting temporary shells
    

Example:

```bash
docker compose run --rm terraform init
```

---

## Overriding the Entrypoint

Some images define a fixed entrypoint.

For example, the Terraform image uses `terraform` as its entrypoint. To run a shell instead, override it:

```bash
docker compose run --rm --entrypoint sh terraform
```

Run a specific shell command:

```bash
docker compose run --rm --entrypoint sh terraform -c "ls -la"
```

The correct option is:

```text
--entrypoint
```

Not:

```text
-entrypoint
```

---

## `docker compose exec`

Runs a command inside an **already running** service container.

```bash
docker compose exec app npm install
```

Open a shell:

```bash
docker compose exec app sh
```

The service must already be running:

```bash
docker compose up -d
docker compose exec app sh
```

Compose automatically allocates an interactive terminal in normal terminal usage, so `-it` is usually unnecessary.

If the command is used in a non-interactive environment, use:

```bash
docker compose exec -T app command
```

---

# 9. `run` vs `exec`

## `docker compose run`

```bash
docker compose run --rm app npm install
```

- Creates a new temporary container.
    
- Uses the service configuration.
    
- Does not require the main service container to be running.
    
- Useful for one-time commands.
    
- Does not publish the service's ports by default.
    
- Can be removed automatically with `--rm`.
    

## `docker compose exec`

```bash
docker compose exec app npm install
```

- Runs the command inside an existing container.
    
- Requires the service to already be running.
    
- Uses the container's current filesystem and runtime state.
    
- Useful for debugging or inspecting a running service.
    

---

# 10. `docker exec` vs `docker compose exec`

## Compose command

Uses the service name:

```bash
docker compose exec app sh
```

## Regular Docker command

Uses the actual container name or container ID:

```bash
docker exec -it local_nginx sh
```

When working with a Compose project, prefer:

```bash
docker compose exec
```

This keeps commands aligned with the service definitions in the Compose file.

---

# 11. Typical Development Workflow

```bash
docker compose run --rm app npm install
docker compose up -d
docker compose exec app sh
docker compose logs -f app
docker compose down
```

For a cleaner and more reproducible setup, dependency installation should normally be placed in a Dockerfile:

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci // npm clean install

COPY . .

CMD ["npm", "run", "dev"]
```

Then build and start the service:

```bash
docker compose up -d --build
```

This avoids reinstalling dependencies manually every time.

# 12. [[dockerfile]]

```bash
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    image: built-image
	...
```