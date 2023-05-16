# Dockerfile

## Example
```dockerfile

# Base layer
FROM centos:7

# Env variable inside the container
# can be defined WITHOUT default value
# ARG TOMCAT_VERSION and the value passed 
# by the docker image build command
ARG TOMCAT_VERSION=8.5.6

# Install dependencies
RUN  yum install -y epel-release java-1.8.0-openjdk.x86_64 wget

# Add new user & group 
RUN groupadd tomcat && mkdir /opt/tomcat
RUN useradd -s /bin/nologin -g tomcat -d /opt/tomcat tomcat

# Set the working directory
WORKDIR /

# Download package depending on the Env variable we passed earlier
RUN wget https://archive.apache.org/dist/tomcat/tomcat-8/v$TOMCAT_VERSION/bin/apache-tomcat-$TOMCAT_VERSION.tar.gz
RUN tar -zxvf apache-tomcat-$TOMCAT_VERSION.tar.gz -C /opt/tomcat --strip-components=1

# Set access control th the UNTARed package
RUN cd /opt/tomcat && chgrp -R tomcat conf
RUN chmod g+rwx /opt/tomcat/conf && chmod g+r /opt/tomcat/conf/*
RUN chown -R tomcat /opt/tomcat/logs/ /opt/tomcat/temp /opt/tomcat/webapps /opt/tomcat/work
RUN chgrp -R tomcat /opt/tomcat/bin && chgrp -R tomcat /opt/tomcat/lib && chmod g+rwx /opt/tomcat/bin && chmod g+r /opt/tomcat/bin/*

# Change again the working directory
WORKDIR /opt/tomcat/webapps

# Download JAVA app 
RUN wget https://tomcat.apache.org/tomcat-7.0-doc/appdev/sample/sample.war

EXPOSE 8080

CMD ["/opt/tomcat/bin/catalina.sh","run"]
```

## Building a custom image
```bash
# version 1
docker image build -t shay79il/tomcat --build-arg TOMCAT_VERSION .

# version 2
# Use the flag --build-arg to pass a new value to the container
docker image build -t shay79il/tomcat:v2 --build-arg TOMCAT_VERSION=8.5.8 .

docker login -u shay79il -p
docker push shay79il/tomcat
docker push shay79il/tomcat:v2

docker image history shay79il/tomcat
```

# Best practices


## Build Context
```bash

# 1. Add .dockerignore file to ignore file/directories
# 2. Watch for build context
#     CONTEXT - where the app files resides
#     CONTEXT = .
#     CONTEXT = /opt/myapp
#     CONTEXT = https://github.com/myaccount/myapp
#     CONTEXT = https://github.com/myaccount/myapp#<branch>
#     CONTEXT = https://github.com/myaccount/myapp:<folder>
# 3. By default docker build command search for Dockerfile file
#    If not present we should use the flag -f with <filename> to run the instructions (FROM,RUN ste...)
docker image build <CONTEXT>
```

## Cache Busting
```bash

# 1. Use Cache Busting --> '&&' for installing packages and apt update 
# 2. Use version pinning --> ( python3-pip=20.0.2 )
# 3. Use COPY instead of ADD instruction
# 4. Use "--no-cache" flag - In order to rerun ALL instructions 
# 5. Use "--no-cache" flag - Intermediate containers created removed automatically

# Bash command
$ docker build --force-rm --no-cache -t <image-name>:<tag> <path-to-dockerfile>

#################################
# Dockerfile
FROM ubuntu

RUN apt update && apt install -y  \
    python                        \
    python-dev                    \
    python3-pip=20.0.2                   

RUN pip3 install flask flask-mysql

COPY app.py /opt/source-code

ENTRYPOINT flask run
```


## COPY vs ADD
```bash
# COPY - Just copy files or directories
# ADD - In addition to copy it cat extract zipped files into a directory in the image 
#       The zipped files can be from the internet or local on the host

# 1. Always prefer COPY over ADD
```

## CMD vs ENTRYPOINT
```bash
# 
# ENTRYPOINT - The actual CMD to run 
# Example:
# ENTRYPOINT ["sleep"]
# 
# CMD - Can include the actual command but used for optional DEFAULT arguments to pass in order to run the command
#       - All arguments MUST be separated
#       - The first argument is the EXECUTABLE
# Example:
# CMD ["sleep", "5"]
# 
# Mixed example:
# ENTRYPOINT ["sleep"]
# CMD ["5"]
```

## Base vs Parent Image
```bash

# Parent Image - FROM is other than scratch
FROM httpd
COPY index.html htdocs/index.html

# Base Image - FROM is scratch
FROM scratch
ADD rootfs.tar.xz /
CMD ["bash"]
```

## Multi-Stage Builds

### The OLD way - Separated stages to build image
```bash
Before Multi-Stage Builds we had the following
# 1. Build app - Dockerfile.builder with 
# --->>> docker image build -f Dockerfile.builder -t builder .
FROM node
COPY . .
RUN npm install
RUN npm run build



# 2. Extract the app from the build stage
docker container create --name builder builder 
docker container cp builder:dist ./dist
docker container rm -f builder

# 3. Containerize - Dockerfile
# --->>> docker image build -t myapp .
FROM nginx
COPY dist /usr/share/nginx/html
CMD [ "nginx", "-g", "daemon off;"]
```


### The NEW way - Multi-stage build image
```bash

FROM node AS builder
COPY . .
RUN npm install
RUN npm run build

FROM nginx
COPY --from=builder dist /usr/share/nginx/html
CMD [ "nginx", "-g", "daemon off;"]

# Just run
docker image build -t myapp .

# In order to build specific stage run the following
docker image build --target builder -t myapp .
```


