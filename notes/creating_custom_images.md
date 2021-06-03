### Abstract

This covers the creation of custom images within podman, based on **`Dockerfiles`**

-  **`Table of contents`:**
	- [Building Base Containers](#building-base-containers)
	- [Building Images with Podman](#building-images-with-podman)
	- [Full Example](#full-example)	
#### **`Building Base Containers: `**
A **`Dockerfile`** is a mechanism to automate the building of container images.
Building an image from a `Dockerfile` is a three-step process.
	1. Create a working directory
	2. Write a the `Dockerfile`
	3. Build the image with *Podman*

- This is an example **`Dockerfile`** for building a simple Apache web server container:
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
- **`FROM`** delcares the new container image extends
- **`LABEL`** responsible for adding generic metadata. A label is a simple key-value pair.
- **`MAINTAINER`** indicates the Author field of the generated container image\'s metadata.
- **`RUN`** executes commands in a new layer on top of the current image.The shell used to execute command is /bin/bash
- **`EXPOSE`** indicates that the container listens on a specified network port at runtime.
- **`ENTRYPOINT`** defines both the command to be executed and the parameters.
- **`ENV`** is responsible for defining environment variables.
- **`ADD`** instruction copies files or folders from local or remote source and adds them to the container file system.
- **`COPY`** copies files from the working directory and adds them to the contianers file system.
- **`CMD`** provides the defult arguments for the `ENTRYPOINT` instruction.
- **`USER`** specifies the username or the `UID` to use when running the contianer image for the `RUN`,`CMD`,& `ENTRYPOINT` instructions.

*It is good to define a different user other than root for security reasons.*

#### `CMD` and `ENTRYPOINT`
There are **two** formats for these commands.
> *Exec form*, which uses a `JSON` array:
```bash
ENTRYPOINT ["command","param1","param2"]
CMD ["param1","param2"]
```
> *Shell form*:
```bash
ENTRYPOINT "command","param1","param2"
CMD "param1","param2"

# Exec form is the preffered form
# Docker file should contain at most one ENTRYPOINT and one CMD instruction.
# if more than one of each is present, then only last instruction takes effect.
```
> Example usage of **`ENTRYPOINT`** and **`CMD: `**

```bash
ENTRYPOINT ["/bin/date"]
CMD ["+%H:%M"]
```
```zsh
# run the image that was build with these params
$ sudo podman run it <image_name> 
>> 10:50 am
#modify command
$ sudo podman run it <image_name>  +%A
>> Tuesday
#define both entry point and command to be executed
ENTRYPOINT ["/bin/date","+%H:%M"]
```
### **`ADD`** and **`COPY`**
*Different forms:*
```bash
#Shell
ADD  <source> ...<destination>
COPY <source> .... <destination>

#Exec form
ADD  ["<source>" ....."<destination>"]
COPY ["<source>" .... "<destination>"]


#You can also specify a URL resource
ADD http:domain.com/file.pdf /var/www/html 
```
> *If the source is a file system path, it must be inside the working directory.*
> *If the file is compressed, then add the decompression command to the destination directory.*

*Example usage for best practices when running successsive commands.*

```bash
 RUN yum --diasblerepo=* --enablerepo="rhel-7-server-rpms" && yum update -y \
 		&& yum install -y httpd
```
> **Red Hat** recommends applying similar formatting rules to other instructions accepting multiple parameters, such as **`LABEL`** and **`ENV`** as shown below:
```bash
LABEL version="2.0" \
      description="This is an example container image" \
      creationDate="01-09-2017"

ENV MYSQL_ROOT_PASSWORD="my_password" \
    MYSQL_DATABASE "my_database"
```
#### **`Building Images with Podman: `**
> The **`podman build`** command processes the **`Dockerfile`** and builds a new image based on the instructions it contains, using the syntax below.

**`sudo podman build -t NAME:TAG DIR`**

- **`DIR`** is the path to the working directory, which includes the **`Dockerfile`**. 
- **`NAME:TAG`** is a name with a tag given to the new image. 
- If **`TAG`** is not specified, then the image is automatically tagged as **`latest`**.

#### **`Full Example: `**
```bash

$ mkdir /docker-practice/Dockerfile
$ vim  /docker-practice/Dockerfile  
```
- Once you have opened the **`Dockerfile`** you can write your commmands to be executed.

```bash
#Dockerfile 
FROM ubi7/ubi:7.4

MAINTAINER your name <your_email>

ENV PORT 8188

RUN yum install -y httpd && yum clean all

#replace Listen 80 in httpd.conf file to our specified port
RUN sed -ri ie "/^/Listen 80/c\Listen ${PORT}" /etc/httpd/conf/httpd.conf && \
	chown -R apache:apache /etc/httpd/logs/ &&  \
	chwon -R apache:apache /run/httpd/

USER apache
EXPOSE ${PORT}

#Copy files in the src folder to Apache doc root
COPY ./src/ /var/www/html/

#start apache in the foreground
CMD ["httpd","-D","FOREGROUND"]
```
> After you close and save file, build image based on **`Dockerfile`** .
```zsh
$ sudo podman build --layers=false \
	-t <image_name> <directory_of_docker_file>
```
> Verify custom image was built successfully.
```zsh
$ sudo podman images
>> REPOSITORY  			      TAG					IMAGE ID
localhost/<image_name>		latest       		dhsh459dk
```
- You can then **`run`** containers based on these images or **`commit`** them to a registry.

Done!





