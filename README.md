# Kubernetes Demo Apps

## Demo Apps 1: Kubernetes Hello Apps
![](https://i.imgur.com/yYosI6V.png)

This application has a deployment and a secured ingress. Before deploying `hello-kubernetes.yaml` you need to create a TLS certificate in your local machine and configure it as a secret in the Kubernetes platform. The steps to create a self-signed cert are below:

### 1. Creating a self-signed certificate

**Step 1: Generate a TLS cert**
```
openssl req -x509 -nodes -days 365 \
-newkey rsa:2048 \
-out hello-ingress-tls.crt \
-keyout hello-ingress-tls.key \
-subj "/CN=hello.apps.corp.local/O=hello-ingress-tls"
```

**Step 2: Create a Secret**
```bash
kubectl create secret tls hello-ingress-tls --namespace shop --key hello-ingress-tls.key --cert hello-ingress-tls.crt
```
Verify that the secret has been created
```
$ kubectl get secret hello-ingress-tls
NAME                TYPE                DATA   AGE
hello-ingress-tls   kubernetes.io/tls   2      19s
```

### 2. Deploy the yaml file
```
kubectl apply -f hello-kubernetes.yaml
```
