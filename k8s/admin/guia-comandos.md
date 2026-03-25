```sh
kubectl apply -f k8s/admin/jenkins-sa.yaml
``` 

```sh
 kubectl apply -f k8s/admin/jenkins-token.yaml
```

## Para ver el token 

```sh

kubectl get secret jenkins-deployer-token -o jsonpath='{.data.token}' | base64 --decode
```