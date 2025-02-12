# Installing Kubernetes on Ubuntu 24.04 (Noble)
## Purpose
This guide will help you install and set up Kubernetes on Ubuntu 24.04.
## Learning Goals
At the end of this guide, you will:
- Install Kubernetes on Ubuntu 24.04.
- Understand Kubernetes setup steps.
- Set up a Kubernetes cluster.
- Deploy a simple server on Kubernetes.
## Outline
1. Setting Up Kubernetes on All Nodes
2. Configuring the Master Node
3. Adding Worker Nodes
4. Deploying an Nginx Server
---
## 1. Setting Up Kubernetes on All Nodes
- We will use two Ubuntu 24.04 machines:
  - One Master Node
  - One Worker Node
- Minimum requirements:
  - **2 CPU Cores**
  - **2 GB RAM**
- Update your system:
  
  ```bash
  sudo apt update -y && sudo apt upgrade -y
  ```
- Install necessary packages:
  ```bash
  sudo apt install -y apt-transport-https ca-certificates curl
  ```
- Add the Kubernetes repository:
  
  ```bash
    sudo mkdir -p /etc/apt/keyrings
    curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
    echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
  ```
- Install Kubernetes tools:
  
  ```bash
  sudo apt update -y
  sudo apt install -y kubelet kubeadm kubectl
  sudo apt-mark hold kubelet kubeadm kubectl
  ```
- Install Docker:
  
  ```bash
  sudo apt install -y docker.io
  sudo systemctl enable docker --now
  sudo usermod -aG docker $USER
  ```
- Configure networking:
  
  ```bash
  cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
  net.bridge.bridge-nf-call-iptables = 1
  net.bridge.bridge-nf-call-ip6tables = 1
  net.ipv4.ip_forward = 1
  EOF
  sudo sysctl --system
  ```
- Configure container runtime:
  
  ```bash
  sudo mkdir /etc/containerd
  containerd config default | sudo tee /etc/containerd/config.toml
  sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
  sudo systemctl restart containerd
  ```
---
## 2. Configuring the Master Node
- Initialize the cluster (replace `<your-master-ip>` with your private IP):
  
  ```bash
  sudo kubeadm init --apiserver-advertise-address=<your-master-ip> --pod-network-cidr=10.244.0.0/16
  ```
- Set up the kubeconfig file:
  
  ```bash
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
  ```
- Install Flannel network:
  
  ```bash
  kubectl apply -f https://github.com/coreos/flannel/raw/master/Documentation/kube-flannel.yml
  ```
- Verify setup:
  
  ```bash
  kubectl get nodes
  ```
---
## 3. Adding Worker Nodes
- On the Master Node, get the join command:
  
  ```bash
  kubeadm token create --print-join-command
  ```
- Copy the command and run it on the Worker Node:
  
  ```bash
  sudo kubeadm join <your-master-ip>:6443 --token <your-token> --discovery-token-ca-cert-hash sha256:<your-hash>
  ```
- Check the cluster from the Master Node:
  
  ```bash
  kubectl get nodes
  ```
---
## 4. Deploying an Nginx Server
- Deploy an Nginx pod:
  
  ```bash
  kubectl run nginx-server --image=nginx --port=80
  ```
- Expose the Nginx pod:
  
  ```bash
  kubectl expose pod nginx-server --port=80 --type=NodePort
  ```
- Get the NodePort:
  
  ```bash
  kubectl get services
  ```
- Access the Nginx server using the worker node’s IP and NodePort.
---
## References
- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
- https://kubernetes.io/docs/concepts/cluster-administration/addons/