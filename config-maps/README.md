###Imperative Approach

```
kubectl create condfigmap \  
app-config --from-literal=APP_COLOR=blue \
--from-literal=APP_ENVIRONMENT=prod
```

```
kubectl create configmap \
app-config --from-file=app_config.properties
```

###Declarative Approach

create a definition file for the config map as follows

```config-map.yaml```
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: red
  APP_MODE: prod  
```
```kubectl create -f config-map.yaml```

###Viewing ConfigMaps
```kubectl get configmaps```  
```kubectl describe configmaps```
&nbsp;  

###Injecting a configmap into a POD definition file
```buildoutcfg
spec: 
  containers: 
  - name: nginx
    image: nginx
    ports:
      - containerPort: 8080
    envFrom:
      - configMapRef:
        name: app-config
```