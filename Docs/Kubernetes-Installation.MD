# Kubernetes Cluster installation using kubeadm
Follow this documentation to set up a Kubernetes cluster on __CentOS 7__ Virtual machines.

This documentation guides you in setting up a cluster with one master node and two worker nodes.

Master: t2.medium
Worker Nodes: t2.micro

# Ports require for kubernetes master
```sh
6443
32750
10250
4443
443
8080
179 --> Should be allowed for Master and Worker nodes. 
```

## On both Kmaster and Kworker Nodes
Perform all the commands as root user unless otherwise specified

##### Install, enable and start docker service
Use the Docker repository to install docker.
> If you use docker from CentOS OS repository, the docker version might be old to work with Kubernetes v1.13.0 and above
```
```sh
yum install -y -q yum-utils device-mapper-persistent-data lvm2 > /dev/null 2>&1
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo > /dev/null 2>&1
yum install -y -q docker-ce >/dev/null 2>&1
```
```sh
systemctl enable docker
systemctl start docker
```
##### Disable SELinux
```sh
setenforce 0
sed -i --follow-symlinks 's/^SELINUX=enforcing/SELINUX=disabled/' /etc/sysconfig/selinux
```
##### Disable Firewall
```sh
systemctl disable firewalld
systemctl stop firewalld
```
##### Disable swap
```sh
sed -i '/swap/d' /etc/fstab
swapoff -a
```
##### Update sysctl settings for Kubernetes networking
```sh
cat >>/etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```
### Kubernetes Setup
##### Add yum repository
```sh
cat >>/etc/yum.repos.d/kubernetes.repo<<EOF
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
        https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```
##### yum update
yum update
yum repoinfo kubernetes  list
Failed to set locale, defaulting to C
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd
Repo-id      : kubernetes
Repo-name    : Kubernetes
Repo-status  : enabled
Repo-revision: 1626395017604613
Repo-updated : Thu Jun  3 11:03:33 51540376
Repo-pkgs    : 687
Repo-size    : 8.8 G
Repo-baseurl : https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
Repo-expire  : 21600 second(s) (last: Thu Jul 29 15:53:03 2021)
  Filter     : read-only:present
Repo-filename: /etc/yum.repos.d/kubernetes.repo

repolist: 687
[root@ip-172-31-74-146 ec2-user]# yum repo-pakgs kubernetes list
Failed to set locale, defaulting to C
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd
No such command: repo-pakgs. Please use /bin/yum --help
[root@ip-172-31-74-146 ec2-user]# yum repo-pkgs kubernetes list
Failed to set locale, defaulting to C
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd
Available Packages
cri-tools.x86_64                                                                                  1.13.0-0                                                                             kubernetes
kubeadm.x86_64                                                                                    1.21.3-0                                                                             kubernetes
kubectl.x86_64                                                                                    1.21.3-0                                                                             kubernetes
kubelet.x86_64                                                                                    1.21.3-0                                                                             kubernetes
kubernetes-cni.x86_64                                                                             0.8.7-0                                                                              kubernetes
rkt.x86_64                                                                                        1.27.0-1                                                                             kubernetes
[root@ip-172-31-74-146 ec2-user]# 

##### Install Kubernetes
```sh
yum install -y kubeadm-1.15.6-0.x86_64 kubelet-1.15.6-0.x86_64 kubectl-1.15.6-0.x86_64
yum install kubeadm.x86_64 kubectl.x86_64 kubelet.x86_64 -y 
```
##### Enable and Start kubelet service
```sh
systemctl enable kubelet
systemctl start kubelet
```
## On kmaster
##### Initialize Kubernetes Cluster
```sh
kubeadm init --apiserver-advertise-address=<MasterServerIP> --pod-network-cidr=192.168.0.0/16
```
##### Copy kube config
To be able to use kubectl command to connect and interact with the cluster, the user needs kube config file.

In my case, the user account is praveen
```sh
mkdir /home/praveen/.kube
cp /etc/kubernetes/admin.conf /home/praveen/.kube/config
chown -R praveen:praveen /home/praveen/.kube
```
##### Deploy Calico network
This has to be done as the user in the above step (in my case it is __praveen__)
```sh
kubectl create -f https://docs.projectcalico.org/v3.9/manifests/calico.yaml
```

##### Cluster join command
```sh
kubeadm token create --print-join-command
```
## On Kworker
##### Join the cluster
Use the output from __kubeadm token create__ command in previous step from the master server and run here.

## Verifying the cluster
##### Get Nodes status
```sh
kubectl get nodes
```
##### Get component status
```sh
kubectl get cs
```
