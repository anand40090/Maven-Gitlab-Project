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

# In this lab we would be using self hosted Gitlab server on docker engine

To install a self-hosted GitLab instance, you can follow these steps. This guide covers the installation on a server running Ubuntu (as it's one of the most common environments for GitLab hosting), but the steps are similar for other Linux distributions.

## Infra Prerequisites:

1. A server running Ubuntu (or a similar Linux distribution).
   
2. Root or sudo access to the server.
   
3. A domain name (optional, but recommended for production).
   
4. At least 4 GB of RAM (8 GB or more is recommended for larger installations).
   
5. A reliable internet connection to download the necessary packages.

6. Docker engine 

### Step: 1 >> Create EC2 instance / VM and install the prerequsites 

Once you create Ubuntu EC2 / VM instance, install the prerequsites - 

```
## System update 
sudo apt upgrade -y

## Install OpenJDK-11 
sudo apt install openjdk-11-jdk -y 

## Install docer engine
sudo apt install docker.io -y

## Run the sonarqube docker container
docker run -d -p 9000:9000 sonarqube

```

### Step 2 >> Pull gitlab community edition docker image and create container

This command will:

1. Start a GitLab container in detached mode.
   
2. Configure the container with the hostname gitlab.example.com.
   
3. Map ports 80 (HTTP) and 443 (HTTPS) from the host to the container.
   
4. Set the container's name to gitlab.
   
5. Ensure the container restarts automatically if it fails or the Docker daemon is restarted.
   
6. Mount important volumes to the container for configuration files, logs, and persistent data, stored on the host under /srv/gitlab/config, /srv/gitlab/logs, and /srv/gitlab/data.
   
7. Use the latest GitLab Community Edition image from Docker Hub.

```
## Run below mentioned c ommand to pull the docker image and run the docker container for Gitlab Community edition

docker run --detach \
  --hostname gitlab.example.com \
  --publish 443:443 --publish 80:80  \
  --name gitlab \
  --restart always \
  --volume /srv/gitlab/config:/etc/gitlab \
  --volume /srv/gitlab/logs:/var/log/gitlab \
  --volume /srv/gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ce:latest


```

### Step 2 >> Pull gitlab community edition docker image and create container

This command starts a new GitLab Runner container with the following properties:

1. The container runs in detached mode (-d), so it operates in the background.
   
2. The container is named gitlab-runner for easy reference.

3. It uses the host’s network (--network host), meaning it shares the host's networking stack (useful for certain types of network communication).

4. The container’s configuration files are stored persistently on the host in /srv/gitlab-runner/config (using --volume), so even if the container is removed, the configuration is preserved.

5. The container is created using the latest version of the official gitlab/gitlab-runner Docker image.

```
docker exec -it gitlab-runner gitlab-runner register

```

#### You'll need the following information to register the runner:

1. GitLab URL: The address of your GitLab instance (e.g., http://gitlab.example.com). >> Use http://localhost:80

2. GitLab registration token: Obtain it from your GitLab project under Settings > CI/CD > Runners.

3. Description: The name/description of the runner.

4. Tags: Optional tags to associate with the runner.

5. Executor: Choose an executor (e.g., docker).

## Step 3 >> Verify that the containers are running

```
sudo docker ps -a
```

## Step 4 >> Wait for GitLab to initialize (may take a few minutes)

```
# Run this command to adjust sleep time of docker conatiner if needed

echo "Waiting for GitLab to initialize..." sleep 60
```

## Step 5 >> Retrieve GitLab root user password

```
## Run the below mentioned command to retrive the Girlab eoot user password

if [ -f "/srv/gitlab/config/initial_root_password" ]; then
    echo "GitLab Root Login Credentials:"
    echo "Username: root"
    echo "Password: $(sudo cat /srv/gitlab/config/initial_root_password | grep Password | awk '{print $2}')"
else
    echo "Password file not found! Resetting root password..."
    sudo docker exec -it gitlab gitlab-rails console <<EOF
user = User.find_by_username('root')
user.password = 'NewSecurePassword'
user.password_confirmation = 'NewSecurePassword'
user.save!
exit
EOF
    echo "Password reset to: NewSecurePassword"
fi

```

## Step 6 >> Display GitLab login URL

```
### Run the below mentioned commend, replcae the IP address with your server IP

echo "Access GitLab at: http://$(curl -s http://169.254.169.254/latest/meta-data/public-ipv4)/" 
```

## Clone the Gitlab project repository 

Once the Ec2 instance is ready to work as a gitlab runner worker,

then clone the gitlab project with your local system from where you need to upload your project data for CI-CD piple build.

Copy the gitlab project url >> Got to Gitlab project >> Find the Clone with HTTPS option >> copy the URL and clone with your EC2 instance or local system from where you want to upload the project data.

```


To clone the gitlab repository >> git clone https://gitlab.com/anand40090/springboot.git
It will craete the springboot folder in your system, save your project data into it and keep it ready to be uploaded on gitlab reposiroty.

To add your project data to commit on gitlab reposiorty >> go to project springboot project folder >> git add *

To commit the project data on gitlab repository >> git commit -m '1st commit'

To push the data to gitlab repository >> git push >> input username password when it will prompt for authentication 
 

```

## Configure sonarqube to intigrate with Gitlab

1. At very bigining we have already downloaded the sonarqube docker image and spined the container of port 9000,
   by running commnd "docker run -d -p 9000:9000 sonarqube"
   
3. Run docker ps -a coommand on the Ubuntu system to check the docker container status of Sonarqube
   
4. Use the EC2 Ubuntu instance ip address:9000 in browser to access the sonarqube dashboard

5. Default username and password for sonarqube is admin / admin

6. Generate the token from sonarqube for integration with gitlab >> Login sonarqube >> Go to My Account >> Security >> Generate Token

7. Save the generated token to be hardcode in Gitlab variable

## Intigrate Sonarqube with Gitlab

1. Login to gitlab account >> go to project >> go to setting >> CI-CD >> Find for variables >> Expand

2. Create two variable - 1] SONAR_TOKEN >> Input the token generated from sonqrqube 2] SONAR_HOST_URL >> Input your sonarqube URL (http://13.126.187.216:9000/)

## Create AWS ECR and IAM User with Secret and Access key to push to Docker Image on ECR

1. Create ECR reposiroty >> Go to AWS Dashboard >> Go to ECR >> Create Reposiorty
   
2. Create AWS User to authenticate the ECR and push docker image to ECR >> Go to AWS Dashboard >> IAM User >> Create User >> Download the Secret Key and Access Key of the user
   
3. Create Variable in Gitlab to call during CI-CD pipeline >> Go to Gitlab account >> go to project >> go to setting >> CI-CD >> Find for variables >> Expand

4. Create 1] AWS_Access_KEY_ID 2] AWS_Default_Region 3] AWS_Secret_access_key

