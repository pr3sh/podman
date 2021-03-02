### Abstract

This covers the creation of custom images within podman, based on Dockerfiles


A `Dockerfile` is a mechanism to automate the building of container images.
Building an image from a `Dockerfile` is a three-step process.
	1. Create a working directory
	2. Write a the `Dockerfile`
	3. Build the image with *Podman*


- This is an example Dockerfile for building a simple Apache web server container:
```bash
#This is a comment line
FROM ubi7/ubi:7.7

LABEL description="This is a custom httpd container image"

MAINTAINER John Doe <name@xyz.com>

RUN yum install -y httpd

EXPOSE 80

ENV Loglevel "info"
ADD http://someserver.com/filename.pdf /var/www/html 
COPY ./src/ /var/www/html 
USER apache 
ENTRYPOINT ["/usr/sbin/httpd"]
CMD ["-D","FOREGROUND"]
```
```
- 	`FROM` delcares the new container image extends
- 	`LABEL` responsible for adding generic metadata. A label is a simple key-value pair.
- 	`MAINTAINER` indicates the Author field of the generated container image\'s metadata.
-	`RUN` executes commands in a new layer on top of the current image.The shell used to execute command is /bin/bash
-	`EXPOSE` indicates that the container listens on a specified network port at runtime.
-	 `ENTRYPOINT` defines both the command to be enxecuted and the parameters

