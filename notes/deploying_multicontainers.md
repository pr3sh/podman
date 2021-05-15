## **`Abstract: `**

The objective of this guy is to cover deployments of multicontainer applications using **`podman`**.

-  **`Table of contents`:**
	- [Understing Multicontainer Deployments](#understanding-multicontainer-deployments)

#### **`Understanding Multicontainer Deployments:`**
- **`Podman`** uses Container Network Interface **`(CNI)`** to create a software-defined network **`(SDN)`** between all containers in the host.
- **`CNI`** assigns a new IP address to a container when it starts.
- Each container exposes all ports to other containers in the same **`SDN`** making services accessible within the same network. 
- Ports within the containers are exposed externally only when explicitly configured.
- Due to the dynamic nature of container **`IP addresses`**, applications cannot rely on either fixed IP addresses or fixed DNS host names to communicate with middleware services and other application services. 
- Containers with dynamic IP addresses can become a problem when working with multi-container applications because each container must be able to communicate with others to use services upon which it depends.

> *Example*
Examine an example of an application which is comprised of a front-end, back-end, and a database, all of which are containers of their own. The front-end container needs to retrieve the IP address of the back-end container. Similarly, the back-end container needs to retrieve the IP address of the database container. Additionally, the IP address could change if a container restarts, so a process is needed to ensure any change in IP triggers an update to existing containers.

![alt text](https://github.com/pr3sh/podman/blob/openshift/multicontainer-consideration.jpg?raw=true)

- change into directory with your directory contianing your **Dockerfile**, and build your **`my-sql`** image.












### Directory layout for **`my-sql`** build
    .
    ├── Dockerfile                                                      
    ├── training.repo                    
    ├── root                    
    └── README.md

- High-level steps of building a multi-container application which is based on **`node.js`** front-end , **`REST`** backend, and **`my-sql`** database for storage.
```bash 
#build sql image
$ sudo podman build -t path/to/mysql-rhel7 --layers=false

#build node application
$ sudo podmn build --layers=false -t path/to/nodejs  \
			--build-args NEXUS_BASE_URL=${NEXUS_SERVER}

$ sudo podman images



```