# Docker Security

```bash
# When building an image from Dockerfile
# 1. Use USER instruction to run the container as NON root user

# 2. IF the root user is needed inside the container
#    use the linux CAPABILITIES: NET_BIND, MAC_ADMIN, NET_RAW, CHOWN...
#    See all CAPABILITIES at /usr/include/linux/capability.h

# 3. Use the following to ADD the wanted capabilities
#       RUN setcap <CAPABILITY-NAME> <APP-NAME>

# 4. Use the following to DROP the wanted capabilities
#       RUN setcap -r <CAPABILITY-NAME> <APP-NAME>

# 5. In order use ALL capabilities as the HOST root user 
#       docker run --privileged <image-name>

FROM ubuntu:latest

RUN apt-get update && apt-get install -y iputils-ping

RUN setcap cap_net_raw+ep /bin/ping

CMD ["ping", "localhost"]


```

## CGROUPS - Control Groups 
### CPU - Control Group
```bash
# By default each container running on the host gets equal amount of cpu time
# and IF NO restriction applied the container WILL BE able to consume as much CPU as it wants
# Restrictions:
# "--cpu-shares" - set the cpu share in comparison to other containers
# "--cpuset-cpus=0-1" - set which core the container will be running on
# "--cpus" - set a value between 0-4 (if there are 4 cores) in order to LIMIT how much cpu a container can consume out of 4 cores
#            for instance "--cpus=2.5" means 2.5 out of 4 is 62.5% a HARD limit of using cpu

# Example:
# Here all 3 webapp will get equal share of the HOST CPU while webapp4 will get lower share of the cpu
docker container run --cpus=2.5  --cpuset-cpus=0-1 --cpu-shares=1024 webapp1
docker container run --cpus=3.5  --cpuset-cpus=0-1  --cpu-shares=1024 webapp2
docker container run --cpus=1.5  --cpuset-cpus=2    --cpu-shares=1024 webapp3
docker container run --cpus=1    --cpuset-cpus=2    --cpu-shares=512 webapp4
```


### MEM - Control Group
```bash
# By default each container running on the host unrestricted memory consumption
# - IF NO restriction applied the container WILL BE able to consume as much memory as it wants
# - IF restriction is applied and the container tries to consume MORE it WILL be KILLED with the exception OOM(Out Of Memory)
# 
# Restrictions:
# "--memory-reservation" - set the MINIMUM memory consumption limit 
# "--memory" - set the memory consumption limit 
#                 b - for bytes
#                 k - for Kilo bytes (1024 bytes)
#                 m - for Mega bytes (1024 kilo)
#                 g - for Giga bytes (1024 mega)
# "--memory-swap" - set the SWAP memory consumption limit 
#                   if --memory-swap=-1 then the container may consume unlimited SWAP memory

# Example:
# The following sets 1GB of memory which can be consume by the container
# 512MB from the RAM
# 512MB from the SWAP (if SWAP is enabled on the host)
docker container run --memory=512m webapp

# The following sets ONLY 512MB of memory which can be consume by the container
# Because we specified the swap flag which calculates as 512 - 512 = 0
docker container run --memory=512m --memory-swap=512m webapp

# The following sets 512MB of RAM and 256MB of SWAP memory which can be consume by the container
# Because we specified the swap flag which calculates as 768 - 512 = 256
docker container run --memory=512m --memory-swap=768m webapp

```

### Docker Networking
```bash
# With Docker installation we have 3 networks

# - bridge (default network)
#   Private internal network created by docker usually 172.17.0.0/16
# "Config": [
#   {
#       "Subnet": "172.17.0.0/16",
#       "Gateway": "172.17.0.1"
#   }

docker run ubuntu

# - none
#   The container will run in an isolated network and not accessible
docker run Ubuntu --network=none ubuntu

# - host
#   The container will be attached and accessible 
#   on the host at the same port we set in the Dockerfile
docker run Ubuntu --network=host ubuntu 

# In order to isolate CIDRs network for containers
# we can create NEW internal network with docker
docker network create --driver=bridge --subnet=182.18.0.0/16 custom-isolated-network

# Embedded DNS
# The name of the container is the DNS name each container may resolve to turn each other
# Docker installation has its won DNS server at 127.0.0.11 (Internal network)
# So if we have 2 containers 
#   - Web container called web
#   - DB  container called mysqlDB
# We could code in the web container something like
# ... 
# mysql.connect(mysqlDB)
# ...


# List all networks
docker network ls

# Inspect network
docker network inspect [network-ID]


```