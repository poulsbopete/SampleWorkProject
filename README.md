# Scenario: 
A warehousing company with many "brick and morter" stores spread across the USA. On the first and third Friday of every month, the sales spike due to members’ paydays. During these times, people checking our produce inventory within the stores (and online) goes up significantly. The company would like to make sure they are able to accurately display inventory so they are not selling produce that they do not have enough of. Anecdotal evidence from cahiers and customers claim inventory lookups are slow during busy peak times. The current strategy is to add more resources to the database cluster but we want to assure the business that they are ready to scale at a reasonable cost.

 

# Assumptions:
All warehouse stores have network connectivity to database resources, cloud, and local data centers
The business will want to provide fault tolorance leveraging on-premise servers and cloud resources
 Provide 99.99% uptime with <5 second response time for a reasonable cost
 Tools to Use:
Docker
Amazon AWS Elastic Container Service

# Create a Docker Image
Amazon ECS task definitions use Docker images to launch containers on the container instances in your clusters. In this section, you create a Docker image of a simple web application, and test it on your local system or EC2 instance, and then push the image to a container registry (such as Amazon ECR or Docker Hub) so you can use it in an ECS task definition.

Create a file called Dockerfile. A Dockerfile is a manifest that describes the base image to use for your Docker image and what you want installed and running on it. For more information about Dockerfiles, go to the Dockerfile Reference.

```touch Dockerfile```
Edit the Dockerfile you just created and add the following content.

---

FROM ubuntu:16.04
MAINTAINER Peter Simkins <psimkins@gmail.com>
LABEL Description="Cutting-edge LAMP stack, based on Ubuntu 16.04 LTS. Includes .htaccess support and popular PHP7 features" \
	License="Apache License 2.0" \
	Version="1.0"

RUN apt-get update
RUN apt-get upgrade -y

RUN apt-get install -y zip unzip
RUN apt-get install -y \
	php7.0 \
	php7.0-bz2 \
	php7.0-cgi \
	php7.0-cli \
	php7.0-common \
	php7.0-curl \
	php7.0-dev \
	php7.0-enchant \
	php7.0-fpm \
	php7.0-gd \
	php7.0-gmp \
	php7.0-imap \
	php7.0-interbase \
	php7.0-intl \
	php7.0-json \
	php7.0-ldap \
	php7.0-mbstring \
	php7.0-mcrypt \
	php7.0-mysql \
	php7.0-odbc \
	php7.0-opcache \
	php7.0-pgsql \
	php7.0-phpdbg \
	php7.0-pspell \
	php7.0-readline \
	php7.0-recode \
	php7.0-snmp \
	php7.0-sqlite3 \
	php7.0-sybase \
	php7.0-tidy \
	php7.0-xmlrpc \
	php7.0-xsl \
	php7.0-zip
RUN apt-get install apache2 libapache2-mod-php7.0 -y
RUN apt-get install mariadb-common mariadb-server mariadb-client -y

EXPOSE 80
EXPOSE 3306


---
This Dockerfile uses the Ubuntu 16.04 image. The RUN instructions update the package caches, install some software packages for the web server. The EXPOSE instruction exposes port 80 on the container, and the CMD instruction starts the web server.

Build the Docker image from your Dockerfile.

```docker build -t lamp-stack .```
Run docker images to verify that the image was created correctly.

```docker images --filter reference=lamp-stack```
Output:

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
lamp-stack         latest              e9ffedc8c286        4 minutes ago       258MB

Run the newly built image. The -p 80:80 option maps the exposed port 80 on the container to port 80 on the host system. For more information about docker run, go to the Docker run reference.

docker run -a stdin -a stdout -i -t lamp-stack /bin/bash


Open a browser and point to the server that is running Docker and hosting your container.
If you are running Docker locally, point your browser to http://localhost/.

If you are using docker-machine on a Windows or Mac computer, find the IP address of the VirtualBox VM that is hosting Docker with the docker-machine ip command, substituting machine-name with the name of the docker machine you are using.

