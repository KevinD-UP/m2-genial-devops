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

## 2-3 Document your quality gate configuration.

To configure the quality gate in SonarCloud, you can follow these steps:

1. Go to the SonarCloud website (https://sonarcloud.io) and sign in to your account.

2. Select the project you want to configure the quality gate for.

3. Click on "Quality Gates" in the left sidebar.

4. Create a new quality gate by clicking on the "+ Create" button.

5. Give your quality gate a name and define the conditions for passing or failing. These conditions can include metrics such as code coverage, code duplication, code smells, and other code quality metrics. You can define thresholds for each metric based on your project requirements.

6. Save the quality gate configuration.

In the test job we can now add this command :
```yaml
mvn clean verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=KevinD-UP_m2-genial-devops -Dsonar.login=${{ secrets.SONAR_TOKEN }}
```

For now, we are using Sonar way which is the default quality gate.

Here is the spec :

Conditions

- Coverage is less than 80.0%
- Duplicated Lines (%) is greater than 3.0%
- Maintainability Rating is worse than A
- Reliability Rating is worse than A
- Security Hotspots Reviewed is less than 100%
- Security Rating is worse than A
