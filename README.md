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

## 🚀 How to Deploy 

** ✅ Prerequisites **

To replicate this setup, you will need:

* An AWS account with permissions for IAM, CodePipeline, CodeBuild, CodeDeploy, CodeArtifact, S3, and EC2.
* An EC2 instance (Linux) designated as the "Web Server" with the **CodeDeploy agent** installed and running.
* An AWS CodeArtifact domain and repository set up.
* An S3 bucket for CodePipeline artifacts.
* Proper IAM roles for CodeBuild and CodeDeploy to access other AWS services.

---

## 🚀 How to Deploy 
**Deployment Steps:**

1.  **Customize Configuration Files:**
    * **CloudFormation (`template.yml`):** Modify parameters like VPC CIDR blocks, instance types, Key Pair names, and any specific resource configurations to match your requirements. You might need to adjust IAM roles and security group rules.
    * **`buildspec.yml`:** Update the build commands based on your project's structure and build tools (e.g., specific Maven goals, Java version). Configure CodeArtifact settings if you are using it.
    * **`appspec.yml`:** Adjust the `files` section to specify the source artifact (e.g., `.war` file) and its destination on the EC2 instance. Modify the lifecycle hooks (`BeforeInstall`, `AfterInstall`, `ApplicationStart`, etc.) to run the correct scripts for your application server (e.g., Tomcat, Jetty).
    * **`settings.xml`:** Update repository URLs and credentials if using CodeArtifact or another private repository.
    * **Application Code:** Replace the example `index.jsp` with your actual application code.

2.  **Provision Infrastructure (CloudFormation):**
    * Deploy the customized CloudFormation template (`template.yml`) via the AWS Management Console or AWS CLI. This will create the VPC, EC2 instance(s), IAM roles, security groups, and potentially the CodeDeploy application and deployment group.
    * Ensure the EC2 instances have the CodeDeploy agent installed and configured.

3.  **Set Up CodeArtifact (If Used):**
    * Create your CodeArtifact domain and repository.
    * Configure `settings.xml` and `buildspec.yml` to use the CodeArtifact repository endpoint.

4.  **Create the CI/CD Pipeline (CodePipeline):**
    * Use the AWS Management Console or AWS CLI to create a new CodePipeline.
    * **Source Stage:** Connect to your GitHub repository (you might need to create an AWS CodeStar connection). Specify the repository and branch.
    * **Build Stage:** Configure the build stage to use AWS CodeBuild. Point it to the `buildspec.yml` file in your repository. Configure input/output artifacts (source code in, build artifact out to S3). Set up necessary permissions for CodeBuild to access CodeArtifact or other services.
    * **Deploy Stage:** Configure the deploy stage to use AWS CodeDeploy. Select the CodeDeploy application and deployment group created by CloudFormation (or manually). Specify the build artifact from the S3 bucket as the input.

5.  **Trigger the Pipeline:**
    * Push a change to your configured GitHub branch.
    * CodePipeline should automatically detect the change and execute the source, build, and deploy stages.

6.  **Verify Deployment:**
    * Access the public IP address or DNS name of your EC2 instance (or Load Balancer) in a web browser to verify that your application has been deployed successfully.
    * Monitor the pipeline execution status in the AWS CodePipeline console. Check logs in CodeBuild and CodeDeploy for any errors.

**Note:** This is a high-level overview. Each step involves detailed configuration within the respective AWS services. Refer to the official AWS documentation for specific setup instructions.
