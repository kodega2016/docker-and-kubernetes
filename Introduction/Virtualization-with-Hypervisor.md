# Virtualization with Hypervisor

<!--toc:start-->

- [Virtualization with Hypervisor](#virtualization-with-hypervisor)
  - [Limitations of Virtualization](#limitations-of-virtualization)
  - [What is Container?](#what-is-container)
    - [Docker Engine Components](#docker-engine-components)
  - [Docker Containers](#docker-containers)
  - [Docker Container Restart Policies](#docker-container-restart-policies)
  - [Wordpress and MySQL Deployment on Docker Engine](#wordpress-and-mysql-deployment-on-docker-engine)
  - [Docker Images](#docker-images)
  - [Docker Engine Storage](#docker-engine-storage)
    - [Bind Mounts](#bind-mounts)
  - [Docker Networking](#docker-networking)
  - [Docker User Defined Bridge Network](#docker-user-defined-bridge-network)
  <!--toc:end-->

Hypervisor is a piece of software that creates and run virtual servers on the top
of the physical server.There are two types of hypervisors:

- Type 1 hypervisor(Hardware hypervisor): It runs directly on the hardware of
  the host machine. Examples: VMware ESXi, Microsoft Hyper-V, Xen, KVM.

- Type 2 hypervisor(Hosted hypervisor): It runs on top of the host operating system.
  Examples: VMware Workstation, Oracle VirtualBox, Parallels Desktop.

## Limitations of Virtualization

- Provisioning time: It takes
- Resource allocations is not dynamic: Resources allocated to a virtual machine
  are fixed and cannot be changed dynamically.
- Resource assigned are utilized by the OS like kernel,drivers, etc.: The resources
  assigned to a virtual machine are utilized by the operating system running on
  the virtual machine, which can lead to resource wastage.

## What is Container?

It is a simple another process on the host machine that has been isolated
from all other processes running on the host machine. It is a lightweight
process that shares the host machine's kernel and resources, but has its own
process space, file system, and network stack.

Containers will have only the libraries and dependencies that are required
for the application to run, which makes them lightweight and fast to start.
There is no any kernel or OS running inside the container.

### Docker Engine Components

- Docker Daemon
- Docker Client
- Docker Registry
- Docker Objects
  - Images
  - Containers
  - Volumes
  - Networks

## Docker Containers

We can run multiple containers on a single host machine, and each container
can run a different application or service. Containers are isolated from the
host machine and from each other, which means that they can run different.

We can run the docker container with the command:

```bash
docker container run -d --name webserver -p 80:80 nginx
```

To view the logs of the docker container, we can use the command:

```bash
docker container logs webserver
```

We can stop and restart the docker container with the command:

```bash
docker container stop webserver
docker container start webserver
```

The command to view the docker container details is:

```bash
docker container inspect webserver
```

Here,we can also get the IP address of the container.

To pass the environment variables to the docker container, we can use the
`-e` flag:

```bash
docker container run -d --name db-server \
    --hostname db-server \
    -e MYSQL_ROOT_PASSWORD=secret \
    -e MYSQL_ROOT_HOST='%' \
    -p 3306:3306 mysql
```

Here,p flag is used to map the port of the host machine to the port of the
the container. The `-e` flag is used to pass the environment variables.

## Docker Container Restart Policies

There are several restart policies that can be used with Docker containers:

- no(default): The container will not be restarted.
- always: The container will be restarted regardless of the exit status.
- unless-stopped: The container will be restarted unless it is stopped manually.
- on-failure: The container will be restarted only if it exits with a non-zero
  exit status.

```bash
docker container run -d --name webserver \
    --restart always \
    -p 80:80 nginx
```

We can also update the container restart policy with the command:

```bash
# Update the restart policy of an existing container
docker container update --restart always webserver
# on-failure with max retries
docker container update --restart on-failure:3 webserver
```

## Wordpress and MySQL Deployment on Docker Engine

We can deploy a WordPress and MySQL application on Docker Engine using the
containers.

Run MySQL container:

```bash
docker container run -d --name mysql-server \
  -e MYSQL_ROOT_PASSWORD=supersecret \
  -e MYSQL_DATABASE=wordpress \
  -e MYSQL_ROOT_HOST='%' \
  mysql:latest
```

After that, we can run the WordPress container:

```bash
docker container run -d --name wordpress \
  -p 8080:80 \
  -e WORDPRESS_DB_HOST=<ip-of-db-container> \
  -e WORDPRESS_DB_USER=root \
  -e WORDPRESS_DB_PASSWORD=supersecret \
  -e WORDPRESS_DB_NAME=wordpress \
  wordpress:latest
```

## Docker Images

We can pull the images from the Docker Hub and run the containers from
the images. We can also build our own images using the Dockerfile.

We can list all the images on the host machine with the command:

```bash
docker image ls
```

To view the details of the image, we can use the command:

```bash
docker image inspect <image-name>
```

To view the layers of the image, we can use the command:

```bash
docker image history <image-name>
docker image history nginx
```

## Docker Engine Storage

There are three types of storage drivers in Docker Engine:

- Volumes: Persistent storage that is managed by Docker and can be shared between
  containers.
- Bind mounts: Allows you to mount a directory from the host machine into a container.
- tmpfs mounts: Temporary storage that is stored in memory and is not persisted
  to disk.

We can run the following command to create a volume:

```bash
docker volume create <volume-name>
docker volume create webdata
```

After that, we can run the container with the volume:

```bash
docker container run -d --name webserver \
  -p 8080:80 \
  -v webdata:/usr/share/nginx/html \
  nginx:latest
```

We can inspect the docker container to view the volume details:

```bash
docker container inspect webserver
```

We also use another type of storage called bind mounts, which allows us to
do same thins as volume.

```bash
docker container run -d --name webserver \
  -p 8080:80 \
  --mount type=volume,source=webdata,target=/usr/share/nginx/html \
  nginx:latest
```

### Bind Mounts

We can mount a directory from the host machine into a container using bind mounts.
For example,we can mount './website-data' directory from the host machine to
nginx container:

```bash
docker container run -d --name webserver \
-p 8080:80 \
-v ./website-data:/usr/share/nginx/html \
nginx:latest
```

Same operation can be done using the `--mount` flag:

```bash
docker container run -d --name webserver \
-p 8080:80 \
-- mount type=bind,source=$(pwd)/website-data,target=/usr/share/nginx/html \
nginx:latest
```

## Docker Networking

Docker provides several networking options to connect containers to each other,there
are are three main types of networks:

By default,there are three main types of networks in Docker:

- Bridge Network: The default network type that allows containers to communicate
  with each other on the same host machine.

- Host Network: Allows containers to share the host machine's network stack,
  which means that they can access the host machine's network interfaces directly.

- Overlay Network: Allows containers to communicate with each other across
  multiple host machines, which is useful for deploying applications in a distributed
  environment.(This is used in Docker Swarm mode)

## Docker User Defined Bridge Network

We can create a user-defined bridge network to connect containers to each other.

To create a user-defined bridge network, we can use the following command:

```bash
docker network create --subnet 10.10.0.0/16 --gateway 10.10.0.1 database-network;
```

We also connect the container to the user-defined bridge network using the
`--network` flag:

```bash
docker container connect <database-network> <container-name>
docker container connect database-network mysql-server
```

We can disconnect the container from the user-defined bridge network using
the `docker network disconnect` command:

```bash
docker network disconnect database-network mysql-server
```

The two containers connected to the same user-defined bridge network can communicate
each other using the container name as the hostname. For example, if we have
webserver and mysql-server containers connected to the same user-defined network,

The webserver container can connect to the mysql-server container using the
hostname `mysql-server`:
