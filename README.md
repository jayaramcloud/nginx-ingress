# nginx-ingress

1. Ensure the applications are deployed and running:
```
kubectl get deployments -n dev
```

2. Install the Nginx Ingress **Controller**:
```
helm version
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
kubectl create ns nginx-ingress
helm install nginx ingress-nginx/ingress-nginx -n nginx-ingress
kubectl get all -n nginx-ingress
```

3. Get the IP address of the Load balancer and enter it in the DNS of the domain management interface:
```
ping the hostname & ensure it resolves
ping mydomain.com
```

4. Install the Nginx Ingress **Resource**:
```
kubectl apply -f nginx-ingress.yaml
```

5. Open a browser and access the apps:
```
www.mysite.com
```
