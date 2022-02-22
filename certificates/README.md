### Certificate Authority

Generate  
```openssl genrsa -out ca.key 2048```  
CSR  
```openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr```  
Signing  
```openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt```  


### Admin User

Generate  
```openssl genrsa -out admin.key 2048```   
CSR  
```openssl req -new -keyu admin.key -subj "/CN=kube-admin/O=system:masters" -out admin.csr```  
Signing  
```openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt```



### apiserver
Generate  
```openssl genrsa -out apiserver.key 2048```  
CSR  
```openssl req -new -apiserver.key -subj "/CN=kube-apiserver" -out apiserver.csr -config openssl.cnf```  
Signing  
```openssl x509 -req -in apiserver.csr -CA ca.crt -CAKey ca.key -out apisrver.crt```  


### View Certificates
```openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout```  