
### Abstract

Overview of using podman

```bash
$ sudo Rodman search rhel                		# search for the rhel image
$ sudo podman pull <image_name>`                     	# pull an image
$ sudo podman run  rhel7:7.5 echo "Hello world"` 	# run the rhel image and echo "Hello world"
$ sudo podman ps 					# list running containers
```
*options:*
```bash
-t or --tty 		# meaning (pseudo-terminal) 
- or --interactive 	# Interactive mode
-d or --detach 		# means the container runs in the background
--name 			# specify the name of the container
-e			# helps specify environment variables
```
*Example:* 
```bash
$ sudo podman run --name mysql-custom \
	-e MYSQL_USER=redhat -e MYSQL_PASSWORD=r3dhat \
	-e MYSQL_DATABASE=items  -e MYSQL_ROOT_PASSWORD=r00tpa55 
	-d rhscl/mysql-57-rhel:5.7-3.14`

#connect to database within container
$ mysql -uroot
```


```bash
# Verify containers started without errors
$ sudo Rodman ps --format "{{.ID}} {{.Images}} {{.Names}}"
$ sudo podman ps -a

#  enter the *my-sql-custom* container in interactive mode and
$ sudo podman exec -it mysql-custom /bin/bash

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


1.  Create directory with owner and group root
2.  grant write access to the directory for MYSQL service which has a UID of 27
3.  Apply `container_file_t` context to the directory(all all subdirectories) to allow containers access to all of its contents
4.  Apply SELinux container policy
5.  Mount volume within container
6.  confirm policy was applied
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
$ sudo podman save [-o FILE_NAME] IMAGE_NAME[:TAG]
$ sudo podman save -o mysql.tar registry.access.redhat.com/rhscl/mysql-57-rhel7
$ sudo podman load -I mysql.tar
$ sudo poems rmi [OPTIONS] IMAGE [IMAGE....]
$ sudo podman rmi -a #delete all images not currently being used by a container, risky!
```
*Example usage*
```bash
$ sudo podman commit [OPTIONS] CONTAINER \
 	[REPOSITORY[:PORT]/]IMAGE[:TAG]
```

- *Options:*
```bash
	--author  ""  		# Identifies who created the container image .
	--message ""  		# Includes a commit message to the registry.
	--format      		# Selects the format for the image.
		       			# Valid options are oci and docker.
```
$ sudo podman ps
$ sudo podman diff <image_name>

$ sudo podman tag [OPTIONS] IMAGE[:TAG] [REGISTRY/][USERNAME/]NAME[:TAG]
$ sudo podman tag mysql-custom devops/mysql:snapshot
$ sudo podman rmi devops/mysql:snapshot   #remove tag from image
$ sudo podman push [OPTIONS] IMAGE [DESTINATION] #if we don't specify destination,podman will use one of the default registries.
$ sudo podman






