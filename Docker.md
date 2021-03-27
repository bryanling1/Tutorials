# Docker 

[Stephen Grider Course](https://www.udemy.com/course/docker-and-kubernetes-the-complete-guide/learn/lecture/18799500?start=0#overview)

## Table of Contents
- [Docker](#docker)
  - [Table of Contents](#table-of-contents)
  - [Diving into Docker](#diving-into-docker)
      - [`docker run hello-world`](#docker-run-hello-world)
      - [What is a container?](#what-is-a-container)
  - [Manipulating Containers with Docker Client](#manipulating-containers-with-docker-client)
      - [`docker run`](#docker-run)
        - [`docker run busybox ls`](#docker-run-busybox-ls)
      - [`docker ps`](#docker-ps)
        - [`--all`](#--all)
      - [`docker create`](#docker-create)
      - [`docker start`](#docker-start)
        - [`-a`](#-a)
      - [`docker system prune`](#docker-system-prune)
      - [`docker logs <containerid>`](#docker-logs-containerid)
      - [`docker stop <container id>`](#docker-stop-container-id)
      - [`docker kill <container id>`](#docker-kill-container-id)
      - [`docker exec -it <container id> <command>`](#docker-exec--it-container-id-command)
        - [How `-it` works](#how--it-works)
  - [Building Custom Images Through Docker Server](#building-custom-images-through-docker-server)
      - [Basic Dockerfile structure](#basic-dockerfile-structure)
      - [What  is a Base Image?](#what--is-a-base-image)
      - [The build process in detail](#the-build-process-in-detail)
      - [Tagging an image with `-t`](#tagging-an-image-with--t)
      - [Manual Image Generation with Docker Commit](#manual-image-generation-with-docker-commit)
  - [NodeJS web app Demo Project](#nodejs-web-app-demo-project)
      - [`Dockerfile`](#dockerfile)
      - [Container Port Mapping](#container-port-mapping)
        - [`docker run -p 8080:8080 <imageid>`](#docker-run--p-80808080-imageid)

## Diving into Docker
<a href="https://ibb.co/ZxzC8KV"><img src="https://i.ibb.co/9WHjT2N/image.png" alt="image" border="0"></a>

#### `docker run hello-world`
<a href="https://ibb.co/jWYgJvX"><img src="https://i.ibb.co/rch4dGr/image.png" alt="image" border="0"></a>

#### What is a container?

<a href="https://ibb.co/qsZW8CZ"><img src="https://i.ibb.co/J7hyTqh/image.png" alt="image" border="0"></a>

<a href="https://ibb.co/v4shzfn"><img src="https://i.ibb.co/Mk8fNK0/image.png" alt="image" border="0"></a>

<a href="https://ibb.co/55n3MF2"><img src="https://i.ibb.co/ZhYrLmN/image.png" alt="image" border="0"></a>

 - `Namespacing` and `Control Groups` are specific to `Linux` and note found in all OS
 - When we install `Docker`, We installed a **Linux Virtual Machine**

<a href="https://imgbb.com/"><img src="https://i.ibb.co/Snjj9Fg/image.png" alt="image" border="0"></a>

<a href="https://ibb.co/KwzKSrf"><img src="https://i.ibb.co/pyrh5LM/image.png" alt="image" border="0"></a>

## Manipulating Containers with Docker Client

#### `docker run`

A comginationg of `docker create` and `docker start`

<a href="https://ibb.co/0fQCDv5"><img src="https://i.ibb.co/4jJ8m3k/image.png" alt="image" border="0"></a>

##### `docker run busybox ls`

<a href="https://ibb.co/3CJxsB9"><img src="https://i.ibb.co/RD5XSzk/image.png" alt="image" border="0"></a>

If we tried to run `docker run hello-word ls` we would get an error because `ls` is not listed as a command in that image

#### `docker ps`

##### `--all`
- Running and non-runnign containers

#### `docker create`

creates a writeable container layer over the specified image

- Set's up the **file system snapshot**
- This (also includeing `docker run`) is where the `COMMAND` is set. We cannot change this. 
  - If we were to restart the container, the same command would run

#### `docker start`

Start a container

##### `-a`

The `-a` watches output from the container and prints it to the terminal


#### `docker system prune`

Removes
- All stopped containers
- All networks not used by a t least one container
- All dangling images
- All build cache (need to re-get images)

#### `docker logs <containerid>`

Gets all the logs from the container

#### `docker stop <container id>`

<a href="https://ibb.co/fXhbJmJ"><img src="https://i.ibb.co/x2pyZVZ/image.png" alt="image" border="0"></a>

- Includes cleanup
- If the container does not `stop` within 10 seconds, `docker kill` is run

#### `docker kill <container id>`

kills the container

#### `docker exec -it <container id> <command>`

-`-i` attack terminal to `stdin`
-`-t` makes the text formatted pretty
-We can use `sh` as `<command>` to open a `shell` in context of the container

##### How `-it` works

<a href="https://ibb.co/7G3NY2w"><img src="https://i.ibb.co/Yt5TRpz/image.png" alt="image" border="0"></a>

## Building Custom Images Through Docker Server

<a href="https://ibb.co/QJWJ72J"><img src="https://i.ibb.co/9pxpmFp/image.png" alt="image" border="0"></a>
#### Basic Dockerfile structure

```docker
# Use an existing docker image as a base
FROM alpine

# Download and isntall a dependency
RUN apk add --update redis

# Tell the image what to do when it starts as a container

CMD ["redis-server"]
```

#### What  is a Base Image?

<a href="https://ibb.co/GvqVVBH"><img src="https://i.ibb.co/0jxff8s/image.png" alt="image" border="0"></a>

#### The build process in detail

For every `step` in the build process, docker creates a new `temporary container` and passes the previous step's `image` to it.

Each command runs as that container's `primary process`.

The last step places the `primary process` but does not run it

<a href="https://ibb.co/j4Gm7xD"><img src="https://i.ibb.co/4NMzBDp/image.png" alt="image" border="0"></a>

- Every snapshot taken stores the `FS` and `Startup Command`

#### Tagging an image with `-t`

We prefix with our `Docker ID`

```
docker build -t bryanling/tagName:latest -t . 
```
- without `:latest` is latest by default

#### Manual Image Generation with Docker Commit

We won't use this very often. 

We can create a container and build an image from it.

```
docker run -it alpine sh
```

Manuall install `redis`

```
apk add --update redis
```

Find the container

```
docker ps
```

Commit

```
docker commit -c 'CMD "redix-server"' 123456
```
- **TIP:** We can proved just the start of the `container ID` and docker will try and find it for us

The run the image

```
docker run abcd1232131
```

## NodeJS web app Demo Project

<a href="https://imgbb.com/"><img src="https://i.ibb.co/cyx0yWY/image.png" alt="image" border="0"></a>

#### `Dockerfile`

```docker
FROM node:alpine

WORKDIR server

COPY . .

RUN npm install

CMD ["npm", "start"]
```
- `alpine` is the tag, means like smallest version

#### Container Port Mapping 

<a href="https://ibb.co/PzJqvGy"><img src="https://i.ibb.co/zHCqTVg/image.png" alt="image" border="0"></a>

##### `docker run -p 8080:8080 <imageid>`
 - First number is Route incoming requests
 - Second number is port inside the `container`
 - `8080` and `8080` ports do not ahve to be identical

