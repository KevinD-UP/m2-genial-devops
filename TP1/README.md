# Database

## QUESTION 1-1 Document your database container essentials: commands and Dockerfile.

Dockerfile:

```
FROM postgres:14.1-alpine

COPY ./01-CreateScheme.sql /docker-entrypoint-initdb.d/
COPY ./02-InsertData.sql /docker-entrypoint-initdb.d/
```

Explanation:

- FROM postgres:14.1-alpine: This line sets the base image for the Docker image as postgres:14.1-alpine, which is an official PostgreSQL image based on Alpine Linux.
- COPY ./01-CreateScheme.sql /docker-entrypoint-initdb.d/: This line copies the file named 01-CreateScheme.sql from the current directory to the /docker-entrypoint-initdb.d/ directory inside the Docker image. This directory is a special location where scripts placed in it will be automatically run when the container is first started.
- COPY ./02-InsertData.sql /docker-entrypoint-initdb.d/: This line copies the file named 02-InsertData.sql from the current directory to the /docker-entrypoint-initdb.d/ directory inside the Docker image.

Commands:

1. Create a Docker volume:
```
docker volume create my_volume
```

Explanation:

- docker volume create my_volume: This command creates a Docker volume named my_volume. Volumes are used to persist data between container runs.

2. Create a Docker network:
```
docker network create tp1-network
```

Explanation:

- docker network create tp1-network: This command creates a Docker network named tp1-network. Networks are used to enable communication between containers.

3. Build the Docker image:
```
docker build . -t database
```

Explanation:

- docker build .: This command builds a Docker image using the Dockerfile found in the current directory (. represents the current directory).
- -t database: This option assigns the tag database to the built image. The tag can be used later to reference the image.

4. Run the Docker container:

```
docker run --env POSTGRES_DB=db --env POSTGRES_USER=usr --env POSTGRES_PASSWORD=pwd -p 5432:5432 -v my_volume:/var/liv/postgresql/data --name database --network tp1-network database
```

Explanation:

- docker run: This command starts a new Docker container based on an image.
- --env POSTGRES_DB=db: This option sets the environment variable POSTGRES_DB to the value db. This specifies the name of the default database to be created in the PostgreSQL container.
- --env POSTGRES_USER=usr: This option sets the environment variable POSTGRES_USER to the value usr. This specifies the username for the default database user.
- --env POSTGRES_PASSWORD=pwd: This option sets the environment variable POSTGRES_PASSWORD to the value pwd. This specifies the password for the default database user.
- -p 5432:5432: This option maps the container's port 5432 (PostgreSQL's default port) to the host's port 5432. This allows external access to the PostgreSQL server running inside the container.
- -v my_volume:/var/liv/postgresql/data: This option mounts the Docker volume my_volume to the directory /var/liv/postgresql/data inside the container. This allows data to be persisted in the volume between container runs.
- --name database: This option assigns the name database to the running container.
- --network tp1-network: This option connects the container to the Docker network named tp1-network.
- database: This specifies the image (tagged as database) to be used for the container.

Please note that when using the provided Dockerfile and commands, you need to make sure that the SQL files 01-CreateScheme.sql and 02-InsertData.sql are present in the same directory as the Dockerfile.

# Backend API

### Dockerfile: 
```
# Build
FROM maven:3.8.6-amazoncorretto-17 AS myapp-build
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn package -DskipTests

# Run
FROM amazoncorretto:17
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar

ENTRYPOINT java -jar myapp.jar
``` 

### Build
```
docker build . -t backend
```

### Run
```
docker run -p 8080:8080 --network tp1-network --name backend backend
```

### Question 1-2 Why do we need a multistage build? And explain each step of this dockerfile.

A multistage build in Docker allows you to create multiple stages or steps in your build process, where each stage can have its own base image and instructions. This is useful for optimizing the size and security of your final Docker image by separating the build environment from the runtime environment.

In the provided Dockerfile, there are two stages defined: myapp-build and the final stage for running the application.

Here's a breakdown of each step in the Dockerfile:

Stage: myapp-build

```
FROM maven:3.8.6-amazoncorretto-17 AS myapp-build
```
This line sets the base image for the build stage to maven:3.8.6-amazoncorretto-17. It includes Maven and Amazon Corretto 17 (a distribution of the OpenJDK Java runtime).

```
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
```
These lines set the environment variable MYAPP_HOME to /opt/myapp and set it as the working directory within the container.

```
COPY pom.xml .
COPY src ./src
These lines copy the pom.xml file and the src directory (containing source code) into the container's working directory.
```

```
RUN mvn package -DskipTests
```
This line executes the mvn package command to build the Java application. The -DskipTests option skips running any tests during the build process. The resulting JAR file will be generated in the target directory.

Stage: Run

```
FROM amazoncorretto:17
```
This line sets the base image for the final runtime stage to amazoncorretto:17, which includes Amazon Corretto 17 (OpenJDK Java runtime).

```
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
```
These lines set the environment variable MYAPP_HOME to /opt/myapp and set it as the working directory within the container (same as in the build stage).

```
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar
```
This line copies the JAR file from the previous myapp-build stage to the current stage. It uses the --from flag to specify the source stage (myapp-build). The JAR file is copied to the $MYAPP_HOME/myapp.jar path.

```
ENTRYPOINT java -jar myapp.jar
```
This line specifies the default command that will be executed when the container starts. It runs the java -jar myapp.jar command to start the Java application inside the container.

