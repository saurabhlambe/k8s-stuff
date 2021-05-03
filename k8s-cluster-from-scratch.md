In this tutorial, I will set up a Kubernetes cluster on top of 3 nodes (1 control-plane and 2 worker nodes).

Reference:
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-runtime

# Add /etc/hosts entries on server and worker nodes respectively

```bash
[root@slambe-kube-control-plane ~]# cat /etc/hosts
172.26.64.112   slambe-kube-control-plane       slambe-kube-control-plane.novalocal     server-node	snode
172.26.64.54    slambe-worker-01                slambe-kube-worker-01.novalocal         worker-node-1	w1
172.26.64.75    slambe-worker-02                slambe-kube-worker-02.novalocal         worker-node-2	w2
```

# Disable SELinux
```bash
setenforce 0
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```

# Permanently disable SWAP memory
https://www.tecmint.com/disable-swap-partition-in-centos-ubuntu

# Load 'br_netfilter' module
```bash
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# Setup required sysctl params, these persist across reboots.
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
```

# Reboot

# Install container runtime

## Download and execute the [install_docker_centos.sh](https://gitlab.com/saurabhlambe/scripts/-/blob/master/install_docker_centos.sh) script

## Configure containerd
```bash
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
```

## Restart containerd
```bash
sudo systemctl restart containerd
```

# Disable SWAP on all nodes
```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

# Installing kubeadm, kubelet, and kubectl (all nodes)

## Create a kubernetes repo
```bash
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF
```

## Install and enable
```bash
sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
sudo systemctl enable --now kubelet
```

# Initialize Control Plane (control-plane node only)
```bash
sudo kubeadm init --pod-network-cidr 192.168.0.0/16
```
And then run the commands to create and initialize the HOME dir for kube config. Also copy the kubeadm join command.

# Verify if kubectl works
```bash
kubectl version
```

# Install and verify the Calico [network add-on](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#pod-network) and containers
```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
kubectl get pods -n kube-system
```

# On each of the worker nodes, execute the 'kubeadm join' command you got from the init command above
```bash
sudo kubeadm join ...
```

# On the control plane node, verify all nodes are connected
```bash
kubectl get nodes
```
