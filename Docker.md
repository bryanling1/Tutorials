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
  - [Project 1: NodeJS web app Demo Project](#project-1-nodejs-web-app-demo-project)
      - [`Dockerfile`](#dockerfile)
      - [Container Port Mapping](#container-port-mapping)
        - [`docker run -p 8080:8080 <imageid>`](#docker-run--p-80808080-imageid)
  - [Project 2: Number of Visits with `docker compose`](#project-2-number-of-visits-with-docker-compose)
      - [server code](#server-code)
      - [Assembling node docker file](#assembling-node-docker-file)
      - [Introducing Docker Compose](#introducing-docker-compose)
      - [Docker compose file](#docker-compose-file)
      - [Networking with Docker Compose](#networking-with-docker-compose)
      - [Docker Compose Commands](#docker-compose-commands)
      - [Stopping Docker Compose Containers](#stopping-docker-compose-containers)
      - [Container Restarts](#container-restarts)
      - [Getting Container Status](#getting-container-status)
  - [Project #3 Development Workflow](#project-3-development-workflow)
      - [Flow Specifics](#flow-specifics)
      - [Creating the Dev Dockerfile](#creating-the-dev-dockerfile)
      - [Starting the container](#starting-the-container)
      - [Docker Volumes](#docker-volumes)
      - [Shorthand with Docker Compose](#shorthand-with-docker-compose)
      - [React App Exited With Code 0](#react-app-exited-with-code-0)
      - [Windows not Detecting Changes](#windows-not-detecting-changes)
      - [Docker Compose for Running Tests](#docker-compose-for-running-tests)
      - [Need for NginX](#need-for-nginx)
      - [Creating `Dockerfile.prod` with Multi-step build process](#creating-dockerfileprod-with-multi-step-build-process)
  - [Project #3 continued: CI and Deployment with AWS](#project-3-continued-ci-and-deployment-with-aws)
      - [Github setup](#github-setup)
      - [Travis CI Setup](#travis-ci-setup)
      - [Travis YML File Configuration](#travis-yml-file-configuration)
      - [Important info about AWS Platform Versions](#important-info-about-aws-platform-versions)
      - [AWS Elastic Beanstalk](#aws-elastic-beanstalk)

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

-`-i` attach terminal to `stdin`
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

## Project 1: NodeJS web app Demo Project

<a href="https://imgbb.com/"><img src="https://i.ibb.co/cyx0yWY/image.png" alt="image" border="0"></a>

#### `Dockerfile`

```docker
FROM node:alpine

WORKDIR /usr/server

COPY ./package.json ./
RUN npm install

COPY ./ ./

CMD ["npm", "start"]
```
- `alpine` is the tag, means like smallest version
- `WORKDIR` creates a working folder, also affects commands in `Dockerfile` and running `sh`
- `COPY ./package.json ./` for caching as we don't want to have to install packages every time we change the source code

#### Container Port Mapping 

<a href="https://ibb.co/PzJqvGy"><img src="https://i.ibb.co/zHCqTVg/image.png" alt="image" border="0"></a>

##### `docker run -p 8080:8080 <imageid>`
 - First number is Route incoming requests
 - Second number is port inside the `container`
 - `8080` and `8080` ports do not ahve to be identical


## Project 2: Number of Visits with `docker compose`

<a href="https://ibb.co/RhYRhXj"><img src="https://i.ibb.co/3f4gfxR/image.png" alt="image" border="0"></a>

#### server code

```js
const express = require('express');
const redix = require('redis');

const app = express();
const client = redis.createClient();
client.set('visits', 0);

app.get('/', (req, res)=>{
    client.get('visits', (err, visits)=>{
        res.send(`Number of visits is ${visits}`);
        client.set('visits', parseInt(visits) + 1))
    })
})

app.listen(80801, () => {
    console.log('listening on port 8081'):
})
```

#### Assembling node docker file

```docker
FROM node:alpine

WORKDIR '/app'

COPY ./package.json ./
RUN npm install

COPY ./ ./

CMD ["npm", "start"]
```

Now building the image

```
docker build -t bryanling/visists .
```

If we try to start this, it will fail to connect to redis
#### Introducing Docker Compose

The redis instance we will use is 
```
docker run redis
```

So for our Node App and Redis container cannot communicate with eachother

We can use `Docker CLI` for network features, but it is a pain

`Docker Compose` aims to fix this

`Docker Compose` automates some of the long-winded arguments we were passing to `docker run`

<a href="https://imgbb.com/"><img src="https://i.ibb.co/74Mvf6s/image.png" alt="image" border="0"></a>

#### Docker compose file

Inside `docker-compose.yml`

```yml
version: '3'
services:
    redis-server:
        image: 'redis'
    node-app:
        build: ./
        ports: 
            - "4001:8081"
```
- `version` is the version of `docker-compose`
- We list our containers in `services`
- `redis-server` is the `name` we camp up with for that container

#### Networking with Docker Compose

Let's connect to our redis server inside `index.js`

```js
...
const client = redis.createClient({
    host: 'redis-server',
    port: 6379
})
...
```
- The `url` in the context of containers is the `name` we assigned to the container earlier. Docker figures out the `url` for us
- `6379` is the default port for redis-server

#### Docker Compose Commands

<a href="https://ibb.co/gz94VB5"><img src="https://i.ibb.co/nzmPC5H/image.png" alt="image" border="0"></a>

- `docker-compose` will automatically create our network for us

#### Stopping Docker Compose Containers

<a href="https://imgbb.com/"><img src="https://i.ibb.co/rdGSyrG/image.png" alt="image" border="0"></a>

#### Container Restarts

Sometimes our containers will crash or hang

<a href="https://ibb.co/7n3CF9p"><img src="https://i.ibb.co/QPWQG5H/image.png" alt="image" border="0"></a>

<a href="https://ibb.co/1s7BSbQ"><img src="https://i.ibb.co/gvmQn69/image.png" alt="image" border="0"></a>

Back in `docker-compose.yml` we can define which `restart` policy we want to use

```yml
...
    node-app:
        build: ./
        ports: 
            - "4001:8081"
        restart: on-failure
```
- We need to add quotes for `no` because of yaml files

#### Getting Container Status

We can see the container status specific to `docker-compose` with:

```sh
docker-compose ps
```
- This should be run in the same directory as `docker-compose.yml`

## Project #3 Development Workflow

#### Flow Specifics

<a href="https://ibb.co/L06Vd5J"><img src="https://i.ibb.co/hgH3FfZ/image.png" alt="image" border="0"></a>

<a href="https://ibb.co/RDPb5qM"><img src="https://i.ibb.co/qr9jZv4/image.png" alt="image" border="0"></a>

We will create a create-react-app project, run it in a container, and publish it and create a `development` and `production` setup

#### Creating the Dev Dockerfile

Inside `Dockerfile.dev`

```docker
FROM node:alpine

WORDKIR /app

COPY package.json .
RUN npm install

COPY . .

CMD ["npm", "run", "start"]
```

To build this image, we run:

```
docker build -f Dockerfile.dev .
```

We should delete the `node_modules` folder inside our computer's directory so that `docker` does not copy that folder over 

#### Starting the container

To run our container

```
docker run -it -p 3000:3000 <containerId>
```
- We need the `-it` flag so the react-app does not close

#### Docker Volumes

<a href="https://ibb.co/cv2FR26"><img src="https://i.ibb.co/DbwCmw9/image.png" alt="image" border="0"></a>

- Similar to mapping ports
  
<a href="https://ibb.co/2jqmWKH"><img src="https://i.ibb.co/dQB1jKx/image.png" alt="image" border="0"></a>

- `pwd` in bash means Personal Working Directory
- When we use `:` it means map
- If there is **no** `:` that means "Don't try to map it with anything". We do this because we deleted `node_modules` in our Local Folder
  
#### Shorthand with Docker Compose

Inside `docker-compose.yml`: 
```yml
version: '3'
services:
    web:
        build:
            context: .
            dockerfile: Dockerfile.dev
        ports: 
            - "3000:3000"
        volumes:
            - /app/node_modules
            - .:/app

```

We could now delete `COPY . .` inside our `Dockerfile.dev`, But we should leave it just in case we no longer user `docker-compose`
#### React App Exited With Code 0

To resolve this bug, in `docker-compose.yml`: 
```yml
web:
    std-in: true
```

#### Windows not Detecting Changes

If on windows and react app isi not automatically reloading after a code change:

```yml
services:
  web:
    environment:
      - CHOKIDAR_USEPOLLING=true
```

#### Docker Compose for Running Tests

We will create a second service in our `docker-compose.yml`

```yml
services: 
    ...
    tests:
        build:
            context: .
            dockerfile: Dockerfile.dev
        volumes: 
            - /app/node_modules
            - .:/app
        command: ["npm", "run", "test"]
```
- overwrie the start command with `command`

But now we don't have direct access to the commandline for `jest`

We want to attack our terminal to the docker container

<a href="https://imgbb.com/"><img src="https://i.ibb.co/SJTrHQQ/image.png" alt="image" border="0"></a>

```
docker attach <container id>
```

But even then we can't directly interact the `jest` container

<a href="https://ibb.co/ftLdX6n"><img src="https://i.ibb.co/82tgmvd/image.png" alt="image" border="0"></a>

If we run `ps` inside the container, we'll notice that our `npm run test` is just running npm. That is because the running of the test is a **secondary process**, which we cannot directly access with `docker attach`

#### Need for NginX

<a href="https://ibb.co/BNmgJKp"><img src="https://i.ibb.co/tXSb6PG/image.png" alt="image" border="0"></a>

<a href="https://ibb.co/rc73PVh"><img src="https://i.ibb.co/YyRX9xS/image.png" alt="image" border="0"></a>

Nginx is our Production Server

#### Creating `Dockerfile.prod` with Multi-step build process

<a href="https://ibb.co/MCMQRFg"><img src="https://i.ibb.co/W2gjcmH/image.png" alt="image" border="0"></a>

<a href="https://ibb.co/CWVTc6P"><img src="https://i.ibb.co/jH31NzJ/image.png" alt="image" border="0"></a>

```docker
FROM node:alpine as builder
WORKDIR /app
COPY package.json . 
RUN npm install
COPY . .
RUN npm run build

FROM nginx
COPY --from=builder /app/build /usr/share/nginx/html
```
- we need to remove the `builder` tag if we want to deploy to AWS. Replace builder with `0` on the last line
- Docker recognizes a phase by seeing another `FROM` statement
- `/usr/share/nginx/html` is from nginx documentation

Now we can build the image
```
docker build . 
```

And run it
```
docker run -p 8080:80 <containerId>
```
- default port is `80` for `nginx`

## Project #3 continued: CI and Deployment with AWS

<a href="https://ibb.co/nD8MgBb"><img src="https://i.ibb.co/2jY3nSy/image.png" alt="image" border="0"></a>
#### Github setup

Make a github repo with our files

#### Travis CI Setup

[Login to Travis with Github](https://travis-ci.org/)

Enable our github repo we want to activate

<a href="https://ibb.co/1mzD57r"><img src="https://i.ibb.co/P6D7nFc/image.png" alt="image" border="0"></a>

#### Travis YML File Configuration

<a href="https://ibb.co/dP6mp9f"><img src="https://i.ibb.co/Lx6Ngw0/image.png" alt="image" border="0"></a>
- Using `Docker.dev` because we want to **run tests**

Create `.travis.yml` in our root directory

```yml
sudo: required
language: generic
services: 
    - docker
before_install:
 - docker build -t bryanling/docker-react -f Dockerfile.dev .

script:
  - docker run -e CI=true bryanling/docker-react npm run test
```
- `before_install` before running test/deploying

When we make a new push to our repo, we should see this:

<a href="https://ibb.co/DVyJkvH"><img src="https://i.ibb.co/1v5gKPt/image.png" alt="image" border="0"></a>

#### Important info about AWS Platform Versions

On October 6, AWS made some significant platform changes that will affect our `single container application`. Among these changes was adding the full support of using `Docker Compose` to provision a production AWS environment. This means that **Elastic Beanstalk** will no longer first look for a `Dockerfil`, it will first look for a `docker-compose` file and attempt to build and run a container from it.

**When creating our Elastic Beanstalk environment in the next lecture, we need to select Docker running on `64bit Amazon Linux`** instead of Docker running on `64bit Amazon Linux 2`. This will ensure that our container is built using the Dockerfile and not the compose file.

#### AWS Elastic Beanstalk

Great for single container applications

After we create a **Elastic Beanstalk** application, we want to create an `environment`

Choose web

Select **Platform** as **Docker**, then **Create Environment**

<a href="https://ibb.co/hYFgwgX"><img src="https://i.ibb.co/9ZHpSpw/image.png" alt="image" border="0"></a>
- Load Balancer is **Elactic Beanstalk's**