By using a multistage build, the final Docker image only contains the necessary runtime dependencies, reducing the image size and potential security risks. The build stage includes Maven and builds the Java application, while the runtime stage only includes the necessary Java runtime environment to execute the application.

# http server

### Dockerfile:
```
FROM httpd:2.4.57
COPY ./public-html/ /usr/local/apache2/htdocs/
COPY ./httpd.conf /usr/local/apache2/conf/httpd.conf
```

### Build
```
docker build -t http-server .
```

### Run
```
docker run -dit --network tp1-network --name http-server -p 8081:80 http-server
```

# Link application

## 1-3 Document docker-compose most important commands. 
Here are some of the most important commands you can use when working with docker-compose:

1. docker-compose up: Starts and runs your containers defined in the docker-compose.yml file. It builds the images if they don't exist and creates and attaches to the containers.

2. docker-compose down: Stops and removes the containers, networks, and volumes defined in the docker-compose.yml file.

3. docker-compose build: Builds or rebuilds the images defined in the docker-compose.yml file. Use this command if you have made changes to your Dockerfiles or need to update the images.

4. docker-compose start: Starts existing containers defined in the docker-compose.yml file without building or recreating them.

5. docker-compose stop: Stops running containers defined in the docker-compose.yml file without removing them.

6. docker-compose restart: Restarts containers defined in the docker-compose.yml file.

7. docker-compose logs: Displays the logs of the containers defined in the docker-compose.yml file.

8. docker-compose ps: Shows the status of the containers defined in the docker-compose.yml file.

9. docker-compose exec: Runs a command in a running container. For example, docker-compose exec <service_name> <command>.

10. docker-compose down -v: Stops and removes containers, networks, and volumes defined in the docker-compose.yml file. This command also removes the volumes associated with the containers.

11. docker-compose config: Validates and displays the composed configuration. It can be used to check for syntax errors in the docker-compose.yml file.

12. docker-compose pull: Pulls the latest image versions defined in the docker-compose.yml file from the registry.

These are some of the essential docker-compose commands that can help you manage your Docker applications effectively. 
Feel free to explore the official Docker documentation for more details and advanced usage: https://docs.docker.com/compose/reference/


## 1-4 Document your docker-compose file.

```yaml
version: '3.7'

services:
    backend:
        build:
          context: ./backend
          dockerfile: Dockerfile
        networks:
          - tp1-network
        depends_on:
          - database

    database:
        build:
          context: ./database
          dockerfile: Dockerfile 
        environment:
          - POSTGRES_USER=usr
          - POSTGRES_PASSWORD=pwd
          - POSTGRES_DB=db
        networks:
          - tp1-network
        volumes:
          - my_volume:/var/liv/postgresql/data

    httpd:
        build:
          context: ./http-server
          dockerfile: Dockerfile
        ports:
          - 81:80
        networks:
          - tp1-network
        depends_on:
          - backend 
          - database

networks:
  tp1-network:

volumes: 
  my_volume:

```

This docker-compose.yml file defines three services: backend, database, and httpd.

The backend service:

- It builds the image for the backend service using the Dockerfile located in the ./backend directory.
- It is connected to the tp1-network network.
- It depends on the database service, meaning the database service will start before the backend service.

The database service:

- It builds the image for the database service using the Dockerfile located in the ./database directory.
- It sets environment variables for the PostgreSQL database, including the username, password, and database name.
- It is connected to the tp1-network network.
- It creates a volume named my_volume and mounts it to the /var/liv/postgresql/data directory inside the container. This allows the database data to persist even if the container is restarted or recreated.

The httpd service:

- It builds the image for the HTTP server service using the Dockerfile located in the ./http-server directory.
- It maps port 81 on the host machine to port 80 inside the container, allowing access to the HTTP server from the host.
- It is connected to the tp1-network network.
- It depends on both the backend and database services, so they will start before the httpd service.
The tp1-network network:

- It defines a custom network named tp1-network. All the services that are connected to this network can communicate with each other using their service names as hostnames.

The my_volume volume:

- It creates a named volume named my_volume. This volume will be used by the database service to store the PostgreSQL data.

## 1-5 Document your publication commands and published images in dockerhub.

1. docker tag database USERNAME/database:1.0

- This command tags an existing Docker image with the name database using the format USERNAME/database:1.0. It creates a new tag for the image with the version 1.0. The USERNAME should be replaced with your Docker username or the desired repository name.

2. docker push USERNAME/database

- This command pushes the Docker image with the tag USERNAME/database to a Docker registry. It uploads the image to the registry associated with your Docker username or the specified repository.

3. docker tag backend USERNAME/backend:1.0

- This command tags an existing Docker image with the name backend using the format USERNAME/backend:1.0. It creates a new tag for the image with the version 1.0. The USERNAME should be replaced with your Docker username or the desired repository name.

4. docker push USERNAME/backend

- This command pushes the Docker image with the tag USERNAME/backend to a Docker registry. It uploads the image to the registry associated with your Docker username or the specified repository.

5. docker tag http-server USERNAME/http-server:1.0

- This command tags an existing Docker image with the name http-server using the format USERNAME/http-server:1.0. It creates a new tag for the image with the version 1.0. The USERNAME should be replaced with your Docker username or the desired repository name.

6. docker push USERNAME/http-server

- This command pushes the Docker image with the tag USERNAME/http-server to a Docker registry. It uploads the image to the registry associated with your Docker username or the specified repository.

Remember to replace USERNAME with your Docker username or the desired repository name in order to push the images to the correct location.

