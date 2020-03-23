### Overview
Docker relies on several UNIX technologies:

1. `namespaces` to provide isolated workspace, called container:
    - `pid` - process isolation
    - `net` - managing network interfaces
    - `ipc` - managing inter-process communication
    - `mnt` - managing filesystem mount points
    - `uts` - isolating kernel and version identifiers
2. `cgroups` (Control groups) to limit hardware resorces for application
3. Union File Systems - file systems that operate by creating layers, making them lightweight and fast.


Docker Engine
```
        Docker CLI
        Docker REST API
        Docker Deamon
images, containers, networks, volumes
```

Example scenario:
- write code locally and share work using Docker containers
- use Docker to push applications into a test environment and execute automated and manual tests
- fix bugs in the development environment and redeploy them to the test environment for testing and validation
- push image to the production environment

<img src="./architecture.svg">

### Docker registries

Docker Hub is a public registry that anyone can use, and Docker is configured to look for images on Docker Hub by default.
Commands: `docker pull, docker run, docker push`

### Docker objects

#### Images

> read-only template with instructions for creating a Docker container

Best practices:
- start with an appropriate base image
- use multi-stage builds
- maybe create own base image if there's multiple similar images
- maybe use `production` image as a base image for debugging
- building, testing, tagging and pushing to DockerHub should be done automatically, using CI/CD pipeline. 
- add meaningful tags to images, instead of default `latest` (`webapp-prod:v0.0.1`)

#### Containers

> running instance of an image + configuration options (set on creation or start)

### Networks:

#### bridge

> containers that need to communicate on the **same** Docker daemon host.

**Default Bridge network** named "bridge" is used by default:
```shell
> docker network ls
bridge
... 
```

**Custom Bridge network** can be created:
```shell
> docker network create --driver bridge my_net
```
On user-defined networks like alpine-net, containers can not only communicate by IP address, but can also resolve a container name to an IP address. This capability is called automatic service discovery.

Docker Compose creates separate Bridge network for each application. 

#### host

> network stack should not be isolated from the Docker host, but other aspects of the container should be isolated.

Container doesn't have it's own IP address - uses host's.

#### overlay

> communication between containers, running on different Docker hosts
> multiple applications work together using swarm services

#### macvlan

Used to assign a MAC address to a container, making it appear as a physical host on your network. The Docker daemon routes traffic to containers by their MAC addresses.

> legacy applications
> applications which monitor network traffic.

