# Rasberry Pi Kubernetes Cluster Project
---
## Table of Contents

- [About](#about)
- [What you need](#need)
- [Set-up](#setup)
- [Enabling Routing](#routing)
- [Editing the Master](#master)

## About <a name = "about"></a>

For one of my capstone projects, I decided to create a raspberry pi Kubernetes cluster to run docker containers. Why would I want this? A few reasons, we are not formally taught Kubernetes and docker in our course work, and I believe these to be incredibly useful tools in the industry. Kubernetes is a powerful platform that can scale applications quite easily. Using this on a low-cost raspberry pi 4, which has great power efficiency over a normal server machine, equates to a platform capable of surprising tasks. 

## What you need <a name = "need"></a>

- At least two Raspberry Pi's (Pi 4 is preferred)
- A power supply for each pi
- A micro sd card for each pi, each flashed with Raspian Lite
- A network switch

## Set-up <a name = "setup">

### Editing the hostname of each Pi

We need to edit the hostname for each machine else when we ssh, we don't know which machine we would be tunneling into. We also need to set up a master machine.

Navigate to **/etc/hosts** and **/etc/hostname** on each sd card and change the name of the instance, for example, I have named my machines:

```
cluster-master
worker-01 
worker-02
```

Name your cluster how you would like, currently, in my cluster, I am running a single Pi 4 4gb and then 2 Pi 3B's. I have set the Pi 4 to be the master and the 3B's as the worker nodes.

### Configuring boot options
Next, we need to edit the **/boot?cmndlin.txt** file, we need to add the following to the **end** of the **first line**:

```
cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory
```

### Installing Updates
```sudo apt update && sudo apt dist-upgrade```

### Disable Swap

```
sudo dphys-swapfile swapoff
| sudo dphys-swapfile uninstall
| sudo apt purge dphys-swapfile
```

### Rebooting
 ```sudo reboot```

### Installing Docker
```
curl -ssl get.docker.com | sh
sudo usermod -aG docker jay
```


### Set Docker daemon options
Edit the daemon.json file:

```sudo vim /etc/docker/daemon.json```

```
 {
   "exec-opts": ["native.cgroupdriver=systemd"],
   "log-driver": "json-file",
   "log-opts": {
     "max-size": "100m"
   },
   "storage-driver": "overlay2"
 }
```

##Enabling Routing<a name = "routing">
Find ** /etc/sysctl.conf** and uncomment this line:

```   #net.ipv4.ip_forward=1```

### Reboot again

```sudo reboot```

### Adding the Kubernetes Repo
```sudo vim /etc/apt/sources.list.d/kubernetes.list```

Add:

``` deb http://apt.kubernetes.io/ kubernetes-xenial main```

Adding the GPG Keys:

 ``` curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add - ```


###Installing all packages

```
 sudo apt update
 sudo apt install kubeadm kubectl kubelet
```

## Editing the Master<a name = "master">


### Initializing Kubernetes
Run:

```
 sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```
After this runs, you will be given a **join** command to add your worker machines to the cluster, save this somewhere else as we are not ready to add them just yet. 

### Setting the config director
The previous command should have given you the following additional commands, run them all:

``` mkdir -p ~.kube
 sudo cp /etc/kubernetes/admin.conf ~/.kube/config
 sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

If it recommends other commands, run those instead. 

### Installing network drivers
Lets install the flannel netwrok driver:

```
 kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

### Checking pods
Let's make sure the workers come up:

```kubectl get pods --all-namespaces```

### Joining worker nodes
You should have saved the **join** command from earlier, run this command on each worker node now. 

### Checking node status
The final step is to validate if the worker nodes have successfully been joined to the master, run the following command until you see each node:

``` kubectl get nodes```