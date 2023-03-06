# Docker Image registry

## There are a few out there

- Docker Trusted registry
- Google Container registry
- Amazon Container registry
- Azure Container registry
- [Docker Hub](https://hub.docker.com)

## In Docker Hub there are 3 types of images

- Official Images
- Verified Images
- User Images

## Images list and pull

```bash
# List images on the host
docker image ls

# Get the total images size
docker system df

# Search for an image
docker search httpd --limit 2

# Pull an image
# docker pull -Registry- -User- -Image-
# docker pull docker.io/httpd/httpd
# docker pull gcr.io/httpd/httpd
#   When NOT specify the default are
#     - Registry docker.io
#     - User httpd
#     - Image httpd
docker pull httpd


```

## Docker PUSH

```bash
# We have to authenticate first
# We have to tag the image to be pushed

# For all the following we MUST be authenticated
docker pull gcr.io/organization/ubuntu

# That why we have to LOGIN
docker login <Docker Registry>

# Create a SOFT link to an existing image
# docker image tag oldTag newTag
docker image tag httpd:alpine shay79il/httpd:v1

# Push an image
docker push httpd shay79il/httpd:v1
```

## Docker remove images

```bash
# List all images
docker image list

# Remove all UNUSED images
docker image prune -a
```

## Docker images layers and configuration

```bash

# See all layers which have been used to create the image
docker image history <imageNmae>

# Inspect an image will give us the following
# - Parent/Base Image
# - Exposed ports
# - Author
# - Size
# - All configs in Dockerfile
# docker image inspect <imageNmae>
# docker image inspect <imageNmae> -f '{{ <JSON-PATH> }}'
docker image inspect ubuntu -f '{{ .Os }}'

```

## Docker save and load images
### Lets say we have a restricted ENV with no internet connection to download images 
```bash

# Pull needed images to a NON-RESTRICTED Host
docker pull ubuntu

# Save the image as tar file
docker image save ubuntu -o ubuntu.tar

# Move the TAR file to the RESTRICTED Host
scp /path/to/RESTRICTED-Host/ubuntu.tar /path/to/NON-RESTRICTED-Host/  

docker image load -i ubuntu.tar
```

## Docker containers exported to images
```bash

# Create image TAR from a running container
docker export [containerID] > container.tar

docker image import container.tar newImage:latest

```

## Docker commit
### Create new container after customize
```bash

docker run -d --name=httpd httpd

# Customize the container
docker exec -it httpd bash
echo "Welcome all" > htdocs/index.html
  
# Stop the container
docker container stop httpd

# create a new IMAGE from the customized container
docker container commit httpd custom-httpd

docker image ls

```



