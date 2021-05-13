
## **`Abstract`:**

-  **`Table of contents`:**
	- [Fetching Container Images](#fetching-container-images)
	- [Running Containers](#running-containers)
		- [Useful Podman CLI Options](#useful-podman-cli-options)
		- [Example](#example)
	- [Managing Containers](#managing-containers)	
	- [Creating Persistent Storage](#creating-persistent-storage)
	- [Accessing Containers](#accessing-containers)
		- [Mapping Network Ports](#mapping-network-ports)
	- [Managing Container Images](#managing-container-images)
		- [Useful Search CLI Options](#useful-search-cli-options)
		- [Registry Authentication](#registry-authentication)
		- [Pulling Images](#pulling-images)
	- [Manipulating Container Images](#manipulating-container-images)
		- [Modifying Images](#modifying-images)
		

#### **`Fetching Container Images:`**

search for an image
```zsh
$ sudo podman search rhel                	
$ sudo podman pull <image_name> #Pull an image                    	
$ sudo podman images #List current images
```
#### **`Running Containers`:**

To run containers, you invoke the **`sudo podman run`** command.
```zsh
$ sudo podman run  rhel7:7.5 echo "Hello world" 	# run the rhel image and echo "Hello world"
```
#### **`Useful Podman CLI Options:`**

|         **Options**             |     **Meaning**                            | 
|---------------------------------|:------------------------------------------:|  
| **`-t`** or **`--tty`**         | pseudo-terminal                            | 
| **`-i`** or **`--interactive`** | Interactive mode                           |   
| **`-d`** or **`--detach`**      | means the container runs in the background |
| **`--name`**                    | specify the name of the container          |
| **`-e`**                        | helps specify environment variables        |    

#### **`Example`:**
```bash
$ sudo podman run --name mysql-custom \
	-e MYSQL_USER=redhat -e MYSQL_PASSWORD=r3dhat \
	-e MYSQL_DATABASE=items  -e MYSQL_ROOT_PASSWORD=r00tpa55 
	-d rhscl/mysql-57-rhel7:5.7-3.14

# enter the *my-sql-custom* container in interactive mode and
$ sudo podman exec -it mysql-custom /bin/bash
#connect to database within container
$ mysql -uroot
```
#### **`Managing Containers:`**
- Listing containers
```zsh
#list running contianers
$ sudo podman ps  
#show stopped containers	
$ sudo podman ps -a 
# Verify containers started without errors	
$ sudo podman ps --format "{{.ID}} {{.Images}} {{.Names}}"	
```
- Executing processes within a container
```zsh
#The exec command starts an additional process inside an already running container
$ sudo podman exec <container_id> cat/etc/hostname
#Executing additional processes within a container in interactive mode
$ sudo podman exec -it <image name> <command> 
#You can skip writing container ID or name in later Podman commands by replace container ID with `-l` option:
$ sudo podman exec -l
```
- Inspect lists metadata about running or stopped containers. The command produces **`JSON`** output.
```zsh
#Inspect lists metadata about running or stopped container:
$ sudo podman inspect my-httpd-container
#inspect http container json file and find IPAddress field
$ sudo podman inspect -f '{{.NetworkSettings.IPAddress}}' my-httpd-container
```
- Stop or Kill running contianers
```zsh
#stop and kill a running container
$ sudo podman stop <container_name>
#Before deleting all containers all running containers must be in a "stopped" status
$ sudo podman stop -a

#More aggresive way of stopping container process.
$ sudo podman kill <container_name> 
#You can specify the Unix signal to be sent to the main process. 
#Podman accepts either the signal name and number
$ sudo podman kill -s SIGKILL <container_name>
```
- Restart and remove containers
```zsh
# Restart a stopped container
$ sudo podman restart <container_name>

# deletes a container and discordes its state and file system
$ sudo podman rm <container_name>
#remove all available containers or images
$ sudo podman rm -a 
```

#### **`Creating Persistent Storage:`**
Container storage is said to be ephemeral, meaning its contents are not preserved after the container is removed. Containerized applications work on the assumption that they always start with empty storage, and this makes creating and destroying containers relatively inexpensive operations. Ephemeral container storage is not sufficient for applications that need to keep data over restarts, such as databases. To support such applications, the administrator must provide a container with persistent storage.
> *High level steps for creating persistent storage:*
> Create directory with owner and group root.
```zsh
$ sudo mkdir -pv /var/dbfiles
```
> Grant write access to the directory for MYSQL service which has a UID of 27.
```zsh
$ sudo chown -Rv 27:27 /var/dbfiles
```
> Apply **`container_file_t`** context to the directory(all all subdirectories) to allow containers access to all of its contents.
```zsh
$ sudo semanage fcontext -a -t container_file_t 'var/dbfiles(/.*)?'
```
> Apply **`SELinux`** container policy.
```zsh
$ sudo restorecon -R /var/dbfiles/
```
> Confirm policy was applied.
```zsh
$ ls -dZ /var/dbfiles
```
> Mount volume from container to host machine. In order to do so, you have to specify the **`-v`** option.
```zsh
$ sudo podman run -v /var/dbfiles:/var/lib/mysql rhmap47/mysql
```

#### **`Accessing Containers: `**
- Containers created with Podman running on different hosts belong to different software-defined networks. 
- Each SDN is isolated, which prevents a container in one network from communicating with a container in a different network. 
- Because of network isolation, a container in one SDN can have the same IP address as a container in a different SDN.
> *It is also important to note that, by default, all container networks are hidden from the host network. That is, containers typically can access the host network, but without explicit configuration, there is no access back into the container network.*

##### **`Mapping Network Ports: `**
- In order to map a network port, you can use the **`sudo podman run`**, followed by the **`-p`** option. 
- The **`-p`** option is following by [<IP address>:][<host port>:]<container port>

> *For Example:*
```zsh
#accessing container*
$ sudo podman run -d --name apache1 -p 8080:80 rhscl/httpd-24-rhel7:2.4
```
> *You can also use the -p option to only forward requests to a container if those requests originate from a specified IP address:*
```zsh
$ sudo podman run -d --name apache2 \
> -p 127.0.0.1:8081:80 rhscl/httpd-24-rhel7:2.4
```
> *To see ports assigned by podman, run:*
```zsh
$ sudo podman port <container_name>
>> 80/tcp -> 127.0.0.1:35134
$ curl 127.0.0.1:35134
>> <html><body><h1>It works!</h1></body></html>
```
- If only a container port is specified with the -p option, a random available host port is assigned to container. 
- Requests to this assigned host port from any IP address are forwarded to the container port.
```zsh
$ sudo podman run -d --name apache4 -p 80 rhscl/httpd-24-rhel7:2.4
$ sudo podman port apache4
>> 80/tcp -> 0.0.0.0:37068
```

#### **`Managing Container Images :`**
- To configure registries for the podman command, you need to update the **`/etc/containers/registries.conf`** file. 
- Edit the registries entry in the **`[registries.search]`** section, adding an entry to the values list as show below.
```zsh
[registries.search]
registries = ["registry.access.redhat.com", "quay.io"]
```
- Secure connections to a registry require a trusted certificate. To support insecure connections, add the commands below in the same file.
```zsh
[registries.insecure]
registries = ['localhost:5000']
```
- Podman search command finds images by image name, user name., or description from all
registeries listed in the **`/etc/containers/registries.conf`** configuration file. 

> *Example Usage*
**sudo podman search [OPTIONS] <term>**	
#### **`Useful Search CLI Options:`**

|         **Options**                       |     **Description**                                                            | 
|-------------------------------------------|:------------------------------------------------------------------------------:|  
| **`--limit <number>`**                    | Limits the number of listed images per registry.                               | 
| **`--filter <filter=value>`**             | Filter output based on conditions provided. Supported filters are below.       |
|    **`stars=<number>`**                   |  Show only images with at least this number of stars.                          |
| **`is-automated=<true>`** or **`false>`** |  Show only images automatically built.                                         |
|  **`is-official=<true>`** or **`false>`** |  Show only images flagged as official.                                         | 
| **`--tls-verify=<true>`** or **`false>`** | Enables or disables **`HTTPS`** certificate validation for all used registries.|    

#### **`Registry Authentication: `**
> Some container image registries require access authorization.
> The **`podman login`** command allows username and password authentication to a registry as shown below.
```zsh
$ sudo podman login -u username \
	-p password registry.access.redhat.com
```
>> *Login Succeeded!*

#### **`Pulling Images:`**
To pull container images from a registry, use the **`podman pull`** command:

> *Example Format*
```zsh
$ sudo podman pull [OPTIONS] [REGISTRY[:PORT]/]NAME[:TAG] #generic format
$ sudo podman pull quay.io/bitnami/nginx #Pull from Quay registry
```
> *List all container images stored locally.*
```zsh
sudo podman images
```
#### **`Manipulating Container Images :`**

There are many different ways to manage container images, and this section will cover some of the techniques which we can use to do so, while adhering to DevOps principles.
	1. Saving and image to a **`.tar`** file.
	2. Publishing the container image to an image registry.
> *Existing images from the Podman local storage can be saved to a .tar file using the **`podman save`** command. The generated file is not a regular TAR archive and contains image metadata and preserves the original image layers. Using this file, Podman can recreate the original image exactly as it was.*

The syntax for saving using the **`podman save`** command is:
```zsh
$ sudo podman save [-o FILE_NAME] IMAGE_NAME[:TAG]
```
> *Save and image to file called *mysql.tar*, based on the RHEL image.*
```zsh
$ sudo podman save -o mysql.tar registry.access.redhat.com/rhscl/mysql-57-rhel7
```
- Use the .tar files generated by the save subcommand for backup purposes. 
- To restore the container image, use the **`podman load`** command. 
- The general syntax of the command is as follows:							
```zsh
$ sudo podman load [-i FILE_NAME]
```
> load an image saved in a file named mysql.tar.
```zsh
$ sudo podman load -i mysql.tar
```
> *To save disk space, compress the file generated by the save subcommand with Gzip using the **`--compress parameter`**. The load subcommand uses the gunzip command before importing the file to the local storage.*
> Removing images follows the following syntax below.
```zsh
$ sudo podman rmi [OPTIONS] IMAGE [IMAGE....]
```
- An image can be referenced using its name or its **ID** for removal purposes. 
- Podman cannot delete images while containers are using that image, thereofore, you must stop and remove **ALL** containers using that image before deleting it.
- You can avoid this by passing the **`--force`** when executing the **`rmi`** command. has the --force option. 
- This option forces the removal of an image even if that the image is used by several containers or these containers are running. 
- Podman stops and removes all containers using the forcefully removed image before removing it.
> Delete all images not currently being used by a container, risky! 
```zsh
$ sudo podman rmi -a 
```
#### **`Modifying Images: `**
- Even though the **`podman commit`** command is the most straightforward approach to creating new images, it is not recommended because of the image size.
- Commit keeps logs and process ID files in the captured layers.
-  A **`Dockerfile`** provides a robust mechanism to customize and implement changes to a container using a human-readable set of commands.
- Additionally **`Dockerfiles`** do not keep extraneous set of files that are generated by the operating system.
> *Commit a messge example format:*
```zsh
$ sudo podman commit [OPTIONS] CONTAINER [REPOSITORY[:PORT]/]IMAGE[:TAG]
```
```zsh
$ sudo podman commit mysql-basic mysql-custom	
```
#### **`Useful Podman Commit Options:`**

|         **Options**             |     **Description**                                                             | 
|---------------------------------|:-------------------------------------------------------------------------------:|  
| **`--author`**                  | Identifies who created the container image.                                     | 
| **`--message`**                 | Includes a commit message to the registry.                                      |   
| **`--format`**                  | Selects the format for the image. Valid options are **`oci`** and **`docker`**. |

 
```bash 

# See the changes made in an image
$ sudo podman diff <image_name>
$ sudo podman inspect \
	-f "{{range .Mounts}}{{println .Destination}}{{end}}" *CONTAINER_NAME/ID*

# tag an image
$ sudo podman tag [OPTIONS] IMAGE[:TAG] \
	[REGISTRYHOST/][USERNAME/]NAME[:TAG]	

$ sudo podman tag mysql-custom devops/mysql:snapshot

#remove tag from the image
$ sudo podman rmi devops/mysql:snapshot   									

#Push image to registry
#if we don't specify destination,podman will use one of the default registries.
$ sudo podman push [OPTIONS] IMAGE [DESTINATION]
$ sudo podman push quay.io/*USER_NAME*/ngix
#oull image
$ sudo podman pull docker.io/nginx:17

```





