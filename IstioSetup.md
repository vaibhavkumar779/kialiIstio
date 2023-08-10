## Setting Up Istio with Minikube:

* Install Docker which will used as driver 

Reference: https://docs.docker.com/engine/install/ubuntu/

* Install Kubectl 

Reference: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

* Install Minikube: There two ways I used 2nd as 1st gave some problems

  * Using this link : https://minikube.sigs.k8s.io/docs/start/
  * Using these steps instead:
      ```bash
      $ wget https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
      $ chmod +x minikube-linux-amd64
      $ sudo mv minikube-linux-amd64 /usr/local/bin/minikube
      ```

* Install Istio

Reference : https://istio.io/latest/docs/setup/getting-started/

**NOTE:** You need to provide PATH every time to use this istio configuration for binary file of Istio
```bash
$ export PATH=IstioDirectoryPath/bin:$PATH
```
**Or**

```bash
$ export PATH=$PWD/bin:$PATH
```

* Up the minikube cluster, I ran with defined cpus and memory
    ```bash 
    $ minikube start –cpus 6 –memory 8192
    😄  minikube v1.26.0 on Ubuntu 22.04
    ✨  Automatically selected the docker driver
    📌  Using Docker driver with root privileges
    👍  Starting control plane node minikube in cluster minikube
    🚜  Pulling base image ...
    🔥  Creating docker container (CPUs=6, Memory=8192MB) ...
    🐳  Preparing Kubernetes v1.24.1 on Docker 20.10.17 ...
        ▪ Generating certificates and keys ...
        ▪ Booting up control plane ...
        ▪ Configuring RBAC rules ...
    🔎  Verifying Kubernetes components...
        ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
    🌟  Enabled addons: storage-provisioner, default-storageclass
    🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default

    ```
  **Note:** To run properly, Istio requires 4 vCPUs and 8GB of RAM to work in minikube.

* Install Istio in Cluster
```bash
$ istioctl install --set profile=default -y
ile=default -y
✔ Istio core installed                                                                              
✔ Istiod installed                                                                                  
✔ Ingress gateways installed                                                                        
✔ Installation complete                                                                             Making this installation the default for injection and validation.

Thank you for installing Istio 1.14.  Please take a few minutes to tell us about your install/upgrade experience!  https://forms.gle/yEtCbt45FZ3VoDT5A
```
This installs istio core , istiod, istio ingress gateway on the cluster within the namespace *istio-system*. You can check it in following way:

```bash
$ kubectl get all -n istio-system
NAME                                        READY   STATUS    RESTARTS   AGE
pod/istio-ingressgateway-5f86977657-cqv56   1/1     Running   0          72s
pod/istiod-7587989b4f-6s9w5                 1/1     Running   0          112s

NAME                           TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                                      AGE
service/istio-ingressgateway   LoadBalancer   10.96.84.182    <pending>     15021:30723/TCP,80:30373/TCP,443:30929/TCP   70s
service/istiod                 ClusterIP      10.101.176.29   <none>        15010/TCP,15012/TCP,443/TCP,15014/TCP        110s

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/istio-ingressgateway   1/1     1            1           72s
deployment.apps/istiod                 1/1     1            1           112s

NAME                                              DESIRED   CURRENT   READY   AGE
replicaset.apps/istio-ingressgateway-5f86977657   1         1         1       72s
replicaset.apps/istiod-7587989b4f                 1         1         1       112s

NAME                                                       REFERENCE                         TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/istio-ingressgateway   Deployment/istio-ingressgateway   <unknown>/80%   1         5         1          71s
horizontalpodautoscaler.autoscaling/istiod                 Deployment/istiod                 <unknown>/80%   1         5         1          111s

```

Since we are using minikube **EXTERNAL-IP** is pending so we do following and check the IP then:

```bash
$ minikube tunnel --cleanup
Status:	
	machine: minikube
	pid: 1124634
	route: 10.96.0.0/12 -> 192.168.49.2
	minikube: Running
	services: [istio-ingressgateway]
    errors: 
		minikube: no errors
		router: no errors
		loadbalancer emulator: no errors

$ kubectl get svc -n istio-system
NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP    PORT(S)                                      AGE
istio-ingressgateway   LoadBalancer   10.96.84.182    10.96.84.182   15021:30723/TCP,80:30373/TCP,443:30929/TCP   5m56s
istiod                 ClusterIP      10.101.176.29   <none>         15010/TCP,15012/TCP,443/TCP,15014/TCP        6m36s
```


