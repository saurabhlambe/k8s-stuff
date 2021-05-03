In this tutorial, I will set up a Kubernetes cluster on top of 3 nodes (1 control-plane and 2 worker nodes).

*Note: Execute all steps except 9, 10, and 11 on ALL nodes. Steps 9-11 are to be executed only on the ControlPlane node.*

1. Edit /etc/hosts so that it contains the hostname entries of all the nodes
```bash
cat /etc/hosts

172.26.64.114	slambe-kube-control-plane	slambe-kube-control-plane.novalocal	server-node	snode
172.26.64.54	slambe-worker-01		slambe-kube-worker-01.novalocal		worker-node-1	w1
172.26.64.105	slambe-worker-02		slambe-kube-worker-02.novalocal		worker-node-2	w2
```

2. Disable SELINUX and SWAP memory
```bash
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

3. Install [Docker](https://gitlab.com/saurabhlambe/scripts/-/blob/master/install_docker_centos.sh).

4. Configure Docker daemon
```bash
sudo mkdir /etc/docker
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
```

5. Restart docker
```bash
sudo systemctl enable docker
sudo systemctl daemon-reload
sudo systemctl restart docker
```

6. Letting iptables see bridged traffic
```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```

7. Configure Kubernetes repository
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

8. Install kubeadm, kubelet, kubectl (all machines
```bash
sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
sudo systemctl enable --now kubelet
```

9. Initialize the control-plane node
```bash
sudo kubeadm init
```
Copy the _kubeadm join_ command. We will need it later while configuring the worker nodes.
To make kubectl work for your non-root user, run these commands, which are also part of the kubeadm init output:
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

10. Install and verify a Pod network add-on
```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
kubectl get pods -A
```

11. Generate the _kubeadm join_ command (OPTIONAL)
```bash
kubeadm token create --print-join-command
```

12. To be run ONLY on worker nodes:
```bash
sudo kubeadm join 172.26.64.114:6443 --token h630p9.5apygti9ims4m8ez \
> --discovery-token-ca-cert-hash sha256:dc378cd7efade62d73c121967420c87c8da705899ad4b02b773cdb50f6ddb724
```
