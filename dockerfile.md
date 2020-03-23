### Docker build
The `docker build` command builds an image from a:
- Dockerfile
- context (set of files, specified by path usually `.`)

To use a file in the build context, the Dockerfile refers to the file specified in an instruction, for example, a `COPY` instruction.

`docker build .`

Specify a tag:
`docker build -t repo_name/app_name:some_tag .`
or tags:
`docker build -t repo_name/app_name:tag_1 -t repo_name/app_name:tag_2 .`

### Dockerfile best practices [more](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#dockerfile-instructions)
- use `.dockerignore` to reduce build context size, sent to Docker deamon.
`Sending build context to Docker daemon  187.8MB`
- avoid unnecessary packages
- decouple applications into separate containers
- minimize number of layers (created with `RUN, COPY, ADD`)
- use multi-stage builds to decrease size of final image
- sort multiline arguments
```docker
RUN apt-get update && apt-get install -y \
  bzr \
  cvs \
  git \
  mercurial \
  subversion
```
- leverage build cache
    - for `ADD` and `COPY` files are matched to validate cache
    - for `RUN` only command is matched to validate cache
- use BuildKit for executing builds instead of default 
`DOCKER_BUILDKIT=1 docker build .` (it's still experimental)

> Once cache is invalidated all subsequent commands generate new images, and cache is not used


### Dockerfile instructions [more](https://docs.docker.com/engine/reference/builder/)

#### Parser directives

Change escape character to allow using \ in Dockerfile
```docker
# escape=`
# escape=\
```

#### Environment variable replacement
```docker
WORKDIR ${FOO}
```

#### FROM
```docker
FROM [--platform=<platform>] <image>[:<tag>] [AS <name>]
```

```docker
FROM <image>
FROM <image>:<tag>
```

#### RUN
```docker
RUN <command>
RUN ["executable", "param1", "param2"] (if executable is not in $path)
```
Use `\` for multiline
Build cache for `RUN` instructions can be invalidated by `docker build --no-cache`

#### CMD
Provide defaults for long-running container. Only one `CMD` can exist in Dockerfile.

Can also pe passed to `docker run`:
```shell
docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]
```

```docker
# Preferred form
CMD ["executable","param1","param2"] # executable param1 param2

# ENTRYPOINT FORM
CMD ["param1","param2"] # defaults for ENTRYPOINT
```

#### ENTRYPOINT
Configure container, that will run as executable.

```docker
ENTRYPOINT ["executable", "param1", "param2"]

# -d will be passed as an argument to "executable"
docker run <image> -d
```

#### LABEL
Metadata for image
```docker
LABEL version="1.0"
LABEL description="This text illustrates \
that label-values can span multiple lines."
```

#### EXPOSE

Informs Docker that the container listens on the specified network ports at runtime. It acts as kind of documentation.

To "really" publish port, use
`docker run -p host_port_number:exposed_port_number` or
`docker run -P` - publish all EXPOSEd ports

```docker
EXPOSE <port> [<port>/<protocol>...] # protocol = tcp | udp
```

#### ENV

```docker
ENV <key> <value>
ENV <key>=<value> ...

# Better not to use it. To set env on individual command, do instead:
RUN <key>=<value> <command>

Can also pe passed to `docker run`:
```shell
docker run -e "deep=purple" -e today --rm alpine
```

#### ADD (rarely used)
Copy/download/extract tar from `(build context)/src` to `(current working directory)/dest`.

```docker
ADD [--chown=<user>:<group>] <src>... <dest>
ADD [--chown=<user>:<group>] ["<src>",... "<dest>"]

ADD --chown=bin files* /somedir/
ADD files* /somedir/
```

#### COPY (often used)
Copy from `(build context)/src` to `(current working directory)/dest`.

```docker
COPY [--chown=<user>:<group>] <src>... <dest>

# src  [file or a folder in a build context | URL]
# dest [file or a folder in a build context]

# add all files, starting with "hom"
COPY hom* /mydir/  
```

#### VOLUME

#### WORKDIR
Working directory for `RUN`, `CMD`, `ENTRYPOINT`, `COPY` and `ADD`.
Can be absolute, or relative.

```docker
WORKDIR /path/to/workdir

WORKDIR /a # absolute
WORKDIR b # relative
WORKDIR c # relative
RUN pwd # /a/b/c
```

#### ARG
Build-time arguments to pass to `docker build`, so it fails if given arguments not specified in `ARG`.
Invalidates build cache if different from previous build.
Predefined `ARG`s: TARGETOS, BUILDOS...

```docker
FROM busybox
ARG user1
ARG CONT_IMG_VER
...

# or default values
FROM busybox
ARG user1=someuser
ARG CONT_IMG_VER=1
...

docker build --build-arg CONT_IMG_VER=v2.0.1 .
```
