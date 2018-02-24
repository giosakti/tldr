# Docker

```
<image> -> <namespace>/<image>
```

## Notable commands

```
docker images
docker run <image>:<image-tag>
docker run -i -t <image>:<image-tag> <command>
docker run -d <image>:<image-tag> <command>
docker run --rm <image>:<image-tag> <command>
docker run --name <name> <image>:<image-tag>
docker ps
docker ps -a
docker stop <container-id>
docker rm <container-id>
docker rmi <image-id>
docker inspect
docker run -it -p <host-port>:<container-port> <image>:<image-tag>
docker logs <container-id>
docker tag <image-id> <image>:<image-tag>
docker login --username=<username>
docker push <image>:<image-tag>
docker exec -it <container-id> <command> # run command inside a running container
docker network ls
docker network inspect <network-name>
docker network create --driver <network-type> <network-name>
docker network connect <network-name> <container-name>
docker network disconnect <network-name> <container-name>

docker-machine ls

docker-compose up
docker-compose ps
docker-compose logs
docker-compose logs -f
docker-compose logs <container-name>
docker-compose stop
docker-compose rm
docker-compose build # rebuild all images
docker-compose run <service-name> <command>
```

## Creating a docker image

approach 1: Commit changes made in a docker container

```
docker commit <container-id> <image>:<image-tag>
```

approach 2: Write a Dockerfile

```
FROM debian:jessie

# Notes:
# - RUN can be chained by '&&' to avoid creating a new layer
# - Please sort package installation alphanumerically
RUN apt-get update
RUN apt-get install -y git
RUN apt-get install -y vim

# CMD: instructions to be run when container is started
# CMD ["echo", "hello world"]

# COPY: instruction for copying files or directories to the container
# COPY <host-path> <container-path>

# ADD: allows you not only to copy files but also download from the internet
#      it also has the ability to automatically unpack compressed files
#

# USER: change active user
# USER <username>

# WORKDIR: change current working directory
# WORKDIR <path>

$ docker build -t <image>:<image-tag> <build-path>
```

Be aware of docker cache

1. each `RUN` command maybe cached from previous execution, solution: chain the `RUN`
2. specify `--no-cache=true` when building image

## Docker Container Links

Run this command on container that want to access the other container.

```
docker run -d -p <host-port>:<container-port> --link <dest-container-name> <image>:<image-tag>
```

Docker container links works by defining host mapping in the `/etc/hosts` file.

## Docker Compose

```
version: '3'
services:
  dockerapp:
    build: .
    ports:
      - "<host-port>:<container-port>"
    depends_on:
      - redis
    networks:
      - my_net
  redis:
    image: redis:3.2.0
    networks:
      - my_net

networks:
  my_net:
    driver: bridge
```

Version 2 or above add automatic linking feature. So we don't have to specify links specifically anymore.

Run all containers using this following command:

```
docker-compose up
```

Docker compose will create new bridged network by default, which will be removed when we issue the `docker-compose down` command.

## Docker Volume

It is not recommended to put data inside a container due to its volatile nature. Use docker volume instead.

## Docker Network Types

- Closed Network / None Network
- Bridge Network (default)
- Host Network
- Overlay Network

```
docker run -d --net <network-name> <image>:<image-tag> <command>
```

## Docker Machine

```
docker-machine ls
docker-machine create --driver <driver-name> <vm-name>
docker-machine env <vm-name>

<on the vm>

docker info

version: '3'
services:
  dockerapp:
    image: <image>:<image-tag>
    ports:
      - "<host-port>:<container-port>"
    depends_on:
      - redis
    networks:
      - my_net
  redis:
    image: redis:3.2.0
    networks:
      - my_net

networks:
  my_net:
    driver: bridge

docker-compose -f prod.yml up -d
```
