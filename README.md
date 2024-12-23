# k8s-proxmox-installation

## prepare VM
```
# Note: the below changes to be done on vms (master nodes & worker nodes)

disable swap

    swapoff -a 
    vi /etc/fstab 
	comment the swap line putting hash infront of the line
	ex: #UUID=9fb8bc78-f247-4b69-8b4a-44171adeda7b none            swap    sw              0       0
```
## install Kubernetes Using Script

### `Step1: On Master Node Only`
```
## Install Docker
sudo wget https://raw.githubusercontent.com/lerndevops/labs/master/scripts/installDocker.sh -P /tmp
sudo chmod 755 /tmp/installDocker.sh
sudo bash /tmp/installDocker.sh
sudo systemctl restart docker.service

## Install CRI-Docker
sudo wget https://raw.githubusercontent.com/lerndevops/labs/master/scripts/installCRIDockerd.sh -P /tmp
sudo chmod 755 /tmp/installCRIDockerd.sh
sudo bash /tmp/installCRIDockerd.sh
sudo systemctl restart cri-docker.service

## Install kubeadm,kubelet,kubectl
sudo wget https://raw.githubusercontent.com/lerndevops/labs/master/scripts/installK8S.sh -P /tmp
sudo chmod 755 /tmp/installK8S.sh
sudo bash /tmp/installK8S.sh

## Initialize kubernetes Master Node

   sudo kubeadm init --cri-socket unix:///var/run/cri-dockerd.sock --ignore-preflight-errors=all

   sudo mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config

   ## install networking driver -- Weave/flannel/canal/calico etc...

   ## below installs calico networking driver

   kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.1/manifests/calico.yaml

   # Validate:  kubectl get nodes

## Load the br_netfilter Module

sudo modprobe br_netfilter

# Verify the Setting:

ls /proc/sys/net/bridge/bridge-nf-call-iptables

# Set the Value to 1

echo 1 | sudo tee /proc/sys/net/bridge/bridge-nf-call-iptables

# Make the Setting Persistent (optional):

echo "net.bridge.bridge-nf-call-iptables = 1" | sudo tee -a /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-ip6tables = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

```

### `Step2: On All Worker Nodes`

```
## Install Docker
sudo wget https://raw.githubusercontent.com/lerndevops/labs/master/scripts/installDocker.sh -P /tmp
sudo chmod 755 /tmp/installDocker.sh
sudo bash /tmp/installDocker.sh
sudo systemctl restart docker.service

## Install CRI-Docker
sudo wget https://raw.githubusercontent.com/lerndevops/labs/master/scripts/installCRIDockerd.sh -P /tmp
sudo chmod 755 /tmp/installCRIDockerd.sh
sudo bash /tmp/installCRIDockerd.sh
sudo systemctl restart cri-docker.service

## Install kubeadm,kubelet,kubectl
sudo wget https://raw.githubusercontent.com/lerndevops/labs/master/scripts/installK8S.sh -P /tmp
sudo chmod 755 /tmp/installK8S.sh
sudo bash /tmp/installK8S.sh

## Load the br_netfilter Module

sudo modprobe br_netfilter

# Verify the Setting:

ls /proc/sys/net/bridge/bridge-nf-call-iptables

# Set the Value to 1

echo 1 | sudo tee /proc/sys/net/bridge/bridge-nf-call-iptables

# Make the Setting Persistent (optional):

echo "net.bridge.bridge-nf-call-iptables = 1" | sudo tee -a /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-ip6tables = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

## Run Below on Master Node to get join token

kubeadm token create --print-join-command

    copy the kubeadm join token from master & add --cri-socket unix:///var/run/cri-dockerd.sock as below & then run on all worker nodes

    Ex: kubeadm join 10.128.15.231:6443 --cri-socket unix:///var/run/cri-dockerd.sock --token mks3y2.v03tyyru0gy12mbt \
           --discovery-token-ca-cert-hash sha256:3de23d42c7002be0893339fbe558ee75e14399e11f22e3f0b34351077b7c4b56
```
