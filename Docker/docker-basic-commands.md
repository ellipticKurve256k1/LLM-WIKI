1. `docker ps`: list every running container
	- `-a`: list every stopped containers 
2. `docker container prune`: delete every stopped containers
3. `docker search <image>`: search for the docker images from docker hub
	- `docker search --filter=stars=10 <image>`
	- `docker search --filter "is-offical=true"`
4. `docker compose`: refer to [[docker-compose]]
5. `docker image ls`: list every pulled images
	- same thing as `docker images`
	- `docker image inspect <image>`
		- to check entrypoint and cmd
6. `docker volume ls`: list every volumes
	- `docker volume rm <volume-name>`
7. 