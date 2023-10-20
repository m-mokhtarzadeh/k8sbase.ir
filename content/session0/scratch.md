---
date: 2022-12-11T15:26:15Z
lastmod: 2022-12-11T15:26:15Z
publishdate: 2022-12-11T15:26:15Z

title: Create Lab from scratch on Rocky linux 9
description: Kubernetes Fundamental
images:
- images/kops.jpg
---

[comment]: <> (# Create Lab from scratch)
### Step 1: Download rocky linux 9
Download rocky linux 9 from following repository:  
[Rocky Linux download repository](https://rockylinux.org/download)

### Step 2: Install dependencies
Remove Rocky default repository and Isfahan university repos:
```bash
rm -rf /etc/yum.repos.d/*
cd /etc/yum.repos.d
curl -O repo.iut.ac.ir/repo/iutrocky9.repo
curl -O repo.iut.ac.ir/repo/iutepel9.repo
dnf update
```

Install following dependencies:
```bash
dnf install vim gpm dhclient nano tar bzip2 kernel-headers kernel-devel open-vm-tools
```

Install Virutlabox extensions:
```bash
mount /dev/sr0 /mnt
cd /mnt
./VBoxLinuxAdditions.run
```

### Step 3: Disable firewall
Disable firewall:  
```bash
systemctl stop firewalld
systemctl disable firewalld
systemctl mask firewalld
```

### Step 4: Disable selinux
Disable selinux:  
```bash
sed -i s/^SELINUX=.*$/SELINUX=disabled/ /etc/selinux/config
```

### Step 5: Update system packages
Update system packages with following command:
```bash
dnf update
```

### Step 6: Install latest version of docker
Add docker repository to server package repository:
```bash
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```
Install docker packages:
```bash
sudo dnf install docker-ce docker-ce-cli containerd.io
```
Enable and start docker service with following commands:
```bash
sudo systemctl start docker
sudo systemctl enable docker
sudo systemctl status docker
```

### Step 7: Pull images for required hands-on lab
```bash
docker pull nginx
```

### Step 9: Configuring System for kubernetes installation
```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

```bash
cat << EOF > /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```

```bash
sysctl --system
```

Disable swap
```bash
swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
# remove swap from /etc/fstab
# make sure swap in lvm mode removed from grub
```

### Step 10: Install kubernetes
Add kubernetes repository to system repository:
```bash
cat << EOF > /etc/yum.repos.d/kubernetes.repo
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

Install kubernetes packages:
```bash
dnf install -y kubelet-1.27.6-0 kubeadm-1.27.6-0 kubectl-1.27.6-0 --disableexcludes=kubernetes
```

Enable kubernetes service:
```bash
systemctl enable kubelet
```

Pull kubernetes required container images:
```bash
kubeadm config images pull --kubernetes-version 1.27.6 --cri-socket /run/cri-dockerd.sock
docker pull docker.io/calico/cni:v3.26.1
docker pull docker.io/calico/kube-controllers:v3.26.1
docker pull docker.io/calico/node:v3.26.1
```

Download required manifest:
```bash
curl https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/calico.yaml -O
```