# Docker

- Build an image

```sh
docker build -t <image_name:tag> .
```

```sh
docker build -t <image_name:tag> -f <dockerfile_path> .
```

```sh
docker build -t <image_name:tag1> -t <image_name:tag2> -f <dockerfile_path> .
```

- Run a container

```sh
docker run -d -p <host_port>:<container_port> <image_name>
```

- Run a command inside a container

```sh
docker exec -it <container_name or id> <command>
```

- Login to the registry

```sh
docker login -u <username>
```

- Remove all images

```sh
docker rmi $(docker images -a -q)
```
