# Maven-Gitlab-Project
In this project we will build maven project on Gitlab. 
To build a Maven project, run a SonarQube scan, build a Docker image, push the image to Docker Hub, and finally deploy and run the image on an Amazon EKS (Elastic Kubernetes Service) cluster via a GitLab CI/CD pipeline, you can follow these steps.

_____

# Project Prerequisites

1 GitLab Project: You need a GitLab project where the Maven project code resides.

2 SonarQube: You need to have SonarQube set up and a SonarQube token (can be saved as GitLab CI/CD variables).

3 Docker Hub Account: Ensure you have Docker Hub credentials stored in GitLab CI/CD variables.

4 EKS Cluster: Ensure you have an Amazon EKS cluster up and running, and you can connect to it using kubectl.

5 AWS IAM: Ensure your GitLab CI runner has appropriate permissions to interact with AWS EKS (via IAM roles and policies).

___

# In this lab we would be using self hosted Gitlab server 

To install a self-hosted GitLab instance, you can follow these steps. This guide covers the installation on a server running Ubuntu (as it's one of the most common environments for GitLab hosting), but the steps are similar for other Linux distributions.

## Infra Prerequisites:

1. A server running Ubuntu (or a similar Linux distribution).
   
2. Root or sudo access to the server.
   
3. A domain name (optional, but recommended for production).
   
4. At least 4 GB of RAM (8 GB or more is recommended for larger installations).
   
5. A reliable internet connection to download the necessary packages.