## Adding the App to be used for service mesh

* create a separate namespace to use
```bash
$ kubectl create namespace devops
namespace/devops created
```
* add the label to enable istio injection to this namespace
```bash
$ kubectl label namespace devops istio-injection=enabled
namespace/devops labeled 
$ kubectl get ns devops --show-labels
NAME     STATUS   AGE     LABELS
devops   Active   2m11s   istio-injection=enabled,kubernetes.io/metadata.name=devops
```
* run the services in this cluster 
```bash
$ cd istio-demo
$ kubectl apply -f bookinfo.yaml -n devops
service/details created
serviceaccount/bookinfo-details created
deployment.apps/details-v1 created
service/ratings created
serviceaccount/bookinfo-ratings created
deployment.apps/ratings-v1 created
service/reviews created
serviceaccount/bookinfo-reviews created
deployment.apps/reviews-v1 created
deployment.apps/reviews-v2 created
deployment.apps/reviews-v3 created
service/productpage created
serviceaccount/bookinfo-productpage created
deployment.apps/productpage-v1 created
```
  It takes time to deploy all services about 4 to 6 minutes. then you can check all in following way
  ```bash
  $ kubectl get all -n devops
  NAME                                  READY   STATUS    RESTARTS   AGE
  pod/details-v1-b48c969c5-z2tl6        2/2     Running   0          4m30s
  pod/productpage-v1-74fdfbd7c7-2pkxn   2/2     Running   0          4m24s
  pod/ratings-v1-b74b895c5-5dqpf        2/2     Running   0          4m30s
  pod/reviews-v1-68b4dcbdb9-4wlxr       2/2     Running   0          4m28s
  pod/reviews-v2-565bcd7987-vpd2k       2/2     Running   0          4m28s
  pod/reviews-v3-d88774f9c-tcgtp        2/2     Running   0          4m27s

  NAME                  TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
  service/details       ClusterIP   10.102.66.54   <none>        9080/TCP   4m30s
  service/productpage   ClusterIP   10.101.71.31   <none>        9080/TCP   4m25s
  service/ratings       ClusterIP   10.100.40.4    <none>        9080/TCP   4m30s
  service/reviews       ClusterIP   10.96.160.13   <none>        9080/TCP   4m30s

  NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
  deployment.apps/details-v1       1/1     1            1           4m30s
  deployment.apps/productpage-v1   1/1     1            1           4m24s
  deployment.apps/ratings-v1       1/1     1            1           4m30s
  deployment.apps/reviews-v1       1/1     1            1           4m29s
  deployment.apps/reviews-v2       1/1     1            1           4m28s
  deployment.apps/reviews-v3       1/1     1            1           4m27s

  NAME                                        DESIRED   CURRENT   READY   AGE
  replicaset.apps/details-v1-b48c969c5        1         1         1       4m30s
  replicaset.apps/productpage-v1-74fdfbd7c7   1         1         1       4m24s
  replicaset.apps/ratings-v1-b74b895c5        1         1         1       4m30s
  replicaset.apps/reviews-v1-68b4dcbdb9       1         1         1       4m29s
  replicaset.apps/reviews-v2-565bcd7987       1         1         1       4m28s
  replicaset.apps/reviews-v3-d88774f9c        1         1         1       4m27s
  ```

  You will notice two pods for each service. This is because we have added label for *istio-injection=enabled* for this namespace. The *istiod* is able to create sidecars for each service using *istio core*.

* deploy the files in istio objects

```bash
$ kubectl apply -f istioobjects/ -n devops
gateway.networking.istio.io/bookinfo-gateway created
virtualservice.networking.istio.io/bookinfo created
destinationrule.networking.istio.io/productpage created
destinationrule.networking.istio.io/reviews created
destinationrule.networking.istio.io/ratings created
destinationrule.networking.istio.io/details created
virtualservice.networking.istio.io/productpage created
virtualservice.networking.istio.io/reviews created
virtualservice.networking.istio.io/ratings created
virtualservice.networking.istio.io/details created
```
  There are 3 things inside the istioobjects:
  * istio gateway:
    * While we have enabled the external ip for load balancer but it is not connected to our app and there is no externl gatway so we use the istio gatway which can be modified as required using istio for external connection.
  
  * destination rule:
    * this provides path to connect the services inside the cluster which can be later on modifed

  * virtual service:
    * this is collection of service configuration of connecting to path based on conditions.


