# k8s-istio
hello-world app using k8s istio

I am using Carl Dunkelberger's app.yaml and also follow this link https://docs.mirantis.com/docker-enterprise/v3.1/dockeree-products/mke/deploy-apps-with-kubernetes/istio-ingress.html to enable Kubernetes Ingress.

Prerequisite: 
1. Using Mirantis Kubernetes Engine 

Steps to deploy hello app using Istio:
1. Open the MKE web UI.
2. Select Admin Settings.
3. Select Ingress.
4. Under the Kubernetes tab: Click the slider to enable Ingress for Kubernetes and the rest just leave it and save.
5. Deploy app.yml and check the resources 
```
kubectl apply -f https://raw.githubusercontent.com/doanbutar/k8s-istio/main/app.yml
namespace/hello-example created
gateway.networking.istio.io/default-http-gateway created
deployment.apps/hello created
service/hello created
virtualservice.networking.istio.io/hello created

$ kubectl get all -n hello-example 
NAME                         READY   STATUS    RESTARTS   AGE
pod/hello-764cdffd8d-v4htv   1/1     Running   0          43s

NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/hello   ClusterIP   10.96.73.224   <none>        80/TCP    44s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/hello   1/1     1            1           45s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/hello-764cdffd8d   1         1         1       45s


$ kubectl get virtualservices.networking.istio.io -n hello-example 
NAME    GATEWAYS                 HOSTS   AGE
hello   [default-http-gateway]   [*]     66s

$ kubectl get gateways.networking.istio.io -n hello-example 
NAME                   AGE
default-http-gateway   75s

$ kubectl get svc -n istio-system 
NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                                                                                      AGE
istio-ingressgateway   NodePort    10.96.195.65    <none>        80:33000/TCP,443:33001/TCP,31400:33002/TCP,15030:33071/TCP,15443:33157/TCP,15020:33470/TCP   67m
istio-pilot            ClusterIP   10.96.142.135   <none>        15010/TCP,15011/TCP,8080/TCP,15014/TCP                                                       67m
istio-policy           ClusterIP   10.96.37.241    <none>        9091/TCP,15004/TCP,15014/TCP                                                                 67m
istio-telemetry        ClusterIP   10.96.94.96     <none>        9091/TCP,15004/TCP,15014/TCP,42422/TCP                                                       67m
```
6. Test the hello app
```
$ INGRESS_PORT=33000           >>> Default HTTP Port for Ingress on MKE web UI.
$ INGRESS_HOST=xx.xx.xx.xx  >>> One of your nodes. You can get it using kubectl get nodes

# curl -v -H "Host:hello.example.com" --resolve "hello.example.com:$INGRESS_PORT:$INGRESS_HOST" "http://hello.example.com:$INGRESS_PORT/"
* Added hello.example.com:33000:XX.XX.XX.XX to DNS cache
* Hostname hello.example.com was found in DNS cache
*   Trying XX.XX.XX.XX...
* TCP_NODELAY set
* Connected to hello.example.com (XX.XX.XX.XX) port 33000 (#0)
> GET / HTTP/1.1
> Host:hello.example.com
> User-Agent: curl/7.58.0
> Accept: */*
>
< HTTP/1.1 200 OK
< x-powered-by: Express
< content-type: text/html; charset=utf-8
< content-length: 111
< etag: W/"6f-U4ut6Q03D1uC/sbzBDyZfMqFSh0"
< date: Thu, 25 Feb 2021 03:17:40 GMT
< x-envoy-upstream-service-time: 65
< server: istio-envoy
<
<link rel="stylesheet" type="text/css" href="css/style.css" />
<div class="container">
    Hello World!
* Connection #0 to host hello.example.com left intact

```
