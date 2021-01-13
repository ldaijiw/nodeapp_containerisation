# Containerising the Node App

To containerise the node app the following actions are written in the Dockerfile
- ``FROM node``: specify the official node Docker container to build from
- ``WORKDIR /usr/src/app``: create app directory and set to working directory
- ``COPY app/ .`` copy local app folder to container
- ``RUN npm install`` run npm install to install all necessary dependencies
- ``EXPOSE 3000`` expose 3000 for app to run correctly
- ``CMD ["npm", "start", "app.js;"]`` final command to start the app

**Development Dockerfile**
```
FROM node

LABEL MAINTAINER=lwaltmann@spartaglobal.com

# Create app directory
WORKDIR /usr/src/app

COPY app/ .

RUN npm install

EXPOSE 3000

CMD ["npm", "start", "app.js;"]
```

- The image can then be built by changing to the directory containing the Dockerfile and running the following commands:
```
docker build -t ldaijiw/eng74_nodeapp .
```
- And running the container with
```
docker run -d -p 80:3000 ldaijiw/eng74_nodeapp
```
- typing in ``localhost`` in browser should then show the home page of the app
- after creating the DockerHub repository: ``eng74_nodeapp``, commit and push the working image to DockerHub with
```
docker commit <container_id> ldaijiw/eng74_nodeapp
docker push ldaijiw/eng74_nodeapp
```
The docker image can then be downloaded and run from any machine by specifying the image name: ``ldaijiw/eng74_nodeapp``

## Multi-Stage Build

Using the development image can be useful for testing and additional configuration, but the image size may be relatively large. Once everything is working with the development image, multi-stage builds can be used to create a lightweight image, without having to write a new Dockerfile.

Add another stage with a more lightweight image (``FROM node:alpine``) and copy any necessary files from the first stage, and then run any necessary commands. For the node app, multi-stage builds reduced the image size from ~970MB to ~120MB.

### Lightweight App Dockerfile

```
FROM node as APP

LABEL MAINTAINER=lwaltmann@spartaglobal.com

# Create app directory
WORKDIR /usr/src/app

COPY app/ .

RUN npm install

FROM node:alpine

COPY --from=app /usr/src/app /usr/src/app

WORKDIR /usr/src/app

EXPOSE 3000

CMD ["npm", "run", "seed;"]

CMD ["npm", "start", "app.js;"]
```

### MongoDB Dockerfile

The Dockerfile to configure the database is relatively straightforward, after copying the ``mongod.conf`` file to bind the IP to ``0.0.0.0``, and exposing port 27017, the service can be started.
```
FROM mongo

COPY mongod.conf /etc/

EXPOSE 27017

CMD ["mongod"]
```

## Docker Compose

[Docker Compose](https://docs.docker.com/compose/) is a tool for defining and running multi-container Docker applications. With Compose, a YAML file is used to configure the application's services. Then with a single command: ``docker-compose up``, the services are all created and started as defined in the configuration.
```
version: "3.9"
services:
  database:
    build: ./db/
    container_name: mongodb
    hostname: mongodb
    ports:
      - "27017:27017"
    restart: unless-stopped
  app:
    build: .
    container_name: nodeapp
    ports:
      - "80:3000"
    environment:
      - DB_HOST=database:27017
    depends_on:
      - database
    links:
      - database
```
The ``docker-compose.yml`` file carries the following vital actions:
- the services ``database`` and ``app`` are defined, which can be referred to throughout the YAML file
- either ``build`` or ``image`` can be specified to either build the container from a Dockerfile with a specified path, or an image that has already been pulled locally
- specify the port mappings
- add environment variable of the IP and port of the database (``database:27017``)
