# [Install Docker Engin](https://docs.docker.com/engine/install/ubuntu/)

```bash
# After installation complete we can see docker on our host at
ls /var/lib/docker
```

# Use Docker CLI remotely

## If We want to use `dockerd` without docker service

```bash
export DOCKER_HOST="tcp://192.168.1.10:2376"

# create TLS certs
openssl genrsa -out server.key 2048
openssl req -new -nodes -key server.key -out server.csr
echo "subjectAltName=DNS:[Your DNS name]" >> extfile.cnf
openssl x509 -req -in server.csr -CA ~/.minikube/ca.crt -CAkey ~/.minikube/ca.key -CAcreateserial -out server.crt -extfile extfile.cnf -days 365


dockerd --debug                             \
        --host=tcp://192.168.1.10:2376      \
        --tls=true                          \
        --tlscert=/var/docker/server.pem    \
        --tlskey=/var/docker/serverkey.pem


# An alternative way is to create `/etc/docker/daemon.json` file
{
  "debug": true,
  "host":[tcp://192.168.1.10:2376],
  "tls": true,
  "tlscert": "/var/docker/server.pem",
  "tlskey": "/var/docker/serverkey.pem"
}

# and just run
dockerd

```

# Use Docker Basics

## Creating and Running containers
```bash
# Build docker image
docker image build .

# Create new docker container WITHOUT running it
docker container create ubuntu

# Start new docker container
docker container start <httpd-conatiner-ID>

# Create and RUN new docker container (two commands combined)
# -it         - used flag for attaching to our terminal and in interactive mode
# -rm         - used flag for removing the container after it is stopped
# --name      - used flag for setting the name of the container
# --hostname  - used flag for setting the hostname of the container
# --restart   - used flag for setting the restart policy of the container
#               with the following options - [no, on-failure, always, unless-stopped]
#               default is NO
docker container run -it --rm --name=ubuntu-server --hostname=ubuntu --restart=ALWAYS ubuntu

# Create and RUN new docker container DETACHED (run in the background)
docker container run -d --name=web-server httpd

# Run the DETACHED in the foreground
docker container attach  <container-name/id>

# Rename container NAME
docker container rename ubuntu-server ubuntu-sv

# Kill and remove running docker container
docker container kill <container-name/id>

# Kill and remove ALL docker containers
docker container rm $(docker container ls -aq)
```

## LISTING
```bash
# List all running containers
docker container ls

# List all LATEST containers
docker container ls -l

# List short ids of running containers
docker container ls -q

# List short ids of running and STOPPED containers
docker container ls -aq

# List all running and STOPPED containers
docker container ls -a
```

## Executing commands
```bash
# Run command inside the container and output it to the terminal
docker container exec <containerID\Name> <Command>
docker container exec ubuntu-sv hostname

# Run command inside the container run it in the foreground
# Equivalent to
# docker container attach  <container-name/id>
docker container exec -it <containerID\Name> <Command>
docker container exec -it ubuntu-sv /bin/bash
```

## Inspecting commands
```bash
# Get all the metadata of a container
docker container inspect ubuntu-sv

# Get all the resource consumption of a container
docker container stats ubuntu-sv

# Get all processes consumption of a container
docker container top ubuntu-sv

# Get all logs of a container (Use -f flag to get LIVE logs)
docker container logs [-f] ubuntu-sv

# From a SYSTEM perspective
docker system events --since 60m
```

## Container states commands
```bash
# Run
docker container run -it ubuntu-sv

# Pause
docker container pause ubuntu-sv

# Continue
docker container unpause ubuntu-sv

# Terminate with grace time
docker container stop ubuntu-sv

# Send SPECIFIC signals to a running container
docker container kill --signal=<SIGNUM> ubuntu-sv

# Remove STOPPED container
docker container rm ubuntu-sv

# Remove container by force
docker container rm -f ubuntu-sv

# Remove ALL containers
docker container stop $(docker container ls -q)

# Remove ALL containers
docker container rm $(docker container ls -qa)

# Remove ALL STOPPED containers
docker container prune
```

## Copy content in container 
```bash
# Path on the container MUST exists

# Copy to a container
docker container cp <path/on/host> <container-name>:<path/on/container>

# Copy from a container
docker container cp <container-name>:<path/on/container> <path/on/host>

```

## Container port mapping
```bash

# Mapping port 80 on the HOST to port 8080 which the container is listening
docker container run --rm --name=webapp --hostname=webapp -p 80:8080 kodekloud/simple-webapp

# Publishing all the ports which the container is listening to a random port on he host
# - When it was created with the Dockerfile with the `EXPOSE` keyword
# The random port on the host is in range of 32768 - 60999
# Look at -->> `cat /proc/sys/net/ipv4/ip_local_port_range`
docker container run --rm --name=webapp --hostname=webapp -P kodekloud/simple-webapp

# In order to inspect which ports are exposed by the container run the following
docker inspect <image-name>
docker inspect kodekloud/simple-webapp


```

## Container logs
```bash

# Get container log configuration by
docker container inspect <ContainerID> | grep -i -A5 logconfig
docker container inspect -f '{{.HostConfig.LogConfig.Type}}' <ContainerID> 

# List containers
docker container ls -a

# Search for the logs in /var/lib/docker/containers
cd /var/lib/docker/containers/<ContainerID>

# An option to configure log driver to the following drivers
# awslogs, fluentd, gcplogs, gelf, journald, json-file, local, logentries, splunk, syslog
# can be retrieved by running 
#     docker system info | grep -i logging -
# In case of awslogs Driver we can configure the region
# And we must set a credential file or use Vault to have permission to send logs to AWS
# Look also at https://docs.docker.com/config/containers/logging/configure/
vi /etc/docker/daemon.json
{
  "debug": true,
  "host":[tcp://192.168.1.10:2376],
  "tls": true,
  "tlscert": "/var/docker/server.pem",
  "tlskey": "/var/docker/serverkey.pem",
  "log-driver": "awslogs",
  "log-opt": {
    "awslogs-region": "us-east-1"
  }

}

```
