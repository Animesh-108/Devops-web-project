# AWS CI/CD Pipeline for a Java Web Application

This repository contains the source code for a Java Maven web application and all the necessary configuration files to deploy it using a fully automated CI/CD pipeline on AWS.

The pipeline leverages a suite of AWS DevOps tools to build, test, and deploy the application to a live web server (EC2 instance running Tomcat and Apache) every time a change is pushed to this repository.

## 🏛️ Architecture

The workflow is visualized in the following architecture diagram:

![CI/CD Pipeline Architecture](https://github.com/Animesh-108/Devops-web-project/blob/master/architecture-complete.png)

## 🛠️ Core Technologies

* **Application:** Java (JSP), built with Maven.
* **Web Server:** Apache Tomcat (proxied by Apache httpd).
* **CI/CD Pipeline:**
    * **Source:** GitHub
    * **Orchestration:** AWS CodePipeline
    * **Build:** AWS CodeBuild
    * **Artifacts:** AWS S3
    * **Deployment:** AWS CodeDeploy
    * **Package Management:** AWS CodeArtifact

---

## 🚀 How It Works: File by File

This repository is structured to be read by the AWS CI/CD services. Here are the key files and what they do.

### 1. The Application

* **`pom.xml`**: This is the Maven project file. It defines the project as `nextwork-web-project` and instructs Maven to package it as a `.war` file.
* **`src/main/webapp/index.jsp`**: This is the main webpage for the application.

### 2. The Build Process (AWS CodeBuild)

* **`buildspec.yml`**: This is the "instruction manual" for AWS CodeBuild.
    * **`pre_build`**: Authenticates with AWS CodeArtifact to securely fetch dependencies.
    * **`build`**: Compiles the Java code and builds the project using `mvn clean install`.
    * **`artifacts`**: Bundles the compiled `.war` file, the `appspec.yml`, and the `scripts` folder to be sent to S3 for the deployment phase.
* **`settings.xml`**: A Maven settings file used by the `buildspec.yml`. It configures Maven to use the CodeArtifact repository URL and the authentication token.

### 3. The Deployment Process (AWS CodeDeploy)

* **`appspec.yml`**: This is the "instruction manual" for AWS CodeDeploy, which runs on the target EC2 server.
    * **`files`**: Specifies that the `nextwork-web-project.war` file should be placed in the Tomcat webapps directory (`/usr/share/tomcat/webapps/`).
    * **`hooks`**: Defines the lifecycle scripts to run as the `root` user:
        * **`ApplicationStop`**: Runs `scripts/stop_server.sh` to gracefully stop the Tomcat and httpd services.
        * **`BeforeInstall`**: Runs `scripts/install_dependencies.sh` to install Tomcat and httpd. It also configures httpd as a reverse proxy, forwarding requests from port 80 to the Tomcat app on port 8080.
        * **`ApplicationStart`**: Runs `scripts/start_server.sh` to start and enable the Tomcat and httpd services, making the new application version live.

---

## ✅ Prerequisites

To replicate this setup, you will need:

* An AWS account with permissions for IAM, CodePipeline, CodeBuild, CodeDeploy, CodeArtifact, S3, and EC2.
* An EC2 instance (Linux) designated as the "Web Server" with the **CodeDeploy agent** installed and running.
* An AWS CodeArtifact domain and repository set up.
* An S3 bucket for CodePipeline artifacts.
* Proper IAM roles for CodeBuild and CodeDeploy to access other AWS services.
