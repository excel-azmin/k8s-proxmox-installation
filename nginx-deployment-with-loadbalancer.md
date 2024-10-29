# Multi Cluster Environment is Required

 * Create Deployment `nginx-deployment.yaml`

```
# nginx-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP

```
* Install Nginx Ingress Controller

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

* Installa Metallb
```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.8/config/manifests/metallb-native.yaml
```
* Configure MetalLB with a pool of IPs in your network `metallb-config.yaml`

```
# metallb-config.yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: nginx-pool
  namespace: metallb-system
spec:
  addresses:
    - 192.168.10.250-192.168.10.254 # Use an IP range in your network

---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: l2-advertisement
  namespace: metallb-system
```
 
*  Create Ingress

  ```
# nginx-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  namespace: default  # Set the namespace
  annotations:
    kubernetes.io/ingress.class: "nginx"  # Define the Ingress class
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: test.local.com  # Ensure this matches your DNS or IP
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80

```

*  Set Up TLS/SSL with Cert-Manager (For HTTPS)
*  Install Cert-Manager to manage SSL certificates
```
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.7.1/cert-manager.yaml
```
* Create a ClusterIssuer for Let’s Encrypt `letsencrypt-issuer.yaml`
```
# letsencrypt-issuer.yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: your-email@example.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
```
*  Update the Ingress to use TLS

```
# nginx-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  namespace: default  # Set the namespace
  annotations:
    kubernetes.io/ingress.class: "nginx"  # Define the Ingress class
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  tls:
  - hosts:
    - test.local.com  # Host for which the TLS certificate is issued
    secretName: test-local-com-tls  # Secret where the TLS cert is stored
  rules:
  - host: test.local.com  # Ensure this matches your DNS or IP
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80

```
