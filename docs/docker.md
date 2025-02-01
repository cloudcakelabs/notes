# Docker

- Build an image

```sh
docker build -t <image_name:tag> .
```

```sh
docker build -t <image_name:tag> -f <dockerfile_path> .
```

- Run a container

```sh
docker run -d -p <host_port>:<container_port> <image_name>
```

- Run a command inside a container

```sh
docker exec -it <container_name or id> <command>
```
