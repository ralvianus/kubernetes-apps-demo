# GSLB Traffic Split Apps

This demo is forked from [yogendra/nsx-alb-demo](https://github.com/yogendra/nsx-alb-demo).

## Demo Apps 5: GSLB Traffic Splitting
![](https://i.imgur.com/vyiLAbc.png)

Simple UI to demonstract Blue Green deployment. When you have a single endpoint to access multiple version of the applicaiton and you are moving/dripping traffic from one instance (blue to green), see it happen visually.

### Creating a self-signed certificate

**Step 1: Create a new namespace**
```
kubectl create ns gslb
oc new-project gslb
```

**Step 2: Generate a TLS cert for `echo-server` and `client`**
```
openssl req -x509 -nodes -days 365 \
-newkey rsa:2048 \
-out client-ingress-tls.crt \
-keyout client-ingress-tls.key \
-subj "/CN=client.apps.acepod.com/O=client-ingress-tls"
```
```
openssl req -x509 -nodes -days 365 \
-newkey rsa:2048 \
-out echo-server-ingress-tls.crt \
-keyout echo-server-ingress-tls.key \
-subj "/CN=echo-server.apps.acepod.com/O=echo-server-ingress-tls"
```

**Step 3: Create a Secret**
```bash
kubectl create secret tls client-ingress-tls --key client-ingress-tls.key --cert client-ingress-tls.crt
```
```bash
kubectl create secret tls echo-server-ingress-tls --key echo-server-ingress-tls.key --cert echo-server-ingress-tls.crt
```


### Deploy the `echo-server` yaml file
Change directory
```bash
cd demo-01-trafficsplit
```
Adjust and apply the `echo-server` yaml file in the correct clusters
```bash
kubectl apply -f server/echo-server-blue.yaml
kubectl apply -f server/echo-server-green.yaml
```

### Deploy the `client` yaml file
#### **Setup Probe Configuration**

Select one of the probe mechanism and setup config for that

a. DNS based probe: edit and Apply `client-dns.yaml`

```bash
kubectl apply -f client-dns.yaml
```

**OR**

b. HTTP based probe: edit and Apply `client-http.yaml`

```bash
kubectl apply -f client-http.yaml
```

#### **Configuration Options**
|value|Meaning | Example |
|-|-|-|
|`port` | Port on which server should run (Number) | `5000` |
|`endpoint.address` | Addres of the endpoint to probe. (URL) | `https://mysite.com` | |
|`endpoint.type` | Type of endpoint - `web` or `dns` | `web`, `dns`| |
|`endpoint.versions` | Version names. Required for web type endpoint (Array of string) | `["blue","green"]` | |
|`endpoint.nameservers` | List of name servers. Required for dns type endpoint | `["8.8.8.8","4.4.4.4"]`|
|`endpoint.versionMap` | Map IP addresses to version, Required for dns type endpoint | `{"10.0.0.1": "blue", "10.0.0.2": "green"}`|
| `probe.interval` | Amount of time in milliseconds between probes. (Number) | `10`|
| `sample.interval` | Amount of time in milliseconds between capturing sample (Number) | `10` |
| `sample.max` | Maximum samples to keep (Number) | `600`|
| `graph.colors` | Array of colors names for the graph | `["blue","green"]` |

#### **Deploy Client Module**

Edit and apply `client.yaml`

```bash
kubectl apply -f client.yaml
```
