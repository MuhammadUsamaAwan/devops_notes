**Docker Basic Commands**

- `docker pull [image-name]:[image-version]` = pull an image
- `docker images` = list all the images
- `docker run [image-name]` = run a new container
- `docker ps` = list all running containers
- `docker run -d [image-name]` = run the container in detach mode
- `docker stop [container-id]` = stops the container
- `docker start [container-id]` = start the container
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
- `docker rmi [image-name]` = delete image that is not used by a running container
- `docker rmi [image-name] -f` = force delete image even if used by a running container
- `docker rm [container]` = delete a container
- `docker container prune` = removes all stopped containers
- `docker image prune` = removes all unused or dangling images (images that do not have a tag)
- `docker system prune` = removes all stopped containers, dangling images, and dangling build caches

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

**Dockerfile**

```dockerfile
FROM node:13-alphine

# Create app directory
WORKDIR /usr/src/app

# Install app dependencies
COPY package*.json ./
RUN npm install

# Bundle app source
COPY . .

# Expose the port
EXPOSE 8080

# Run node
CMD [ "node", "server.js" ]
```

`docker build -t [name]:[tag] [PATH]` = build a docker image based on the docker file

**Docker Registry**

Using AWS ECR. Create a repository. Click view push commands to view commands for pushing the image to AWS.

**Deploy Docker Application**

- First config the Dockerfile
  ```yaml
  version: '3'
  services:
  [image-name]:
    image: [image-registry-url]
    ports:
      - 3000:3000
  mongodb:
  ```
- Change you mongo url from localhost to mongodb. It can now directly talk to the mongodb because it is on the same network.
- Make sure you login `docker login` before running the container
- `docker-compose -f [yaml-file] up` = run all the images

**Docker Volumes**

Types:

- Host Volume = `-v [host-directory]:[container-directory]`, you decide where on the host file system the reference is made
- Anonymous Volume = `-v [container-directory]`, you don't specify where on the host file system the reference is made, you can reference the volume by the path
- Name Volume = `-v name:[container-directory]`, you don't specify where on the host file system the reference is made, you can reference the volume by the name
- container paths; mongo = data/db, mysql = var/lib/mysql, postgres = var/lib/postgresql/data

In docker compose file

```yaml
services:
  mongodb:
    volumes:
      - mongo-data:/data/db
volumes:
  mongo-data:
    driver: local
```

**Pull/Push Nexus Repository**

- Create a docker hosted repository
- Create a new role with docker privileges
- Apply the role to the user
- Create a reposity container for docker hosted repository, check the HTTP connect and specify a port
- Open this port on firewall
- Go to realms and activate docker bear token realm
- Edit the docker to allow HTTP communication; docker ui>preferences>docker engine>add this

  ```json
  "insecure-registries": ["[nexus-ip]:[repository-connector-port]"]
  ```

- `docker login [nexus-ip]:[nexus-port]`, enter nexus username and password to login
- `docker tag [image-name]:[image-tag] [nexus-ip]:[nexus-port]/[image-name]:[image-tag]` = tag image for nexus
- `docker push [image-name]` = push the image to nexus

**Run Nexus as Docker Container**

Instead of installing java and nexus we can also create a nexus container

- `docker volume create --name nexus-data`
- `docker run -d -p 8081:8081 --name nexus -v nexus-data:/nexus-data sonatype/nexus3`
- Creating a nexus user and starting nexus with that user will be done out of the box
