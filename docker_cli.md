### Overview

#### run command

Example: 
```shell
> docker run -i -t ubuntu /bin/bash
```

- pull ubuntu image from registry (as in `docker pull ubuntu`)
- create container (as in `docker container create`)
- create filesystem for container
- create network interface to connect container to default network(including assignment of an IP address to container). Container will be able to connect to external networks using host machine's network connection.
- start container, execute `/bin/bash`

`-i -t` options mean container runs interactively (`terminal input -> container`, `container output -> terminal`)
To stop running container: `exit`
