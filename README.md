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

## Step 6 >>Display GitLab login URL

```
### Run the below mentioned commend, replcae the IP address with your server IP

echo "Access GitLab at: http://$(curl -s http://169.254.169.254/latest/meta-data/public-ipv4)/" 
```
