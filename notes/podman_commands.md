
## **`Abstract`:**

-  **`Table of contents`:**
	- [Fetching Container Images](#fetching-container-images)
	- [Running Containers](#running-containers)
		- [Useful Podman CLI Options](#useful-podman-cli-options)
		- [Example](#example)
	- [Managing Containers](#managing-containers)	
	- [Creating Persistent Storage](#creating-persistent-storage)
	- [Manipulating Container Images](#manipulating-container-images)
	- [Accessing Containers](#accessing-containers)
		- [Mapping Network Ports](#mapping-network-ports)
	- [Managing Container Images](#managing-container-images)
		

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
#### **`Example`:***

```zsh
$ sudo podman run --name mysqldb-port -e MYSQL_USER=user -e MYSQL_PASSWORD=mypa55 \
	-e MYSQL_DATABASE=items -e MYSQL_ROOT_PASSWORD=r00tpa55 -p 13306:3306 -d  \
	-v /var/local/mysql:/var/lib/mysql/data rascal/mysql-57-rhel7 
```



#### **`Managing Container Images :`**

Podman search command finds images by image name, user name., or description from all
registeries listed in the **`/etc/containers/registries.conf`** configuration file. 
> *Example Usage*
**sudo podman search [OPTIONS] <term>**	
#### **`Useful Search CLI Options:`**

|         **Options**             |     **Description**                                                            | 
|---------------------------------|:------------------------------------------------------------------------------:|  
| **`--limit <number>`**          | Limits the number of listed images per registry.                               | 
| **`--filter <filter=value>`**   | Filter output based on conditions provided. Supported filters are below.       |
|    **`stars=<number>`**         |  Show only images with at least this number of stars.                          |
| **`is-automated=<true|false>`** |  Show only images automatically built.                                         |
|  **`is-official=<true|false>`** |  Show only images flagged as official.                                         | 
| **`--tls-verify <true|false>`** | Enables or disables **`HTTPS`** certificate validation for all used registries.|    


```zsh
# Save and image to file called *mysql.tar*, based on the RHEL image.
$ sudo podman save [-o FILE_NAME] IMAGE_NAME[:TAG]
$ sudo podman save -o mysql.tar registry.access.redhat.com/rhscl/mysql-57-rhel7

# Load image based on tarfile 							
$ sudo podman load [-i FILE_NAME]
$ sudo podman load -i mysql.tar

# Remove image
$ sudo podman rmi [OPTIONS] IMAGE [IMAGE....]
#delete all images not currently being used by a container, risky!
$ sudo podman rmi -a 
```
> *Commit a messge example format:*
```zsh
$ sudo podman commit [OPTIONS] CONTAINER \
 	[REPOSITORY[:PORT]/]IMAGE[:TAG]
```
- *Options:*
	--author  ""  		# Identifies who created the container image .
	--message ""  		# Includes a commit message to the registry.
	--format      		# Selects the format for the image.
		       			# Valid options are oci and docker.

$ sudo podman commit mysql-basic mysql-custom		       			
 ```
 
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





