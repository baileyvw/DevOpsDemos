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

yum repoinfo kubernetes list

yum repo-pkgs kubernetes list 

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
## if you get an error message on systemctl start or status on kubelet you will need to edit the docker systemd file
## Aug  3 16:37:52 ip-172-31-4-255 kubelet: E0803 16:37:52.528123   22962 server.go:204] "Failed to load kubelet config file" err="failed to load Kubelet config file /var/lib/kubelet/config.yaml, error failed to read kubelet config file \"/var/lib/kubelet/config.yaml\", error: open /var/lib/kubelet/config.yaml: no such file or directory" path="/var/lib/kubelet/config.yaml"
## Aug  3 16:37:52 ip-172-31-4-255 systemd: kubelet.service: main process exited, code=exited, status=1/FAILURE

```
vi /etc/docker/daemon.json file and change " native.cgrougdirver=systemd"
or 
vi /etc/systemd/system/multi-user.target.wants/docker.service uncomment "TasksMax=infinity" to allow system to work.
# Uncomment TasksMax if your systemd version supports it.
# Only systemd 226 and above support this version.
TasksMax=infinity
TimeoutStartSec=0
save file
systemctl stop docker
systemctl status docker
systemctl daemon-reload
systemctl start docker

```
##### Install Kubernetes
```sh
yum install -y kubeadm-1.15.6-0.x86_64 kubelet-1.15.6-0.x86_64 kubectl-1.15.6-0.x86_64
yum install kubeadm.x86_64 kubectl.x86_64 kubelet.x86_64 -y 
```
##### Enable and Start kubelet service
```sh
systemctl enable kubelet
systemctl start kubelet
systemctl status kubelet
there should be no more errors

systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; vendor preset: disabled)
  Drop-In: /usr/lib/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
   Active: active (running) since Wed 2021-08-04 11:23:41 UTC; 17min ago
     Docs: https://kubernetes.io/docs/
 Main PID: 4092 (kubelet)
    Tasks: 14
   Memory: 120.5M
   CGroup: /system.slice/kubelet.service
           └─4092 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --network-plu...

Aug 04 11:40:54 ip-172-31-4-255.ec2.internal kubelet[4092]: I0804 11:40:54.703736    4092 cni.go:239] "Unable to update cni config" err="no networks found in /etc/cni/net.d"
Aug 04 11:40:57 ip-172-31-4-255.ec2.internal kubelet[4092]: E0804 11:40:57.372756    4092 kubelet.go:2211] "Container runtime network not ready" networkReady="NetworkReady=false r...nitialized"
Aug 04 11:40:59 ip-172-31-4-255.ec2.internal kubelet[4092]: I0804 11:40:59.703970    4092 cni.go:239] "Unable to update cni config" err="no networks found in /etc/cni/net.d"
Aug 04 11:41:02 ip-172-31-4-255.ec2.internal kubelet[4092]: E0804 11:41:02.384182    4092 kubelet.go:2211] "Container runtime network not ready" networkReady="NetworkReady=false r...nitialized"
Aug 04 11:41:04 ip-172-31-4-255.ec2.internal kubelet[4092]: I0804 11:41:04.705213    4092 cni.go:239] "Unable to update cni config" err="no networks found in /etc/cni/net.d"
Aug 04 11:41:07 ip-172-31-4-255.ec2.internal kubelet[4092]: E0804 11:41:07.395256    4092 kubelet.go:2211] "Container runtime network not ready" networkReady="NetworkReady=false r...nitialized"
Aug 04 11:41:09 ip-172-31-4-255.ec2.internal kubelet[4092]: I0804 11:41:09.705434    4092 cni.go:239] "Unable to update cni config" err="no networks found in /etc/cni/net.d"
Aug 04 11:41:12 ip-172-31-4-255.ec2.internal kubelet[4092]: E0804 11:41:12.406014    4092 kubelet.go:2211] "Container runtime network not ready" networkReady="NetworkReady=false r...nitialized"
Aug 04 11:41:14 ip-172-31-4-255.ec2.internal kubelet[4092]: I0804 11:41:14.705674    4092 cni.go:239] "Unable to update cni config" err="no networks found in /etc/cni/net.d"
Aug 04 11:41:17 ip-172-31-4-255.ec2.internal kubelet[4092]: E0804 11:41:17.417558    4092 kubelet.go:2211] "Container runtime network not ready" networkReady="NetworkReady=false r...nitialized"
Hint: Some lines were ellipsized, use -l to show in full.


```
## On kmaster
##### Initialize Kubernetes Cluster
```sh
#### do ifconfig or ip address to get the ip-address " MasterServerIP" the pod pod-network-cidr adddress can be any address but is used in 
#### Calico network yaml for all pods that are started
kubeadm init --apiserver-advertise-address=<MasterServerIP> --pod-network-cidr=192.168.0.0/16

```

##### Copy kube config
To be able to use kubectl command to connect and interact with the cluster, the user needs kube config file.

In my case, the user account is praveen
```sh
create a normail user account
useradd vincebailey

mkdir /home/vincebailey/.kube
cp /etc/kubernetes/admin.conf /home/vincebailey/.kube/config
chown -R praveen:praveen /home/vincebailey/.kube
```
##### Deploy Calico network
This has to be done as the user in the above step (in my case it is __praveen__)
```sh
### Add the pod-network-cidr adddress that you have set in the calicio.yaml file

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
