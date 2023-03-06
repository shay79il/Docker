# Docker Networking

```bash
# With Docker installation we have 3 networks

# 1. Bridge (default network)
###########################
#   Private internal network created by docker usually 172.17.0.0/16
# "Config": [
#   {
#       "Subnet": "172.17.0.0/16",
#       "Gateway": "172.17.0.1"
#   }
docker run ubuntu


# 2. None
#######
# The container will run in an isolated network and not accessible
docker run Ubuntu --network=none ubuntu


# 3. Host
#######
#   The container will be attached and accessible 
#   on the host at the same port we set in the Dockerfile
docker run Ubuntu --network=host ubuntu 
```

## Custom network
```bash
# In order to isolate CIDRs network for containers
# we can create NEW internal network with docker
docker network create --driver=bridge --subnet=182.18.0.0/16 custom-isolated-network

# Remove network
docker network rm [network-ID]

# Remove all unused networks
docker network prune

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

# Set the CIDR network to a container
docker network connect [network-ID] [container-ID] 

# Unset the CIDR network to a container
docker network disconnect [network-ID] [container-ID] 


```