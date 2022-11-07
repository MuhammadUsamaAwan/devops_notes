**Docker Basic Commands**

- `docker pull [image-name]:[image-version]` = pull an image
- `docker images` = list all the images
- `docker run [image-name]` = run a new container
- `docker ps` = list all running containers
- `docker run [image-name] -d` = run the container in detach mode
- `docker stop [container-id]` = stops the container
- `docker run [container-id]` = start the container
- `docker ps -a` = list all containers
- `docker run -p [host-port]:[container-port] [image-name]` = runs container on host port, container port can be the same
- `docker run --name [container-name] [image-name]` = runs container with name
- `docker logs [container-id | container-name]` = shows logs of container
- `docker logs -f [container-id | container-name]` = follow log output
- `docker logs --tail -n [container-id | container-name]` = number of lines to show from the end of the logs
- `docker exec -it [container-id | container-name]` = get terminal of the container
- `exit` = exit from the terminal
- `docker network ls` = list all networks
- `docker network create [network-name]` = create a network, containers on the same network can talk to each other directly with only names
- `docker run -e [enivornment-variable-name]=[enivornment-variable-value] -e [enivornment-variable-name2]=[enivornment-variable-value2] [image-name]` = setup envornment variables
- `docker run --net [network-name] [image-name]` = run image on a specific network

**Docker Compose**

```yaml
version: '3'
services:
  mongodb:
    image: mongo
    ports:
      - 27017:27017
    environment:
      - MONGO..._USERNAME=admin
```

This will automatically create a common network

- `docker-compose -f [yaml-file] up` = start all container in yaml file
- `docker-compose -f [yaml-file] down` = stops all container in yaml file
