### Switch Routing

- Useful Commands  

```ip link```

```ip addr add 192.168.1.10/24 dev eth0``` (Device 1)    
```ip addr add 192.168.1.11/24 dev eth0``` (Device 2)  

```route```  

```ip route add 192.168.2.0/24 via 192.168.1.1```  
```ip route add 192.168.1.0/24 via 192.168.2.1```  

```ip route add default via 192.168.2.1```  
```ip route add 0.0.0.0/0 via 192.168.2.1```  

| A | B | C |  
|--|--|--|
| 192.168.1.5 | 192.168.1.6 | 192.168.2.5 |   
||192.168.2.6||

For A to access C  
- on A add  
```ip route add 192.168.2.0/24 via 192.168.1.6```  
- on B add  
```ip route add 192.168.1.0/24 via 192.168.2.6```    

Additional Step:  
```cat /proc/sysnet/ipv4/ip_forward```  
- 0 - do not forward
- 1 - allow forwarding

To persist changes after a reboot you will need to persist changes in the following files:

modify ```/etc/sysctl.conf```  
```
...
net.ipv4.ip_forward = 1
...
```  

```/etc/network/interfaces```

### DNS

```cat /etc/hosts```  
```
10.10.10.100		HOME-PC
```  

```cat /etc/resolv.conf```  
```
nameserver		10.10.10.53
nameserver		1.1.1.1
search			home.network	wfh.network
```
- To pick order of resolution

```cat /etc/nsswitch.conf```  
```
hosts:		files dns
```

- Tools
  - dig
  - nslookup
  - ping


### CoreDNS - Simple configuration


```cat /etc/hosts```
```
192.168.1.100	web
192.168.1.101	db
192.168.1.200	pc
192.168.1.201	laptop
```  

```cat Corefile```
```
.{
	hosts /etc/hosts
}
```

- Diving a little deeping into the configuration, the portion of interest is the plugin for **kubernetes**
```buildoutcfg
.:53 {
  errors
  health
  kubernetes cluster.local in-addr-arpa ip6.arpa { #local dns name
    pods insecure   #responsible for creating a record for a pod in the cluster 
    # /etc/resolve.conf on the dns server in the cluster 
    # the dash names are disabled by default, this line enables them
    upstream  
    fallthrough in-addr.arpa ip6.arpa
  }
  prometheus :9153
  proxy . /etc/resolv.conf
  cache 30
  reload
}
```  
### Configuration of resolv.conf
  - Pod points to local resolve.conf
  - local pod resolve.conf points to the coredns service IP
  - coredns POD resolve.conf points to the node
  - The nod resolve.conf points to an external name server

This core file _/etc/coredns/Corefile_ is passed into k8s as a configmap named _coredns_  
```kubectl get configmap -n kube-system```
  
To see the settings on the kubelet for clusterDNS  
```cat /var/lib/kubelet/config.yaml```  
```
...
clusterDNS:
- 10.96.0.10
clusterDomain: cluster.local
...
```  

To run coreDNS
```./coredns```

**Note:** the POD resolv.conf also has a search entry  
```search default.svc.cluster.local svc.cluster.local cluster.local```
It only has service entries, therefore to get to a POD, you need to specify the full FQDN.

### Network Namespaces

- Creating a pipe using a pair of virtual eth

-------     --------
- red ------- blue -
-------     --------

- Create the pipe 
ip link add veth-red type veth peer name veth-blue

- Attach each interface to each namespace
ip link set veth-red netns red
ip link set veth-blue netns blue

- Assign an address to each namespace
ip -n red addr add 192.168.15.1 dev veth-red
ip -n blue addr add 192.168.15.1 dev veth-blue

- Set the interfaces as up
ip -n red link set veth-red up
ip -n blue link set veth-blue up

- Visibility
ip netns exec red arp
(this display the blue neighbour as an entry)
ip netns exec blue arp
(this display the red neighbour as an entry)


-- Creating a Bridge

=======             ========  
= red =             = blue =   
=======             ========  
       \            /  
        \          /  
	     ==========  
         = bridge = (v-net-0)  
	     ==========
		/          \  
	   /	        \  
==========          ========  
= orange =          = grey =  
==========          ========

Create the bridge    
```ip link add v-net-0 type bridge```  

===========  
= v-net-0 =  
===========  

- Set the bridge to up
ip link set dev v-net-0 up

comment: ip -n red link del veth-red

- Create the pipes
-------------     ----------------
- veth-red ------- veth-red-br -
-------------     ----------------

-------------     ----------------
- veth-blue ------- veth-blue-br -
-------------     ----------------

ip link add veth-red type veth peer name veth-red-br
ip link add veth-blue type veth peer name veth-blue-br


- Connect the namespaces to the virtual devices
ip link set veth-red netns red
- specify the master for it
ip link set veth-red-br master v-net-0

- Do the same for the blue namespace
ip link set veth-blue netns blue
ip link set veth-blue-br master v-net-0

- Assign an address to each namespace
ip -n red addr add 192.168.15.1 dev veth-red
ip -n blue addr add 192.168.15.1 dev veth-blue

- Set the interfaces as up
ip -n red link set veth-red up
ip -n blue link set veth-blue up

- Add ip address to the bridge
ip addr add 192.168.15.5/24 dev v-net-0


-- Routing traffic outside these networks

- Add route to the localhost  (assuming the localhost has two addresses already 192.168.15.5 and 192.168.1.2)
ip netns exec blue ip route add 192.168.1.0/24 via 192.168.15.5

- you need to enable nat on the host (localhost) so it has a return path
iptables -t nat -A POSTROUTING -s 192.168.15.0/24 -j MASQUERADE


-- Routing traffic to the internet
ip netns exec blue ip route add default via 192.168.15.5

-- Routing traffic into the namespaces
- route traffic on a port on the local host into the namespace
iptables -t nat -A PREROUTING --dport 80 --to-destination 192.168.15.2:80 -j DNAT
