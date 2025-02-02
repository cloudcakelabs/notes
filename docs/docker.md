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

- Create a new tag for a remote registry

```sh
docker tag <image_name:tag> <registry_url>/<repository>/<image_name:tag>
```

- Push an image to a remote registry

```sh
docker push <registry_url>/<repository>/<image_name:tag>
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

- Limiting resources

```sh
docker run -d --name <container_name> \
--publish 8080:8080 \
--memory 200m \
--memory-swap 1G \
--cpu-shares 1024 \
<image_name>
```
