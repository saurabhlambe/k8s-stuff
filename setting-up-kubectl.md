This tutorial is a guidebook on installing kubectl on CentOS 8

# Install Docker
Run this [script](https://gitlab.com/saurabhlambe/scripts/-/blob/74d3f23a26ab78b0a8defbe3f52377285de912d6/install_docker_centos.sh) to install Docker.

OR

Follow the official Docker [documentation](https://docs.docker.com/engine/install/centos/).

# Install and set up minikube

## 1. Switch to non-root user to initialize minikube
```bash
su $USER
```
Here $USER is saurabhlambe

>The below command will throw an error since $USER is not part of the docker group:
```bash
minikube start --driver=docker
 minikube v1.19.0 on Centos 8.3.2011
 Using the docker driver based on user configuration
 Exiting due to PROVIDER_DOCKER_NEWGRP: "docker version --format -" exit status 1: Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.24/version: dial unix /var/run/docker.sock: connect: permission denied
 Suggestion: Add your user to the 'docker' group: 'sudo usermod -aG docker $USER && newgrp docker'
 Documentation: https://docs.docker.com/engine/install/linux-postinstall/
```

## 2. Add the $USER to docker group:
```bash
sudo usermod -aG docker saurabhlambe && newgrp docker
```

## 3. Start a cluster using the docker driver:
```bash
minikube start --driver=docker
```

# If kubectl is installed, run this command to list all the pods running currently:
```bash
kubectl get po -A
```
