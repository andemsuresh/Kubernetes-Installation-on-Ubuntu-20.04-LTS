# Kubernetes-Installation-on-Ubuntu-20.04-LTS
This repo is for Installation of latest version of docker &amp; k8s cluster on Ubuntu 20.04 LTS

#### Please follow from step 1 to step 3 on every node, step 4 & step 5 only on master node and step 6 only on worker nodes.

### **Step 1: Update & Upgrade The Servers**

> Once the servers are ready, update them.
```
> sudo apt update
```
> After updating,upgrade the servers
```
> sudo apt -y upgrade
```
### **Step 2: Install Docker, kubelet, kubeadm and kubectl**
> Add the GPG key for Docker
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```
> Add docker Repo
```
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```
> Add the GPG key for kubernetes
```
> curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```
> Add the kubernetes repository
```
cat << EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
```
> Update the repository
```
sudo apt update
```
> Install Docker and Kubernetes packages.
```

# if you want to know the latest kubelet , kubeadm and kubectl versions run below commands
```
apt update
```
> apt-cache madison kubeadm
```

# find the latest 1.24 version in the list
# it should look like 1.24.x-00, where x is the latest patch
```

> sudo apt-get install -y docker-ce=5:20.10.7~3-0~ubuntu-$(lsb_release -cs) kubelet=1.24.4-00 kubeadm=1.24.4-00 kubectl=1.24.4-00
```
To hold the versions so that the versions will not get accidently upgraded.
```
> sudo apt-mark hold docker-ce kubelet kubeadm kubectl
```
> Start and enable Docker Services
```
sudo systemctl restart docker
```
sudo systemctl enable docker
```
> Confirm installation by checking the version of kubectl
```
> kubectl version --client && kubeadm version
```

### **Step 3: Disable Swap**

> Turn off swap
```
sudo swapoff -a
```
> Enable the iptables bridge(Set a value in the sysctl file , to allow proper network settings for Kubernetes on all the servers)
```
echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf
```
> Reload sysctl
```
sudo sysctl -p
```

# **Step 4 & Step 5 installation has to be done only on Master Node.**

### **Step 4: Initialize master node**

#### For Flannel Network

```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

#### For calico Network

```
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```
# if you receive the below error after running the "kubeadm init" command
#[init] Using Kubernetes version: v1.24.4
#[preflight] Running pre-flight checks
#error execution phase preflight: [preflight] Some fatal errors occurred:
 #       [ERROR CRI]: container runtime is not running: output: E0820 19:06:35.025990    6022 remote_runtime.go:925] "Status from runtime service failed" #err="rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.RuntimeService"
#time="2022-08-20T19:06:35Z" level=fatal msg="getting status of runtime: rpc error: code = Unimplemented desc = unknown service #runtime.v1alpha2.RuntimeService"
#, error: exit status 1
#[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
#To see the stack trace of this error execute with --v=5 or higher

run the following commands

> sudo rm -rf /etc/containerd/config.toml
> sudo systemctl restart containerd

#after the above commands run again  "kubeadm init" command

> To start using the cluster with current user.
```
mkdir -p $HOME/.kube
```
```
sudo cp -f /etc/kubernetes/admin.conf $HOME/.kube/config
```
```
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
To check the Check cluster status:
```
kubectl cluster-info
```
### **Step 5: Install network plugin on Master**

> To Set with flannel networking
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```
> To Set with Calico networking
```
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```
### **Step 6: Add worker nodes**

> Joining the node to the cluster:
```
# use sudo before kubeadm join
#Please do not copy this command, this is just an example
sudo kubeadm join $controller_private_ip:6443 --token $token --discovery-token-ca-cert-hash $hash
```
## TIP
> If the joining code is lost, it can be retrieved using below command
```
kubeadm token create --print-join-command
```

##### Now please login to the master node and check whether the worker nodes has joined the cluster or not.

> Using below command
```
kubectl get nodes
```
##### Similarly follow the same steps for creating the other worker node or for remaining nodes as well.
