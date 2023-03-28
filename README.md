#Prerequisite:

3 - Ubuntu Serves  (Choose Ubuntu Version 20.04 AMI)

1 - Manager  (4GB RAM , 2 Core) t2.medium

2 - Workers  (1 GB, 1 Core)     t2.micro


Note: Open Required Ports In AWS Security Groups. For now we will open All trafic.

#+++++++++++++++++COMMON FOR MASTER & Worker START++++++++++++++++++++++

# First, login as ‘root’ user because the following set of commands need to be executed with ‘sudo’ permissions.

$ sudo su -


# Install And Enable Docker

sudo apt-get update
apt-get install docker.io -y
systemctl enable docker
systemctl status docker
systemctl start docker

# Turn Off Swap Space

swapoff -a
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab


# Install Required packages and apt keys.


apt-get install -y apt-transport-https

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add

cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

apt-get update -y


#Install kubeadm, Kubelet And Kubectl
Note: Kubectl only in master

apt-get install -y kubelet kubeadm kubectl kubernetes-cni

# Enable and start kubelet service

systemctl daemon-reload
systemctl start kubelet
systemctl enable kubelet.service

==========COMMON FOR MASTER & SLAVES END=====



=========== Commands for only Master Node Start====================
# Steps Only For Kubernetes Master

# Switch to the root user.

sudo su -

# Initialize Kubernates master by executing below commond.

$ kubeadm init
$ kubeadm init --apiserver-advertise-address=192.168.56.2 --pod-network-cidr=10.244.0.0/16

#exit root user & exeucte as normal user

$ exit

$ mkdir -p $HOME/.kube

$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

$ sudo chown $(id -u):$(id -g) $HOME/.kube/config


# To verify, if kubectl is working or not, run the following command.

$ kubectl get pods -o wide --all-namespaces

#You will notice from the previous command, that all the pods are running except one: ‘kube-dns’. For resolving this we will install a # pod network. To install the weave pod network, run the following command:

$ sudo kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

$ kubectl get nodes

$ kubectl get pods --all-namespaces


# Get token

$ kubeadm token create --print-join-command

kubeadm join 172.31.30.65:6443 --token ts22qz.zxechhnr6v43kqf1 --discovery-token-ca-cert-hash sha256:77fd61885dd51a0754c9bb4a7d16968d8fb6109ec6210be1adeed232f0a482ae

=========In Master Node End====================


Add Worker Machines to Kubernates Master
=========================================

Copy kubeadm join token from master node and execute in Worker Nodes to join to cluster

Note: kubectl commonds has to be executed in master machine.

# Check Nodes

$ kubectl get nodes