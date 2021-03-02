
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
