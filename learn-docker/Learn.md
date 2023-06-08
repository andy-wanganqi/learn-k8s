# App service example
```
git clone https://github.com/docker/getting-started.git
```
The root folder is under the ./app of this repo.

# Docker scripts
## Build an image
```
docker build -t getting-started .
```
* **-t** will flag the image

## Run a container with the image
```
docker run -dp 3000:3000 getting-started
```
* **-d** will run the image in "detached" mode (in the background)
* **-p** will create a mapping between the host's port 3000 to the container's port 3000

## List the containers
```
docker ps
```

## Stop a container
```
docker stop <container-id>
```

## Remove a container
```
docker rm <container-id>
```

## Stop and remove a container
```
docker rm -f <container-id>
```
* **-f** is the force flag

## Login to Docker Hub
```
docker login -u YOUR-USER-NAME
```

## Give a new tag to image
```
docker tag getting-started YOUR-USER-NAME/getting-started
```

## Push docker to hub
```
docker push YOUR-USER-NAME/getting-started
```

## Run a container with an image in docker hub
```
docker run -dp 3000:3000 YOUR-USER-NAME/getting-started
```

## Create a volume
```
docker volume create todo-db
```

## Run a container with a volume mount
```
docker run -dp 3000:3000 --mount type=volume,src=todo-db,target=/etc/todos getting-started
```

## Inspect the volume
```
docker volume inspect todo-db
```

## Run a container with a bind mount
```
docker run -it --mount type=bind,src="$(pwd)",target=/src ubuntu bash
```
* **pwd** means the / path in linux

## Launch an app service with a bind mount
```
docker run -dp 3000:3000 \
    -w /app --mount type=bind,src="$(pwd)",target=/app \
    node:18-alpine \
    sh -c "yarn install && yarn run dev"
```
* **-w** means the working directory in the container
* **sh -c** will start a sh shell and run the command

## Watch the logs of an image
```
docker logs -f <container-id>
```

## Create a network
```
docker network create todo-app
```

## Launch a mysql container for the app service 
```
docker run -d \
     --network todo-app --network-alias mysql \
     -v todo-mysql-data:/var/lib/mysql \
     -e MYSQL_ROOT_PASSWORD=secret \
     -e MYSQL_DATABASE=todos \
     mysql:8.0
```
* **--network** will connect the container to the network
* **--network-alias** 
* **-v** will set a volume mount
* **-e** will set an environment variable

## Execute into the container
```
docker exec -it <mysql-container-id> mysql -u root -p
```
* **-i** will use interactive mode
* **-t** will allocate TTL

## Start a container and connect to a network
```
docker run -it --network todo-app nicolaka/netshoot
```

## Launch the app service with mysql environment variables
```
docker run -dp 3000:3000 \
   -w /app -v "$(pwd):/app" \
   --network todo-app \
   -e MYSQL_HOST=mysql \
   -e MYSQL_USER=root \
   -e MYSQL_PASSWORD=secret \
   -e MYSQL_DB=todos \
   node:18-alpine \
   sh -c "yarn install && yarn run dev"
```

## See the layers of an image
```
docker image history getting-started
```

# Docker compose
Use a **docker-compose.yml** file to define docker compse

## Docker compose version
```
docker compose version
```

## Docker compose file
```
docker-compose.yml
```

## Run the application stack
```
docker compose up -d
```

## Tean down the application stack
```
docker compose down
```

