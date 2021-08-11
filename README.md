# Kubernetes Demo Apps

## Overview
This demo app is deployed in the lab environment. I have setup a DNS name `apps.corp.local` as a delegated names. You need to adjust the `host` value in each yaml file to reflect the domain name you are using.

This demo app is deployed in the environment where there is integration with NSX Advanced Load Balancer (formerly AVI Networks). NSX ALB provides ingress controller for multiple Kubernetes/Openshift clusters. The ingress settings in the yaml file is labelled with `avi-gslb` so that NSX ALB will be able to pickup the configuration and realize it in the NSX ALB platform.

More on the NSX ALB and how it integrates with Kubernetes/Openshift platform in my [blog post](https://alvianus.net/posts/2020/10/installing-avi-kubernetes-operator-ako-on-openshift-container-platform-4.5/)

> Some container images are pulled from Docker hub. As Docker hub impose a pull limit, you may need to authenticate with Docker hub or increasing the limit as documented [here](https://www.docker.com/increase-rate-limits)

## Demo Apps 1: Kubernetes Hello Application
This application has a deployment and a secured ingress. Before deploying `hello-kubernetes.yaml` you need to create a TLS certificate in your local machine and configure it as a secret in the Kubernetes platform. The steps to create a self-signed cert are below:

### Creating a self-signed certificate

**Step 1: Create a new namespace**
```
kubectl create ns hello
oc new-project hello
```

**Step 2: Generate a TLS cert**
```
openssl req -x509 -nodes -days 365 \
-newkey rsa:2048 \
-out hello-ingress-tls.crt \
-keyout hello-ingress-tls.key \
-subj "/CN=hello.apps.lab01.one/O=hello-ingress-tls"
```

**Step 3: Create a Secret**
```bash
kubectl create secret tls hello-ingress-tls --key hello-ingress-tls.key --cert hello-ingress-tls.crt
```
Verify that the secret has been created
```
$ kubectl get secret hello-ingress-tls
NAME                TYPE                DATA   AGE
hello-ingress-tls   kubernetes.io/tls   2      19s
```

### Deploy the yaml file
Change directory
```bash
cd demo-01-hello-kubernetes
```
Apply the yaml file
```bash
kubectl apply -f hello-kubernetes.yaml
```

Verify the Deployment
```bash
$ kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
hello-kubernetes   3/3     3            3           10d
```

Verify the Services
```bash
$ kubectl get services
NAME               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
hello-kubernetes   ClusterIP   172.30.112.148   <none>        80/TCP    10d
```

Verify the Ingress
```bash
$ kubectl get ingress
NAME                       CLASS    HOSTS                   ADDRESS   PORTS     AGE
hello-kubernetes-ingress   <none>   hello.apps.corp.local             80, 443   10d
```

## Demo Apps 2: Online Shopping Application

**Online Boutique** is a cloud-native microservices demo application.
Online Boutique consists of a 10-tier microservices application. The application is a
web-based e-commerce app where users can browse items,
add them to the cart, and purchase them.

This application can be found on its [Github Page](https://github.com/GoogleCloudPlatform/microservices-demo)

The example apps above does not include ingress service, so I add ingress service as per yaml below:
```
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: frontend-ingress
  labels:
    app: avi-gslb
spec:
  tls:
  - hosts:
      - shop.apps.lab01.one
    secretName: shop-ingress-tls
  rules:
    - host: shop.apps.lab01.one
      http:
        paths:
        - path: /
          pathType: Prefix
          backend:
            serviceName: frontend
            servicePort: 80
```
I have ingress on subdomain `shop.apps.lab01.one` and it is redirecting to frontend service.

### Creating a self-signed certificate

**Step 1: Create a new namespace**
```
kubectl create ns shop
oc new-project shop
```

**Step 2: Generate a TLS cert**
```
openssl req -x509 -nodes -days 365 \
-newkey rsa:2048 \
-out shop-ingress-tls.crt \
-keyout shop-ingress-tls.key \
-subj "/CN=shop.apps.lab01.one/O=shop-ingress-tls"
```

**Step 3: Create a Secret**
```bash
kubectl create secret tls shop-ingress-tls --key shop-ingress-tls.key --cert shop-ingress-tls.crt
```
Verify that the secret has been created
```
$ kubectl get secret shop-ingress-tls
NAME               TYPE                DATA   AGE
shop-ingress-tls   kubernetes.io/tls   2      7s
```

### Deploy the yaml file
Change directory
```bash
cd demo-02-shop
```
Apply the yaml file
```bash
kubectl apply -f shop.yaml
```

Verify the Deployment
```bash
$ kubectl get deployments
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
adservice               1/1     1            1           18d
cartservice             1/1     1            1           18d
checkoutservice         1/1     1            1           18d
currencyservice         1/1     1            1           18d
emailservice            1/1     1            1           18d
frontend                3/3     3            3           18d
paymentservice          1/1     1            1           18d
productcatalogservice   1/1     1            1           18d
recommendationservice   1/1     1            1           18d
redis-cart              1/1     1            1           18d
shippingservice         1/1     1            1           18d

```

Verify the Ingress
```bash
$ kubectl get ingress
NAME               CLASS    HOSTS                  ADDRESS        PORTS   AGE
frontend-ingress   <none>   shop.apps.lab01.one   10.20.10.150   80      18d
```

## Demo Apps 3: DVWA Apps
[Damn Vulnerable Web Application (DVWA)](https://dvwa.co.uk/) is a PHP/MySQL web application that is damn vulnerable. Its main goal is to be an aid for security professionals to test their skills and tools in a legal environment, help web developers better understand the processes of securing web applications and to aid both students & teachers to learn about web application security in a controlled class room environment.

The aim of DVWA is to practice some of the most common web vulnerabilities, with various levels of difficulty, with a simple straightforward interface. Please note, there are both documented and undocumented vulnerabilities with this software. This is intentional. You are encouraged to try and discover as many issues as possible.

The purpose of this demo application is to test Web Application Firewall (WAF) capability from NSX Advanced Load Balancer (formerly AVI Networks).

### Creating a self-signed certificate

**Step 1: Create a new namespace**
```
kubectl create ns dvwa-apps
oc new-project dvwa-apps
```

**Step 2: Generate a TLS cert**
```
openssl req -x509 -nodes -days 365 \
-newkey rsa:2048 \
-out dvwa-ingress-tls.crt \
-keyout dvwa-ingress-tls.key \
-subj "/CN=dvwa.apps.lab01.one/O=dvwa-ingress-tls"
```

**Step 3: Create a Secret**
```bash
kubectl create secret tls dvwa-ingress-tls --namespace dvwa-apps --key dvwa-ingress-tls.key --cert dvwa-ingress-tls.crt
```
Verify that the secret has been created
```
$ kubectl get secret dvwa-ingress-tls
NAME               TYPE                DATA   AGE
dvwa-ingress-tls   kubernetes.io/tls   2      24s
```

### Deploy the yaml file
Change directory
```bash
cd demo-03-dvwa-app
```
Apply the yaml file
```bash
kubectl apply -f dvwa-app.yaml
```
Verify the Deployment
```bash
$ kubectl get deployments
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
dvwa-apps   1/1     1            1           8m6s
```
Verify the Pod has been deployed
```bash
kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
dvwa-apps-5d9fbf5995-wgzrb   1/1     Running   0          8m35s
```
Verify the Pod log similar to this output:
```
$ kubectl logs dvwa-apps-5d9fbf5995-wgzrb
[+] Starting mysql...
Starting MariaDB database server: mysqld.
[+] Starting apache
Starting Apache httpd web server: apache2AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 10.129.2.97. Set the 'ServerName' directive globally to suppress this message
.
==> /var/log/apache2/access.log <==

==> /var/log/apache2/error.log <==
[Mon Mar 29 14:28:29.227224 2021] [mpm_prefork:notice] [pid 270] AH00163: Apache/2.4.25 (Debian) configured -- resuming normal operations
[Mon Mar 29 14:28:29.227326 2021] [core:notice] [pid 270] AH00094: Command line: '/usr/sbin/apache2'

==> /var/log/apache2/other_vhosts_access.log <==

```

### Attaching WAF Policy into Ingress
WAF policy is a specific set of rules that protects the application. This policy is enabled by associating it with a virtual service. More info on the WAF policy can be found [here](https://avinetworks.com/docs/20.1/waf-policy/)

HostRule CRD is primarily targeted to be used by the Operator. This CRD can be used to express additional virtual host properties. The virtual host FQDN is matched from either Kubernetes Ingress or OpenShift Route based objects.

HostRule CRD is the means to attach WAF Policy into ingress with specific FQDN. More details can be found [on AVI Networks Github](https://github.com/avinetworks/avi-helm-charts/blob/master/docs/AKO/crds/hostrule.md)

> :warning: WAF Policy has to be created in the NSX ALB Controller before creating the HostRule object
>
> :warning: WAF Policy only applies to secured FQDN (TLS based ingress)


Deploy the HostRule object
```bash
kubectl apply -f dvwa-hostrule.yaml
```

Verify if the HostRule object is accepted.
```
$ kubectl get hr
NAME            HOST                   STATUS     AGE
dvwa-hostrule   dvwa.apps.corp.local   Accepted   26m
```
Look at the Application Dashboard in the NSX ALB Controller, a new Virtual Service Object is created. The Virtual Service has a secured icon which means WAF is enabled for that particular Virtual Service
![](https://i.imgur.com/oYUpVA8.png)

## Demo Apps 4: Bookinfo App
See <https://istio.io/docs/examples/bookinfo/>.

![](https://i.imgur.com/PwUFeb6.png)

> :warning: This app is deployed with Istio Service Mesh
>

### Creating a self-signed certificate

**Step 1: Generate a TLS cert**
```bash
openssl req -x509 -nodes -days 365 \
-newkey rsa:2048 \
-out bookinfo-ingress-tls.crt \
-keyout bookinfo-ingress-tls.key \
-subj "/CN=bookinfo.apps.corp.local/O=bookinfo-ingress-tls"
```

**Step 2: Create a Secret**
```bash
kubectl create secret tls bookinfo-ingress-tls --key bookinfo-ingress-tls.key --cert bookinfo-ingress-tls.crt
```
Verify that the secret has been created
```bash
$ kubectl get secret bookinfo-ingress-tls
NAME                   TYPE                DATA   AGE
bookinfo-ingress-tls   kubernetes.io/tls   2      <invalid>
```

### Deploy the yaml file
Change directory
```bash
cd demo-04-bookinfo
```
Apply the yaml file
```bash
kubectl apply -f bookinfo.yaml
kubectl apply -f bookinfo-ingress.yaml
kubectl apply -f bookinfo.yaml
```
