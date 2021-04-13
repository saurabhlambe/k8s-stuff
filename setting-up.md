1. Remove all existing docker packages:
```bash
# sudo yum remove docker \
>                   docker-client \
>                   docker-client-latest \
>                   docker-common \
>                   docker-latest \
>                   docker-latest-logrotate \
>                   docker-logrotate \
>                   docker-engine
```

2. Install prerequisites
```bash
yum install -y yum-utils
```
3. Create docker repo
```bash
yum-config-manager \
>     --add-repo \
>     https://download.docker.com/linux/centos/docker-ce.repo
```

4. Install docker
```bash
yum install docker-ce docker-ce-cli containerd.io
```

5. Initialize docker
```bash
systemctl enable docker
systemctl start docker
```

6. Switch to non-root user to initialize minikube
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

7. Add the $USER to docker group:
```bash
sudo usermod -aG docker saurabhlambe && newgrp docker
```

8. Start a cluster using the docker driver:
```bash
minikube start --driver=docker
```

9. If kubectl is installed, run this command to list all the pods running currently:
```bash
kubectl get po -A
```
