
## Abstract

-  **Table of contents**:
	- [Fetching Container Images](#fetching-container-images)
	- [Running Containers](#running-containers)
		- [Useful Commandline Options](#useful-commandline-options)
		
#### **Fetching Container Images:**

search for an image
```zsh
$ sudo podman search rhel                	
$ sudo podman pull <image_name> #Pull an image                    	
$ sudo podman images #List current images
```
#### **Running Containers**

To run containers, you invoke the **`sudo podman run`** command.
```zsh
$ sudo podman run  rhel7:7.5 echo "Hello world" 	# run the rhel image and echo "Hello world"
```
List running containers
```zsh
$ sudo podman ps 					
```
#### **Useful Commandline Options:**
```
- *options:*
```bash
-t  or   --tty 		        # meaning (pseudo-terminal) 
-i   or   --interactive 	# Interactive mode
-d  or   --detach 		    # means the container runs in the background
--name 			            # specify the name of the container
-e                          # helps specify environment variables
```

*Example:* 
```bash
$ sudo podman run --name mysql-custom \
	-e MYSQL_USER=redhat -e MYSQL_PASSWORD=r3dhat \
	-e MYSQL_DATABASE=items  -e MYSQL_ROOT_PASSWORD=r00tpa55 
	-d rhscl/mysql-57-rhel:5.7-3.14`
# Verify containers started without errors
$ sudo podman ps --format "{{.ID}} {{.Images}} {{.Names}}"\
#or 
$ sudo podman ps -a

#  enter the *my-sql-custom* container in interactive mode and
$ sudo podman exec -it mysql-custom /bin/bash
#connect to database within container
$ mysql -uroot
```


```bash
# The exec command starts an additional process inside an already running container
$ sudo podman exec <container_id> cat/etc/hostname

#You can skip writing container ID or name in later Podman commands by replace container ID with `-l` option:
$ sudo podman exec -l

#Inspect lists metadata about running or stopped container:
$ sudo podman inspect my-httpd-container

#inspect http container json file and find IPAddress field
$ sudo podman inspect -f '{{.NetworkSettings.IPAddress}}' my-httpd-container

#stop and kill a running container
$ sudo podman stop <container_name>
$ sudo podman kill <container_name> 

#You can skip writing container ID or name in later Podman commands by replace container ID with `-l` option:
$ sudo podman exec -l

#Inspect lists metadata about running or stopped container:
$ sudo podman inspect my-httpd-container

#inspect http container json file and find IPAddress field
$ sudo podman inspect -f '{{.NetworkSettings.IPAddress}}' my-httpd-container

#stop and kill a running container
$ sudo podman stop <container_name>
$ sudo podman kill <container_name> 

#You can specify the Unix signal to be sent to the main process. 
#Podman accepts either the signal name and number
$ sudo podman kill -s SIGKILL <container_name>

# Restart a stopped container
$ sudo podman restart <container_name>

# deletes a container nd discordes its state and file system
$ sudo podman rm <container_name>
$ sudo podman rm -a remove all available containers or images

#Before deleting all containers all running containers must be in a "stopped" status
$ sudo podman stop -a
```

#### Creating Persistent Storage

*steps:*
- Create directory with owner and group root
- grant write access to the directory for MYSQL service which has a UID of 27
- Apply `container_file_t` context to the directory(all all subdirectories) to allow containers access to all of its contents
- Apply SELinux container policy
- Mount volume within container
- confirm policy was applied
```bash
$ sudo mkdir -pv /var/dbfiles
$ sudo chown -Rv 27:27 /var/dbfiles
$ sudo semanage fcontext -a -t container_file_t 'var/dbfiles(/.*)?'
$ sudo restorecon -R /var/dbfiles/
$ ls -dZ /var/dbfiles
```

```bash
#accessing container*
$ sudo podman run -d --name apache4 -p 80 httpd:2.4

# To see ports assigned by podman
$ sudo podman port apache3
```

*Example*

```bash
$ sudo podman run --name mysqldb-port -e MYSQL_USER=user -e MYSQL_PASSWORD=mypa55 \
	-e MYSQL_DATABASE=items -e MYSQL_ROOT_PASSWORD=r00tpa55 -p 13306:3306 -d  \
	-v /var/local/mysql:/var/lib/mysql/data rascal/mysql-57-rhel7 
```

Podman search command finds images by image name, user name., or description from all
registeries listed in the `/etc/containers/registries.conf` configuration file 

`$ sudo podman search [options]`	

#### manipulting container images:

```bash
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

- *Commit a messge example format:*
```bash
$ sudo podman commit [OPTIONS] CONTAINER \
 	[REPOSITORY[:PORT]/]IMAGE[:TAG]

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





