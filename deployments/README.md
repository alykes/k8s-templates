## Commands for working  with deployments

#### Create
```kubectl create deployment -f deployment-definition.yaml```
#### Get
```kubectl get deployments```
#### Update
```kubectl apply -f deployment-definition.yaml```  
```kubectl set image deployment/my-deployment nginx=nginx:1.9.0```
#### Status
```kubectl rollout status deployment/my-deployment```  
```kubectl rollout history deployment/my-deployment```
#### Rollback
```kubectl rollout undo deployment/my-deployment```
