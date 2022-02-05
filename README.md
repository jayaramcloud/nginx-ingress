# nginx-ingress

Ensure the applications are deployed and running:
```
kubectl get deployments -n dev
```

Install the Nginx Ingress Controller
```
helm version
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
kubectl create ns nginx-ingress
helm install nginx ingress-nginx/ingress-nginx -n nginx-ingress
kubectl get all -n nginx-ingress
```

Get the IP address of the Load balancer and enter it in the DNS of the domain management interface:
```
ping the hostname & ensure it resolves
ping mydomain.com
```


Install the Nginx Ingress 
```
kubectl apply -f nginx-ingress.yaml
```

Open a browser and access the apps:
```
www.mysite.com
```
