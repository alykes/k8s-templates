- **Draining Nodes** 
  - Drain a node with pods only  
  ```kubectl drain <node-name>```
  - Drain a node with daemonsets  
  ```kubectl drain <node-name> --ignore-daemonsets```
  - Drain a node with daemonsets plus additional components (eg, ReplicaSets)  
  ```kubectl drain <node-name> --ignore-daemonsets --force```
  

- **Scheduling/Unscheduling Nodes**

  - Cordon a node (Make it unschedulable)  
  ```kubectl cordon <node-name>```

  - Uncordon a node (Make it schedulable)  
  ```kubectl uncordon <node-name>```

### Upgrading a Node  

Control plane must be upgraded first  
- Upgrade Plan to see which version is available to upgrade to  
```kubeadm upgrade plan```  


Upgrade kubeadm first, then components.
```shell
apt-mark unhold kubeadm && \
apt-get update && apt-get install -y kubeadm=1.20.0-00 && \
apt-mark hold kubeadm
```

```kubeadm version```  

```sudo kubeadm upgrade apply v1.20.0```  

```shell
apt-mark unhold kubelet kubectl && \
apt-get update && apt-get install -y kubelet=1.20.0-00 kubectl=1.20.0-00 && \
apt-mark hold kubelet kubectl
```
```shell
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

```shell
kubectl uncordon <node-name>
```

- Upgrade Worker Nodes

```shell
apt-mark unhold kubeadm && \
apt-get update && apt-get install -y kubeadm=1.20.x-00 && \
apt-mark hold kubeadm
```

```sudo kubeadm upgrade node```  

```kubectl drain <node-name> --ignore-daemonsets```

```shell
apt-mark unhold kubelet kubectl && \
apt-get update && apt-get install -y kubelet=1.20.0-00 kubectl=1.20.0-00 && \
apt-mark hold kubelet kubectl
```

```shell
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

```kubectl uncordon <node-name>```