Now to check if the app is able to connect we first create some environment varibales.
```bash
$ export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
$ export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
$ export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].port}')
$ echo "$INGRESS_HOST"
10.96.84.182
$ echo "$INGRESS_PORT"
80
$ echo "$SECURE_INGRESS_PORT"
443
```
Similarly add GATEWAY URL and check if the you can access the app externally by using curl or copy-paste it on browser
```bash
$ export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
$ echo "$GATEWAY_URL"
10.96.84.182:80
$ echo "http://$GATEWAY_URL/productpage"
http://10.96.84.182:80/productpage
$ curl "http://$GATEWAY_URL/productpage"
<!DOCTYPE html>
<html>
  <head>
    <title>Simple Bookstore App</title>
<meta charset="utf-8">
<meta http-equiv="X-UA-Compatible" content="IE=edge">
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<!-- Latest compiled and minified CSS -->
<link rel="stylesheet" href="static/bootstrap/css/bootstrap.min.css">

<!-- Optional theme -->
<link rel="stylesheet" href="static/bootstrap/css/bootstrap-theme.min.css">

  </head>
  <body>
    
    

<nav class="navbar navbar-inverse navbar-static-top">
  <div class="container">
    <div class="navbar-header">
      <a class="navbar-brand" href="#">BookInfo Sample</a>
    </div>
    
    <button type="button" class="btn btn-default navbar-btn navbar-right" data-toggle="modal" href="#login-modal">Sign
      in</button>
    
  </div>
</nav>

<!---
<div class="navbar navbar-inverse navbar-fixed-top">
  <div class="container">
    <div class="navbar-header pull-left">
      <a class="navbar-brand" href="#">Microservices Fabric BookInfo Demo</a>
    </div>
    <div class="navbar-header pull-right">
      <button type="button" class="navbar-toggle" data-toggle="collapse" data-target=".navbar-collapse">
        <span class="icon-bar"></span>
        <span class="icon-bar"></span>
        <span class="icon-bar"></span>
      </button>
    </div>
    <div class="navbar-collapse collapse">

      <button type="button" class="btn btn-default navbar-btn pull-right" data-toggle="modal" data-target="#login-modal">Sign in</button>

    </div>
  </div>
</div>
-->

<div id="login-modal" class="modal fade" role="dialog">
  <div class="modal-dialog">
    <div class="modal-content">
      <div class="modal-header">
        <button type="button" class="close" data-dismiss="modal">&times;</button>
        <h4 class="modal-title">Please sign in</h4>
      </div>
      <div class="modal-body">
        <form method="post" action='login' name="login_form">
          <p><input type="text" class="form-control" name="username" id="username" placeholder="User Name"></p>
          <p><input type="password" class="form-control" name="passwd" placeholder="Password"></p>
          <p>
            <button type="submit" class="btn btn-primary">Sign in</button>
            <button type="button" class="btn btn-default" data-dismiss="modal">Cancel</button>
          </p>
        </form>
      </div>
    </div>

  </div>
</div>

<div class="container-fluid">
  <div class="row">
    <div class="col-md-12">
      <h3 class="text-center text-primary">The Comedy of Errors</h3>
      
      <p>Summary: <a href="https://en.wikipedia.org/wiki/The_Comedy_of_Errors">Wikipedia Summary</a>: The Comedy of Errors is one of <b>William Shakespeare's</b> early plays. It is his shortest and one of his most farcical comedies, with a major part of the humour coming from slapstick and mistaken identity, in addition to puns and word play.</p>
      
    </div>
  </div>

  <div class="row">
    <div class="col-md-6">
      
      <h4 class="text-center text-primary">Book Details</h4>
      <dl>
        <dt>Type:</dt>paperback
        <dt>Pages:</dt>200
        <dt>Publisher:</dt>PublisherA
        <dt>Language:</dt>English
        <dt>ISBN-10:</dt>1234567890
        <dt>ISBN-13:</dt>123-1234567890
      </dl>
      
    </div>

    <div class="col-md-6">
      
      <h4 class="text-center text-primary">Book Reviews</h4>
      
      <blockquote>
        <p>An extremely entertaining play by Shakespeare. The slapstick humour is refreshing!</p>
        <small>Reviewer1</small>
        
      </blockquote>
      
      <blockquote>
        <p>Absolutely fun and entertaining. The play lacks thematic depth when compared to other plays by Shakespeare.</p>
        <small>Reviewer2</small>
        
      </blockquote>
      
      <dl>
        <dt>Reviews served by:</dt>
        <u>reviews-v1-68b4dcbdb9-4wlxr</u>
        
      </dl>
      
    </div>
  </div>
</div>


    
<!-- Latest compiled and minified JavaScript -->
<script src="static/jquery.min.js"></script>

<!-- Latest compiled and minified JavaScript -->
<script src="static/bootstrap/js/bootstrap.min.js"></script>

<script type="text/javascript">
  $('#login-modal').on('shown.bs.modal', function () {
    $('#username').focus();
  });
</script>

  </body>
</html>

```

