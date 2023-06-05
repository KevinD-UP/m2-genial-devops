# Database

### QUESTION 1-1 Document your database container essentials: commands and Dockerfile.

The Dockerfile is using postgres:14.1-alpine image to run a postgresql database

We also use `COPY` to copy the sql file into the `/docker-entrypoint-initdb.d` directory inside the container to run them.

## Build
First we need to create a volume to persist data :
```bash
docker volume create my_volume
docker network create tp1-network
docker build . -t database
```

## Run
```bash
docker run --env POSTGRES_DB=db --env POSTGRES_USER=usr --env POSTGRES_PASSWORD=pwd -p 5432:5432 -v my_volume:/var/liv/postgresql/data --name database --network tp1-network database
```

To answer to the one of a tip inside the TP :
The -env (or -e) flag in Docker is used to pass environment variables to a container when it is run. Environment variables are a way to provide configuration or runtime information to an application or service inside the container.

Here are a few reasons why using the -env flag to provide environment variables is beneficial:

Configuration flexibility: Environment variables allow you to decouple the configuration of your containerized application from the container image itself. By using environment variables, you can modify the behavior of the application without having to rebuild or modify the container image. This flexibility is particularly useful when deploying the same container image to different environments (e.g., development, staging, production) with varying configurations.

Security: Environment variables can be used to store sensitive information such as API keys, database credentials, or other secrets. By passing these values as environment variables, you can avoid hard-coding them directly into the container image or exposing them in command-line arguments. This approach helps to prevent accidental exposure of sensitive information.

Scalability and portability: By using environment variables, you can make your containerized application more scalable and portable. You can dynamically adjust the behavior of the application by changing the environment variables without requiring modifications to the container image or the application code. This flexibility allows for easier scaling and deployment across different environments.

Consistency and standardization: Environment variables provide a standardized way to configure containerized applications. By adopting a consistent approach across your containerized ecosystem, you can ensure that different services or microservices are configured in a similar manner, making them easier to manage and maintain.

# Backend API

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

## Build
```
docker build -t http-server .
```

## Run
```
docker run -dit --network tp1-network --name http-server -p 8081:80 http-server
```

# Link application

### 1-3 Document docker-compose most important commands. 

### 1-4 Document your docker-compose file.

### 1-5 Document your publication commands and published images in dockerhub.