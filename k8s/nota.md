## Crear la imagen dentro del Cluster de minikube

1. Vincular la terminal: Apunta la consola al Docker interno de Minikube

```sh
eval $(minikube docker-env)
```
2. Construir imagen de app de Node: Crea la imagen localmente dentro cluster (paso previo ejecutar eval $(minikube docker-env))
```sh
docker build -t node-api-ts:latest .
```
3. Verificar: Confirma que la imagen fue creada dentro de cluster de Minikube
```sh
minikube image ls
```