docker-machine ip machine-name
You should see a web page.

Stop the Docker container by typing exit.

(Optional) Push your image to Amazon Elastic Container Registry
Amazon ECR is a managed AWS Docker registry service. Customers can use the familiar Docker CLI to push, pull, and manage images. For Amazon ECR product details, featured customer case studies, and FAQs, see the Amazon Elastic Container Registry product detail pages.

Note

This section requires the AWS CLI. If you do not have the AWS CLI installed on your system, see Installing the AWS Command Line Interface in the AWS Command Line Interface User Guide.

To tag your image and push it to Amazon ECR

Create an Amazon ECR repository to store your lamp-stack image. Note the repositoryUri in the output.

aws ecr create-repository --repository-name lamp-stack
Output:

{
    "repository": {
        "registryId": "aws_account_id",
        "repositoryName": "lamp-stack",
        "repositoryArn": "arn:aws:ecr:us-east-1:aws_account_id:repository/lamp-stack",
        "createdAt": 1505337806.0,
        "repositoryUri": "aws_account_id.dkr.ecr.us-east-1.amazonaws.com/lamp-stack"
    }
}
Tag the lamp-stack image with the repositoryUri value from the previous step.

docker tag lamp-stack aws_account_id.dkr.ecr.us-east-1.amazonaws.com/lamp-stack
Run the aws ecr get-login --no-include-email command to get the docker login authentication command string for your registry.

Note

The get-login command is available in the AWS CLI starting with version 1.9.15; however, we recommend version 1.11.91 or later for recent versions of Docker (17.06 or later). You can check your AWS CLI version with the aws --version command. If you are using Docker version 17.06 or later, include the --no-include-email option after get-login. If you receive an Unknown options: --no-include-email error, install the latest version of the AWS CLI. For more information, see Installing the AWS Command Line Interface in the AWS Command Line Interface User Guide.

aws ecr get-login --no-include-email
Run the docker login command that was returned in the previous step. This command provides an authorization token that is valid for 12 hours.

Important

When you execute this docker login command, the command string can be visible to other users on your system in a process list (ps -e) display. Because the docker login command contains authentication credentials, there is a risk that other users on your system could view them this way and use them to gain push and pull access to your repositories. If you are not on a secure system, you should consider this risk and log in interactively by omitting the -p password option, and then entering the password when prompted.

Push the image to Amazon ECR with the repositoryUri value from the earlier step.

docker push aws_account_id.dkr.ecr.us-east-1.amazonaws.com/lamp-stack
Next Steps
After the image push is finished, you can use your image in your Amazon ECS task definitions, which you can use to run tasks with.

Note

This section requires the AWS CLI. If you do not have the AWS CLI installed on your system, see Installing the AWS Command Line Interface in the AWS Command Line Interface User Guide.

To register a task definition with the lamp-stack image

Create a file called lamp-stack-task-def.json with the following contents, substituting the repositoryUri from the previous section for the image field.

{
    "family": "lamp-stack",
    "containerDefinitions": [
        {
            "name": "lamp-stack",
            "image": "aws_account_id.dkr.ecr.us-east-1.amazonaws.com/lamp-stack",
            "cpu": 10,
            "memory": 500,
            "portMappings": [
                {
                    "containerPort": 80,
                    "hostPort": 80
                }
            ],
            "entryPoint": [
                "/usr/sbin/apache2",
                "-D",
                "FOREGROUND"
            ],
            "essential": true
        }
    ]
}
Register a task definition with the lamp-stack-task-def.json file.

aws ecs register-task-definition --cli-input-json file://lamp-stack-task-def.json
The task definition is registered in the lamp-stack family as defined in the JSON file.

To run a task with the lamp-stack task definition

Important

Before you can run tasks in Amazon ECS, you need to launch container instances into a default cluster. For more information about how to set up and launch container instances, see Setting Up with Amazon ECS and Getting Started with Amazon ECS using Fargate.

Use the following AWS CLI command to run a task with the lamp-stack task definition.

aws ecs run-task --task-definition lamp-stack
