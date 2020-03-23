## Multi stage build

By splitting image building into stages, resulting image may have smaller size.
Same stage can be used as a `FROM` for multiple following stages.

Multi stage build = Dockerfile with multiple `FROM` + `COPY` directives.

```docker
FROM image_1
# create ./stage_1_artefact

FROM image_2
COPY ./stage_1_artefact ./stage_2_workdir
# Do something else with artefact
```

### Dockerfile example
```docker
# ---- Build Stage ----
FROM elixir:1.9.1 AS app_builder

# Set environment variables for building the application
ENV MIX_ENV=prod \
LANG=C.UTF-8

# Install hex and rebar
RUN mix local.hex --force && \
mix local.rebar --force

# Create the application build directory
RUN mkdir /app
WORKDIR /app

# Copy over all the necessary application files and directories
COPY config ./config
COPY lib ./lib
COPY priv ./priv
COPY mix.exs .
COPY mix.lock .

# Fetch the application dependencies and build the application
RUN mix deps.get
RUN mix deps.compile
RUN mix release
```

> TODO Since Docker relies on layers, which it caches and reuses later on, I'd advise to first copy the mix.exs and mix.lock, get the dependencies, and only then copy the actual code over. This trick would effectively cache your fetched dependencies and reuse the layer in the future. Assuming that your actual code changes much more often than your dependency configuration, this will speed your image builds by an order of magnitude.

```docker
# ---- Application Stage ----
FROM debian:stretch AS app

ENV LANG=C.UTF-8

# Install openssl
RUN apt-get update && apt-get install -y openssl

# Copy over the build artifact from the previous step and create a non root user
RUN useradd --create-home app
WORKDIR /home/app
COPY --from=app_builder /app/_build .
RUN chown -R app: ./prod
USER app

# Run the Phoenix app
CMD ["./prod/rel/docker_elixir_19_release/bin/docker_elixir_19_release", "start"]
```

```shell
docker build -t my-app .

docker images
my-app
...

docker run --publish 4000:4000 &&
    --env COOL_TEXT='ELIXIR ROCKS!!!!' &&
    --env SECRET_KEY_BASE=$(mix phx.gen.secret) &&
    --env APP_PORT=4000  &&
    my-app:latest
```

### Inspection of running container

1. Inspect from same node (note using --sname instead of --name)
```shell
# Run shell in the container (my_app is container name)
docker exec -it my_app bash

# Get the `sname` of the running elixir session
ps aux | grep elixir

# INFO: remsh = sname@id/name_of_container
# Run iex on same node
iex \
    --sname console \
    --cookie secret_cookie \
    --remsh c3f2473a1e@3160c633294f
```

2. Inspect from another node (security issues)

```shell
# Convenience bash script
function reiex() {
 if [ $1 = "staging" ]; then
  iex --name "$(whoami)@reiex" &&
    --cookie secret_cookie &&
    --remsh "node@staging.example.com"
 elif [ $1 = "prod" ]; then
  iex &&
    --name "$(whoami)@reiex" &&
    --cookie secret_cookie &&
    --remsh "node@prod.example.com"
 else
  echo "Environment not found!"
 fi
}

# Usage
reiex <environment name>
```

### Use previous stage as new stage
```docker
FROM alpine:latest as builder
RUN apk --no-cache add build-base

# Use builder stage, name it build1
FROM builder as build1
COPY source1.cpp source.cpp
RUN g++ -o /binary source.cpp

# Use builder stage, name it build2
FROM builder as build2
COPY source2.cpp source.cpp
RUN g++ -o /binary source.cpp
```

### Debugging multi-stage build
Build image up to `--target` stage:
```shell
docker build --target stage_name -t my_image_name:latest .
```
Examples of stages to debug:
- a debug stage with all debugging symbols or tools enabled, and a lean production stage
- a testing stage in which your app gets populated with test data, but building for production using a different stage which uses real data