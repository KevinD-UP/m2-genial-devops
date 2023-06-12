## 2-1 What are testcontainers?


Testcontainers refer to a concept and set of tools that facilitate the creation and management of disposable containers for testing purposes. While the original Testcontainers library is primarily used in the Java ecosystem, the concept itself can be applied to various programming languages and frameworks.

The core idea behind Testcontainers is to provide a lightweight and isolated environment for running tests that require external dependencies. These dependencies can include databases, message queues, web servers, third-party APIs, and other services that an application relies on.

By using containers, which are lightweight, isolated instances of software environments, Testcontainers allows developers to create consistent and reproducible testing environments. Containers can be easily spun up, configured, and torn down programmatically, ensuring that each test runs in a clean and controlled environment.

Testcontainers leverages containerization technologies like Docker to manage the lifecycle of these containers. Docker provides a platform for packaging applications and their dependencies into standardized containers that can be run on any system.


## 2-2 Document your Github Actions configurations.

```yaml
name: CI devops 2023

on:
  push:
    branches:
      - main
      - develop
  pull_request:

jobs:
  test-backend:
    runs-on: ubuntu-22.04

    steps:
      - name: Checkout code
        uses: actions/checkout@v2.5.0
        # This step checks out your repository's code for the workflow to run on

      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: 17
          distribution: 'adopt'
        # This step sets up JDK 17 for the workflow to use

      - name: Build and test with Maven
        run: |
          cd TP1/backend
          mvn test
        # This step changes the directory to 'TP1/backend' and runs the 'mvn test' command to build and test your backend code
```