## Create Variables in the Gitlab

Before creating the .gitlab-ci.yml, add the following variables in GitLab project settings (Settings > CI / CD > Variables):

1. SONARQUBE_URL: The URL of your SonarQube instance (e.g., http://sonarqube.local).

2. SONARQUBE_TOKEN: Your SonarQube token.

3. DOCKER_REGISTRY: The Docker registry (e.g., docker.io).

4. DOCKER_USERNAME: Your Docker Hub username.

5. DOCKER_PASSWORD: Your Docker Hub password or access token.

6. AWS_ACCESS_KEY_ID: AWS Access Key ID for EKS access.

7. AWS_SECRET_ACCESS_KEY: AWS Secret Access Key for EKS access.

8. AWS_DEFAULT_REGION: AWS region where your EKS cluster is hosted (e.g., us-east-1).

9. ECR_REPO_NAME: The ECR repository name where the image will be pushed (optional if pushing to DockerHub).

10. KUBECONFIG: The kubeconfig file to access the EKS cluster (can be configured using aws eks update-kubeconfig).


## Create .gitlab-ci.yml file in the gitlab project reposiroty to run the CI-CD pipeline

Got to gitlab project >> create the .gitlab-ci-.yml file

Once this file is created and commited over the gitlab project, it will trigger the CI-CD pipeline and start the stages as mentioned in the file.

```
stages:
  - build
  - sonar_scan
  - docker_build
  - docker_push
  - deploy

before_script:
  - echo "Setting up Docker and AWS CLI"
  - apk add --no-cache curl git bash python3 py3-pip
  - pip3 install awscli
  - aws --version
  - echo "$AWS_ACCESS_KEY_ID" | aws configure set aws_access_key_id
  - echo "$AWS_SECRET_ACCESS_KEY" | aws configure set aws_secret_access_key
  - echo "$AWS_DEFAULT_REGION" | aws configure set region
  - echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin

# Build the Maven project
build_maven:
  image: maven:3.8.5-jdk-11  
  stage: build
  script:
    - echo "Building Maven project"
    - mvn clean install $MAVEN_CLI_OPTS

# Run SonarQube analysis
sonar_scan:
  stage: sonar_scan
  script:
    - echo "Running SonarQube scan"
    - mvn sonar:sonar -Dsonar.host.url=$SONARQUBE_URL -Dsonar.login=$SONARQUBE_TOKEN

# Build the Docker image
docker_build:
  stage: docker_build
  script:
    - echo "Building Docker image"
    - docker build -t $DOCKER_USERNAME/$DOCKER_IMAGE_NAME:$CI_COMMIT_REF_NAME .

# Push the Docker image to Docker Hub
docker_push:
  stage: docker_push
  script:
    - echo "Pushing Docker image to Docker Hub"
    - docker push $DOCKER_USERNAME/$DOCKER_IMAGE_NAME:$CI_COMMIT_REF_NAME
# Deploy the Docker image to EKS
deploy_to_eks:
  stage: deploy
  script:
    - echo "Setting up Kubernetes configuration for EKS"
    - aws eks update-kubeconfig --name $EKS_CLUSTER_NAME --region $AWS_DEFAULT_REGION
    - kubectl get svc
    - echo "Deploying Docker image to EKS"
    - kubectl set image deployment/$KUBERNETES_DEPLOYMENT_NAME $KUBERNETES_DEPLOYMENT_NAME=$DOCKER_USERNAME/$DOCKER_IMAGE_NAME:$CI_COMMIT_REF_NAME
    - kubectl rollout status deployment/$KUBERNETES_DEPLOYMENT_NAME
  only:
    - main  # Run this job only on the main branch (adjust if needed)    

```
