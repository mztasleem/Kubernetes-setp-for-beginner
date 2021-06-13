# Kubernetes-setp-for-beginner
This repository help to learn students all activities required to setup first kubernetes. Starting from creating master-child nodes till deploying/running applications.


Step 1: Create a Debian VM (Name it Master)
Step 2: Install Docker Engine on master

sudo apt-get update

sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release


curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian \
 $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io

Step 3: Installing kubelet, kubectl, kubeadm
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl

sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

Step 4: Creating worker nodes
Create machine image of  node.

Create VM Instance (name it worker1) using master machine image.

Create VM Instance (name it worker2) using master machine image.
Step 5: Validate kubectl, kubelet on master and worker nodes
Open CLI for all three VMs (1 master and 2 worker nodes) using ssh

And execute below commands one by one
Kubectl version
Kubelet --version


Step 6: Creating cluster on master node
sudo su
export MASTER_IP=<Internal IP adress>
kubeadm init --apiserver-advertise-address=${MASTER_IP} --pod-network-cidr=10.244.0.0/16


It will take few minutes and as a result of this command, you will see below at the end of output (Note them down in notepad):

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubeadm join 10.128.0.17:6443 --token u2cr90.abi0e10018m25rn3 \ 
--discovery-token-ca-cert-hash sha256:d515a3b4e74d1e6e748f101b6accf5fe976678f01628135b2463b8eee7e6c285

kubeadm join 10.128.0.17:6443 --token bzx747.o7gxwamlaabpop4g \
        --discovery-token-ca-cert-hash sha256:b760b79d186fd6409d42a0cd2d067829aca75af471586fc3835dd4916b8a2a17


Step 7: Setting up config file on master  node
Execute below command to set config file
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubectl get nodes
NAME STATUS ROLES AGE VERSION
master Ready control-plane,master 74m v1.21.1

Step 8: Joining worker nodes with master node
Go to ssh CLI prompt of worker nodes and execute the join the command copied above(highlighted in red).

As a result output, below will be shown:
"This node has joined the cluster:"

kubectl get nodes
NAME STATUS ROLES AGE VERSION
master Ready control-plane,master 74m v1.21.1
worker1 Ready <none> 59m v1.21.1
worker2 Ready <none> 59m v1.21.1


Step 9: Setting up CNI on Master node
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
Step 10 : Installing Webserver on Cluster
kubectl create deployment nginx --image nginx
kubectl expose deployment nginx --type NodePort --port 80
kubectl get services

Result will be something like this:
NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
kubernetes ClusterIP 10.96.0.1 <none> 443/TCP 41m
nginx NodePort 10.106.27.241 <none> 80:31014/TCP 13m


Step 11 : Test your web application running on Cluster
Copy external ip of master node and 5 digit port number from your screen same as highlighted above in red font.
Open a new browser and paste as below:
<External ip address of machine>:<port number>



Additional Info: Cleaning master and worker nodes in case of reset
sudo su
kubeadm reset
iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X
rm -rf $HOME/.kube/config
swapoff -a

Reboot the master/worker node
