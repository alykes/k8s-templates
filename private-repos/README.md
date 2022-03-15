Create a secret for the docker registry

```shell
kubectl create docker-registry docker-creds \
--docker-server= \
--docker-username= \
--docker-password= \
--docker-email=
```



in Pod definition  
```yaml
spec: 
  containers:
  - name: private-container
  image: private-registry.io/apps/private-image
  imagePullSecrets:
  - name: docker-credz
```

