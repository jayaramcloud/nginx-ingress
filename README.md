# nginx-ingress

1. Ensure the applications are **deployed** and running:

 The code to build the React Cat application is:
 
  https://github.com/jayaramcloud/react-cat

 The docker file whick builds the Container is:
 
   https://github.com/jayaramcloud/react-cat/blob/main/Dockerfile

 The CI/CD project which builds the container from this Dockerfile is:
 
  https://github.com/jayaramcloud/react-cat/blob/main/.github/workflows/build-docker-image.yml 

 The Docker container image is present at:
 
  https://hub.docker.com/repository/docker/canada/en

 After this login to the Kuberntes Cluster to deploy the application.
```
neofinone@cloudshell:~$ kubectl apply -f  react-cat-service.yaml 
service/react-cat created
neofinone@cloudshell:~$ kubectl get deployment -n dev
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
react-cat   3/3     3            3           53s

neofinone@cloudshell:~$ kubectl apply -f  react-cat.yaml         
deployment.apps/react-cat created
neofinone@cloudshell:~$ kubectl get service -n dev
NAME        TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
react-cat   ClusterIP   10.8.12.178   <none>        80/TCP    91s

```

2. Install the Nginx Ingress **Controller**:
```
helm version
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
kubectl create ns nginx-ingress
helm install nginx ingress-nginx/ingress-nginx -n nginx-ingress
kubectl get all -n nginx-ingress
```

```
neofinone@cloudshell:~ (neofine)$ helm version
version.BuildInfo{Version:"v3.5.0", GitCommit:"32c22239423b3b4ba6706d450bd044baffdcf9e6", GitTreeState:"clean", GoVersion:"go1.15.6"}
neofinone@cloudshell:~ (neofine)$ helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
"ingress-nginx" has been added to your repositories
neofinone@cloudshell:~ (neofine)$ kubectl create ns nginx-ingress
namespace/nginx-ingress created
neofinone@cloudshell:~ (neofine)$ helm install nginx ingress-nginx/ingress-nginx -n nginx-ingress
NAME: nginx
LAST DEPLOYED: Sat Feb  5 16:29:49 2022
NAMESPACE: nginx-ingress
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The ingress-nginx controller has been installed.
It may take a few minutes for the LoadBalancer IP to be available.
You can watch the status by running 'kubectl --namespace nginx-ingress get services -o wide -w nginx-ingress-nginx-controller'

An example Ingress that makes use of the controller:
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: example
    namespace: foo
  spec:
    ingressClassName: nginx
    rules:
      - host: www.example.com
        http:
          paths:
            - backend:
                service:
                  name: exampleService
                  port:
                    number: 80
              path: /
    # This section is only required if TLS is to be enabled for the Ingress
    tls:
      - hosts:
        - www.example.com
        secretName: example-tls

If TLS is enabled for the Ingress, a Secret containing the certificate and key must also be provided:

  apiVersion: v1
  kind: Secret
  metadata:
    name: example-tls
    namespace: foo
  data:
    tls.crt: <base64 encoded cert>
    tls.key: <base64 encoded key>
  type: kubernetes.io/tls
neofinone@cloudshell:~ (neofine)$ kubectl get all -n nginx-ingress
NAME                                                  READY   STATUS    RESTARTS   AGE
pod/nginx-ingress-nginx-controller-559779bdcc-7dgqd   0/1     Running   0          15s

NAME                                               TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)                      AGE
service/nginx-ingress-nginx-controller             LoadBalancer   10.8.1.72    <pending>     80:31671/TCP,443:32493/TCP   15s
service/nginx-ingress-nginx-controller-admission   ClusterIP      10.8.2.214   <none>        443/TCP                      15s

NAME                                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-ingress-nginx-controller   0/1     1            0           15s

NAME                                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-ingress-nginx-controller-559779bdcc   1         1         0       15s

----------------If you see the LoadBalancer is in <pending> state, wait for 3-5 minutes & run the command again --------------
neofinone@cloudshell:~ (neofine)$ kubectl get all -n nginx-ingress
NAME                                                  READY   STATUS    RESTARTS   AGE
pod/nginx-ingress-nginx-controller-559779bdcc-7dgqd   1/1     Running   0          119s

NAME                                               TYPE           CLUSTER-IP   EXTERNAL-IP     PORT(S)                      AGE
service/nginx-ingress-nginx-controller             LoadBalancer   10.8.1.72    34.136.24.118   80:31671/TCP,443:32493/TCP   119s
service/nginx-ingress-nginx-controller-admission   ClusterIP      10.8.2.214   <none>          443/TCP                      119s

NAME                                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-ingress-nginx-controller   1/1     1            1           119s

NAME                                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-ingress-nginx-controller-559779bdcc   1         1         1       119s
```

3. Get the IP address of the Load balancer and enter it in the DNS of the domain management interface:
https://domains.google.com/registrar/reallifeprojects.com/dns#resourcerecords
```
neofinone@cloudshell:~ (neofine)$ sudo apt-get install iputils-ping
```

ping the hostname  www.reallifeprojects.com & ensure it resolves with the new IP address:

```
neofinone@cloudshell:~ (neofine)$ ping www.reallifeprojects.com
PING www.reallifeprojects.com (34.136.24.118) 56(84) bytes of data.
64 bytes from 118.24.136.34.bc.googleusercontent.com (34.136.24.118): icmp_seq=1 ttl=108 time=39.0 ms
64 bytes from 118.24.136.34.bc.googleusercontent.com (34.136.24.118): icmp_seq=2 ttl=108 time=38.9 ms
64 bytes from 118.24.136.34.bc.googleusercontent.com (34.136.24.118): icmp_seq=3 ttl=108 time=39.0 ms
64 bytes from 118.24.136.34.bc.googleusercontent.com (34.136.24.118): icmp_seq=4 ttl=108 time=38.9 ms
^C
--- www.reallifeprojects.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 6ms
rtt min/avg/max/mdev = 38.912/38.960/38.999/0.200 ms
```

4. Install the Nginx Ingress **Resource**:
```
neofinone@cloudshell:~$ kubectl apply -f nginx-ingress.yaml 
ingress.networking.k8s.io/reallifeprojects created
neofinone@cloudshell:~$ kubectl get ing -n dev
NAME               CLASS   HOSTS                      ADDRESS   PORTS   AGE
reallifeprojects   nginx   www.reallifeprojects.com             80      34s

neofinone@cloudshell:~$ curl www.reallifeprojects.com
<!DOCTYPE html>
<html lang="en">
  <head>
--------------
    -->
  </body>
</html>
neofinone@cloudshell:~$ 
```

5. Open a browser and access the apps:
```
www.reallifeprojects.com
```
