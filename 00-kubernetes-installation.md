# Kubenetes installation on master and worker nodes using kubeadm

https://kubernetes.io/docs/setup/

## Installation on centos/rhel nodes

### perform below steps on both master and worker nodes

#### Step1: System pre-req
```sh
sudo su
sudo setenforce 0
sudo swapoff --all
sudo iptables --flush and iptables -tnat --flush
sudo systemctl stop firewalld && sudo systemctl disable firewalld
```

#### Step2: Install docker
```sh
sudo su
sudo yum install -y yum-utils
sudo yum-config-manager \
     --add-repo \
     https://download.docker.com/linux/centos/docker-ce.repo
    
sudo yum install docker-ce docker-ce-cli containerd.io
sudo systemctl enable docker && sudo systemctl start docker
```

#### Step 3: Kubernetes repo configuration
```sh
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=0
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg   
EOF
```

#### Step 4: Kernel Parameter Configuration
```sh
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```

#### Step 5: Install kubernetes binaries 
```sh
sudo yum install -y kubelet kubeadm kubectl 
```


#### Step 6: On master node initialize the cluster using kubeadm
```sh
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```
```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```    

#### Step 7: Install Network Addon (flannel):
```sh
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

#### Step 8: On Worker nodes, join the worker nodes to cluster
```sh
sudo su
<kubeadm join command copies from master node>
```

#### Step 9: Wait for nodes and system pod to be ready
```sh
    kubectl get nodes
    kubectl get pods --all-namespaces
```

#### Optional 
If you want your master to run user-defined pods as well, execute below
```sh
kubectl taint nodes --all node-role.kubernetes.io/master-
```
