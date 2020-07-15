---
layout: post
title:  "Docker-Compose cheat sheet"
date:   2020-07-15 13:10:00
tags:   [Docker]
---
### Description
This post is guding docker-compose.yml writing examples and easy way to docker-compose.
<br/>
<br/>


### docker-compose.yml file version
Version must be defined on the top of docker-compose.yml file. docker-compose.yml version is mapping with docker version. Just type `docker --version` and check with the flowing this table.
<br/>
| Compose file format | Docker Engine release |
| :-----------------: | :-------------------: |
| 3.8                 | 19.03.0+              |
| 3.7                 | 18.06.0+              |
| 3.6                 | 18.02.0+              |
| 3.5                 | 17.12.0+              |
| 3.4                 | 17.09.0+              |
| 3.3                 | 17.06.0+              |
| 3.2                 | 17.04.0+              |
| 3.1                 | 1.13.1+               |
| 3.0                 | 1.13.0+               |
| 2.4                 | 17.12.0+              |
| 2.3                 | 17.06.0+              |
| 2.2                 | 1.13.0+               |
| 2.1                 | 1.12.0+               |
| 2.0                 | 1.10.0+               |
| 1.0                 | 1.9.1.+               |
<br/>
<br/>


### Basic docker-compose.yml
```yml
# docker-compose.yml
version: '3.8' # Refer version mapping table above.

services: # Services
    web: # Define web container
        container_name: express-server # Set container name.
        build: # Build image with Dockerfile.custom from ./docker directory.
            context: ./docker # Directory Dockerfile exists.
            dockerfile: Dockerfile.custom # Custom name of Dockerfile.
        ports:
            - "80:8080" # Create binding from host's 80 port to container's 8080 port.
        volumes:
            - ./app:/app # Create directory voulme from host's ./app directory to container's /app directory.
        working_dir: /app # Set working directory of container.
        networks: # Set network band and ipv4 address.
            express-mongo: # Uses express-mongo custom network.
                ipv4_address: 172.0.0.10 # Container ipv4 address.
 
    db: # Define another db container
        container_name: mongodb # Set container name.
        image: mongo:latest # This container uses last version of mongo image without building Dockerfile.
        environment: # Set container environments.
            - MONGO_INITDB_DATABASE: web
            - MONGO_INITDB_ROOT_USERNAME: root
            - MONGO_INITDB_ROOT_PASSWORD: root
        networks:
            express-mongo:
                ipv4_address: 172.0.0.20
                
networks: # Define network.
    express-mongo: # Network name si express-mongo.
        driver: bridge # Use bridge.
        ipam:
            config:
                subnet: 172.0.0.0/24 # Network subnet.
```

It would be updated soon...
