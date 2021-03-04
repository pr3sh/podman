# Abstract

The objective of this guy is to cover deployments of multicontainer applications using **`podman`**.


- change into directory with your directory contianing your **Dockerfile**, and build your **`my-sql`** image.



### A typical top-level directory layout

    .
    ├── Dockerfile                   # Compiled files (alternatively `dist`)
    ├── docs                    # Documentation files (alternatively `doc`)
    ├── src                     # Source files (alternatively `lib` or `app`)
    ├── test                    # Automated tests (alternatively `spec` or `tests`)
    ├── root                     # Tools and utilities
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