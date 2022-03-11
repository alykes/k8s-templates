Create private key
```shell
openssl genrsa -out mydude.key 2048
openssl req -new -key mydude.key -out mydude.csr
```

Encode the csr cert
```cat mydude.csr | base64 | tr -d "\n"```

Create CertificateSigningRequest
```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: user
spec:
  request: <encoded-csr output from the command above>
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400  # one day or change it to whatever you like.
  usages:
  - client auth # must be included for normal users
```

Approve certificate signing request  
```kubectl certificate approve mydude```

Retrieve the certificate from the CSR:  
```kubectl get csr/mydude -o yaml```

**Note:** _The certificate value is in Base64-encoded format under status.certificate._  

```kubectl get csr mydude -o jsonpath='{.status.certificate}'| base64 -d > mydude.crt```

Create Role and RoleBinding imperatively  
```kubectl create role developer --verb=create --verb=get --verb=list --verb=update --verb=delete --resource=pods```    
```kubectl create rolebinding developer-binding-mydude --role=developer --user=mydude```

Add this user into the kubeconfig file.

```kubectl config set-credentials mydude --client-key=mydude.key --client-certificate=mydude.crt --embed-certs=true```

Then, you need to add the context:  
```kubectl config set-context mydude --cluster=kubernetes --user=mydude```

To test it, change the context to mydude:  
```kubectl config use-context mydude```
