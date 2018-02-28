# MRAMESH

This repository is powered by MRA & NGINMESH projects:

https://github.com/nginxinc/router-mesh-architecture

https://github.com/nginmesh/nginmesh

Demo is running in(temporary):
http://nginmesh.nginxps.com



NGINX Microservices Reference Architecture (MRA) Applications used:

1. Web
The web container is a simple PHP application consisting of a single web page rendered dynamically by a twig template served by the PHP 5 built-in server. The content for the template comes from a request that the PHP class makes to the backend container

2. Backend
The backend container is a simple rails app that runs on unicorn. Requests to the ```/backend``` route will render the static content that is defined in the app.rb.

## Quick Start
Below are instructions to setup the Istio service mesh in a Kubernetes cluster using NGINX as a sidecar.

### Prerequisites
Make sure you have a Kubernetes cluster with alpha feature enabled. Please see [Prerequisites](https://istio.io/docs/setup/kubernetes/quick-start.html#prerequisites) for setting up a cluster.

### Installing Istio and nginmesh
Below are instructions for installing Istio with NGINX as a sidecar:
1. Download Istio release 0.3.0:
```
curl -L https://git.io/getLatestIstio | ISTIO_VERSION=0.3.0 sh -
```
2. Download nginmesh release 0.3.0:
```
curl -L https://github.com/nginmesh/nginmesh/releases/download/0.3.0/nginmesh-0.3.0.tar.gz | tar zx
```
3. Download mramesh repo:
```
git clone https://github.com/nginmesh/mramesh.git
```

4. Deploy Istio without enabled mutual TLS (mTLS) authentication between sidecars:

```
kubectl create -f istio-0.3.0/install/kubernetes/istio.yaml
```

5. Deploy an initializer for automatic sidecar injection:
```
kubectl apply -f nginmesh-0.3.0/install/kubernetes/istio-initializer.yaml
```

6. Ensure the following Kubernetes services are deployed: istio-pilot, istio-mixer, istio-ingress, istio-egress:
```
kubectl get svc  -n istio-system  
```
```
 NAME            CLUSTER-IP      EXTERNAL-IP       PORT(S)                       AGE
  istio-egress    10.83.247.89    <none>            80/TCP                        5h
  istio-ingress   10.83.245.171   35.184.245.62     80:32730/TCP,443:30574/TCP    5h
  istio-pilot     10.83.251.173   <none>            8080/TCP,8081/TCP             5h
  istio-mixer     10.83.244.253   <none>            9091/TCP,9094/TCP,42422/TCP   5h
```

7. Ensure the following Kubernetes pods are up and running: istio-pilot-* , istio-mixer-* , istio-ingress-* , istio-egress-* and istio-initializer-* :
```
kubectl get pods -n istio-system    
```
```
  istio-ca-3657790228-j21b9           1/1       Running   0          5h
  istio-egress-1684034556-fhw89       1/1       Running   0          5h
  istio-ingress-1842462111-j3vcs      1/1       Running   0          5h
  istio-initializer-184129454-zdgf5   1/1       Running   0          5h
  istio-pilot-2275554717-93c43        1/1       Running   0          5h
  istio-mixer-2104784889-20rm8        2/2       Running   0          5h
```
### Deploy MRA Application
In this section we deploy the Bookinfo application, which is taken from the Istio samples. Please see [Bookinfo](https://istio.io/docs/guides/bookinfo.html)  for more details.

1. Create nginx-mra namespace for application, and set it as default one:
```
kubectl apply -f mramesh/namespace.json
kubectl config set-context $(kubectl config current-context) --namespace=nginx-mra
```

2. Build and push images(GCP in our case, could use Docker hub as well) for application:

```
cd mramesh/backend && make build && make gpush
cd mramesh/web && make build && make gpush
```
2. Deploy the application:
```
kubectl apply -f mramesh/backend/backend.yaml
kubectl apply -f mramesh/web/web.yaml
kubectl apply -f mramesh/ingress.yaml
```
3. Confirm that both application services are deployed: backend and web.
```
kubectl get services -n nginx-mra
```
```
NAME                       CLUSTER-IP   EXTERNAL-IP   PORT(S)              AGE
backend                    10.0.0.31    <none>        8080/TCP              6m
web                        10.0.0.1     <none>        8080/TCP              7d
```

4. Confirm that all application pods are running --backend-v1-* , web-v1-* :
```
kubectl get pods -n nginx-mra
```
```
NAME                                        READY     STATUS    RESTARTS   AGE
backend-v1-88855b877-k9qkr                  2/2       Running   0          6h
web-v1-84654d84cf-xtbll                     2/2       Running   0          6h
```

5. Get the public IP of the Istio Ingress controller. If the cluster is running in an environment that supports external load balancers, run:
```
kubectl get svc -n istio-system | grep -E 'EXTERNAL-IP|istio-ingress'
```
OR
```
kubectl get ingress -o wide       
```

6. Open the MRA application in a browser using the following link:
```
http://<Public-IP-of-the-Ingress-Controller>/
```
### Uninstalling the Application
1. To uninstall application, run:
```
kubectl delete -f mramesh/backend/backend.yaml
kubectl delete -f mramesh/web/web.yaml
kubectl delete -f mramesh/ingress.yaml
```


### Uninstalling Istio
1. To uninstall the Istio core components:

```
kubectl delete -f istio-0.3.0/install/kubernetes/istio.yaml
```

2. To uninstall the initializer, run:
```
kubectl delete -f nginmesh-0.3.0/install/kubernetes/istio-initializer.yaml
```



