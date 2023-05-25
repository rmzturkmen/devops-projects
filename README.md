# DevOps Projects

## List of projects within AWS workshop as follows;

[001-dockerization-bookstore-api-on-python-flask-mysql](001-dockerization-bookstore-api-on-python-flask-mysql/README.md)
Bookstore Web API Application aims to create a bookstore web service using Docker to give students the understanding to dockerization of an application. The application code is to be deployed as a RESTful web service with Flask using Dockerfile and Docker Compose on AWS Elastic Compute Cloud (EC2) Instance using AWS Cloudformation Service.

[002-jenkins-pipeline-for-phonebook-app-on-docker-swarm](002-jenkins-pipeline-for-phonebook-app-on-docker-swarm/README.md)
This project aims to create a Jenkins pipeline to deploy the Phonebook Application web application with Docker Swarm on Elastic Compute Cloud (EC2) Instances by pulling the app images from the AWS Elastic Container Registry (ECR) repository.

[003-Kubernetes-Microservice-Phonebook](003-Kubernetes-Microservice-Phonebook/README.md)
Phonebook Microservice Web Application aims to create a web application with MySQL Database using Docker and Kubernetes to give students the understanding of Microservice architecture. In this application, we have a frontend service and a backend service to interact with database service. Each service will be managed by a Kubernetes deployment. The backend service will be a gateway for the application and it will serve the necessary web pages for create, delete and update operations while the frontend service will serve a search page in order to conduct read operations. To preserve the data in the database, persistent volume and persistent volume claim concepts should be adopted.

[004-AWS-codepipeline-terraform-cicd](004-aws-codepipeline-terraform-cicd/README.md)
Terraform is an infrastructure-as-code (IaC) tool that helps you create, update, and version your infrastructure in a secure and repeatable manner.

The scope of this pattern is to provide a guide and ready to use terraform configurations to setup validation pipelines with end-to-end tests based on AWS CodePipeline, AWS CodeBuild, AWS CodeCommit and Terraform.