* add monitoring to cluster for service mesh
```bash
$ kubectl apply -f monitor/
serviceaccount/kiali created
configmap/kiali created
clusterrole.rbac.authorization.k8s.io/kiali-viewer created
clusterrole.rbac.authorization.k8s.io/kiali created
clusterrolebinding.rbac.authorization.k8s.io/kiali created
role.rbac.authorization.k8s.io/kiali-controlplane created
rolebinding.rbac.authorization.k8s.io/kiali-controlplane created
service/kiali created
deployment.apps/kiali created
serviceaccount/prometheus created
configmap/prometheus created
clusterrole.rbac.authorization.k8s.io/prometheus created
clusterrolebinding.rbac.authorization.k8s.io/prometheus created
service/prometheus created
deployment.apps/prometheus created
```
  This adds two thing prometheus for data extractor and kiali to work on it. It is configured for using as per the *istio-system* namespace.

  ```
  $ kubectl get all -n istio-system
  NAME                                        READY   STATUS              RESTARTS   AGE
  pod/istio-ingressgateway-5f86977657-cqv56   1/1     Running             0          70m
  pod/istiod-7587989b4f-6s9w5                 1/1     Running             0          70m
  pod/kiali-575cc8cbf-9qj6b                   0/1     ContainerCreating   0          14s
  pod/prometheus-6544454f65-nm65s             0/2     ContainerCreating   0          13s

  NAME                           TYPE           CLUSTER-IP      EXTERNAL-IP    PORT(S)                                      AGE
  service/istio-ingressgateway   LoadBalancer   10.96.84.182    10.96.84.182   15021:30723/TCP,80:30373/TCP,443:30929/TCP   70m
  service/istiod                 ClusterIP      10.101.176.29   <none>         15010/TCP,15012/TCP,443/TCP,15014/TCP        70m
  service/kiali                  ClusterIP      10.98.6.199     <none>         20001/TCP,9090/TCP                           15s
  service/prometheus             ClusterIP      10.111.174.15   <none>         9090/TCP                                     13s

  NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
  deployment.apps/istio-ingressgateway   1/1     1            1           70m
  deployment.apps/istiod                 1/1     1            1           70m
  deployment.apps/kiali                  0/1     1            0           15s
  deployment.apps/prometheus             0/1     1            0           13s

  NAME                                              DESIRED   CURRENT   READY   AGE
  replicaset.apps/istio-ingressgateway-5f86977657   1         1         1       70m
  replicaset.apps/istiod-7587989b4f                 1         1         1       70m
  replicaset.apps/kiali-575cc8cbf                   1         1         0       15s
  replicaset.apps/prometheus-6544454f65             1         1         0       13s

  NAME                                                       REFERENCE                         TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
  horizontalpodautoscaler.autoscaling/istio-ingressgateway   Deployment/istio-ingressgateway   <unknown>/80%   1         5         1          70m
  horizontalpodautoscaler.autoscaling/istiod                 Deployment/istiod                 <unknown>/80%   1         5         1          70m
  ```
  You can see two new services or apps being added to *istio-system*
  Now in UI setup we use kiali to configure various things.
