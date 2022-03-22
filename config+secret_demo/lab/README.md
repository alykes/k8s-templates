Create a password  
```htpasswd -c .htpasswd user```  
_eg. user:secret_

Create a secret with the file that it creates  
```kubectl create secret generic nginx-htpasswd --from-file=.htpasswd```  

Deploy the pod and test that you can connect to it with user/pass  
```kubectl exec -it busybox -- curl -u user:secret 192.168.194.70```

