##Commands for working  with deployments

###create
```kubectl create deployment -f deployment-definition.yaml```
###get
```kubectl get deployments```
###update
```kubectl apply -f deployment-definition.yaml```  
```kubectl set image deployment/my-deployment nginx=nginx:1.9.0```
###status
```kubectl rollout status deployment/my-deployment```  
```kubectl rollout history deployment/my-deployment```
###rollback
```kubectl rollout undo deployment/my-deployment```