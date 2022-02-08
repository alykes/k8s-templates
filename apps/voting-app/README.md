for testing the webpages from the big scary outside world

kubectl port-forward --address 0.0.0.0 svc/result-service 30005:80
kubectl port-forward --address 0.0.0.0 svc/voting-service 30004:80
