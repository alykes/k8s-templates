```shell
sudo swapoff -a
```

```shell
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sudo modprobe br_netfilter

sudo sysctl --system  


```



```shell
sudo apt-get update  

sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release  


curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg  

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null  

sudo apt-get update  
sudo apt-get install docker-ce docker-ce-cli containerd.io  
```


```shell
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl


sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet=1.23.0-00 kubeadm=1.23.0-00 kubectl=1.23.0-00
sudo apt-mark hold kubelet kubeadm kubectl
```
### Cgroup Driver  
Make sure docker and kubelet are using the same one

docker info | grep -i cgroup

modify file 
/etc/systemd/system/kubelet.service.d/10-kubeadm.conf

add the flag --cgroup-driver=cgroupfs 
```
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --cgroup-driver=cgroupfs"
```

On the **master** node, initialise the cluster with kubeadm
```shell
sudo kubeadm init --pod-network-cidr 10.244.0.0/16 --kubernetes-version 1.23.0
```

```shell
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```


- Install CNI ```kubectl apply -f "https://```

- join nodes ```kubeadm token create --print-join-command```