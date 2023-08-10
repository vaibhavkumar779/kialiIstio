### Certificate Management

**Rgistration Authority(RA):** key role to approve requests and sign hte request if valid.
**Certification Authority(CA):** sign workload requests after RA approves it.

Istiod works as RA and CA by default.

## Plugin CA : default

For istio profile=default

istiod search for CA cert secret exists in istio-system 

if not itself generates ca certificate for it

* recommendation : use your own PKI 

HOW self signed Roots works

1.  find it namespace *istio-system*
```bash
$ kubectl get secret -n istio-system
NAME              TYPE               DATA   AGE
istio-ca-secret   istio.io/ca-root   5      60m

$ kubectl get cm -n istio-system
NAME                                  DATA   AGE
istio                                 2      61m
istio-ca-root-cert                    1      60m
istio-gateway-deployment-leader       0      60m
istio-gateway-status-leader           0      60m
istio-leader                          0      60m
istio-namespace-controller-election   0      60m
istio-sidecar-injector                2      61m
kiali                                 1      58m
kube-root-ca.crt                      1      61m
prometheus                            5      58m
```

istio-ca-root-cert configmap created


2. explore its contents
```bash
$ kubectl get cm istio-ca-root-cert -o json | jq '[.data[]][0]' -r | openssl x509 -noout -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            16:53:52:b6:a6:4e:64:01:37:42:4b:76:78:f5:ce:f0
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: O = cluster.local
        Validity
            Not Before: Aug  4 07:01:44 2022 GMT
            Not After : Aug  1 07:01:44 2032 GMT
        Subject: O = cluster.local
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:e4:3d:57:3f:59:9b:cb:ae:b4:fc:d0:9a:02:ab:
                    f8:f6:6a:8b:cb:a5:62:ff:b5:e5:2c:42:f2:8e:25:
                    f1:dd:9f:fb:c3:b5:86:cb:0f:ba:31:d0:44:9a:e3:
                    2f:6b:56:fa:45:39:70:71:16:f3:cd:bc:f1:df:47:
                    37:ba:bc:07:a8:ac:eb:58:0e:a6:b5:fb:8b:9a:9a:
                    7d:9b:98:d8:34:72:1d:28:9b:d6:11:0e:25:7c:41:
                    80:a9:e7:75:38:1f:e2:a4:f0:ce:2c:5b:80:09:95:
                    84:c1:5b:ee:0e:84:de:55:0f:14:3a:f1:3e:06:a4:
                    eb:bb:31:ad:a3:3e:ab:6d:ed:b0:06:97:11:5b:2e:
                    51:e8:00:7e:eb:b0:ad:23:46:23:62:01:2a:86:b1:
                    6e:3c:2a:77:b5:bd:70:30:db:02:e2:ea:26:1c:e7:
                    6c:80:80:09:5a:79:28:7e:b4:0a:cc:d3:1e:7a:05:
                    af:40:0a:ce:81:84:6b:69:e3:0e:2f:cd:1a:09:cb:
                    08:c3:49:c0:c4:15:ea:2c:54:2c:1f:b9:dc:89:0e:
                    aa:99:23:68:79:ef:f5:78:44:b0:80:2d:39:88:f6:
                    10:e9:b1:c7:67:48:79:48:f4:9f:68:7b:ad:13:88:
                    5e:7d:a9:46:47:85:a8:38:d0:37:52:b9:d1:54:da:
                    91:bf
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Key Usage: critical
                Certificate Sign
            X509v3 Basic Constraints: critical
                CA:TRUE
            X509v3 Subject Key Identifier: 
                D2:9E:90:EB:BB:06:99:7D:04:9F:90:9D:5C:41:26:73:B2:FB:11:33
    Signature Algorithm: sha256WithRSAEncryption
    Signature Value:
        3f:21:5c:88:6a:65:6c:1f:f6:80:c9:1c:bf:b8:cb:88:9a:97:
        ab:b0:6f:2f:6b:ef:79:58:95:21:2f:be:a4:49:ef:d6:73:6c:
        e2:26:a1:17:4d:81:41:28:6a:19:70:77:c4:ef:99:cc:00:f8:
        c3:1d:42:f4:37:95:45:6f:e1:75:f8:96:a2:f5:5f:99:6e:08:
        3b:f2:23:cd:9d:cd:fd:59:aa:c4:77:f1:b6:15:7a:fe:9c:56:
        31:ff:30:fd:33:e0:82:4c:d3:68:85:39:7f:f6:f5:2d:7e:75:
        ff:b6:b7:b2:69:45:0c:12:d3:de:4e:aa:81:13:cc:b5:b1:24:
        c4:14:ab:f8:41:91:16:4d:bf:01:06:0d:1d:a1:c2:ff:f1:1d:
        af:da:78:0f:31:24:b3:44:79:2a:02:c1:65:1f:38:ec:43:13:
        dc:bc:c5:a1:e6:eb:5d:db:82:92:16:a4:18:3a:e6:6d:6e:c8:
        ea:60:f1:d0:e7:05:a5:e8:05:43:e8:0e:42:ad:1c:b5:7b:08:
        69:f4:f8:56:22:76:18:61:15:6a:22:d6:23:08:ec:ff:67:dc:
        5a:a0:3c:b3:c6:ec:7f:3b:dd:f0:24:a1:9b:55:43:12:8b:81:
        aa:72:01:07:36:46:e0:0c:63:e7:b5:75:d9:7b:df:83:23:08:
        0e:66:79:16
```
created by local system and valid for 10 years.
used RSA 256 algorithm with its own pki here.

### Trust Distribution

* istiod copies the config map istio-ca-root-cert to every namespace created in k8s cluster.

* services use these cert for mutual TLS and request RBAC.

```bash
$ kubectl get cm -A -l  istio.io/config=true
NAMESPACE      NAME                 DATA   AGE
default        istio-ca-root-cert   1      70m
devops         istio-ca-root-cert   1      70m
istio-system   istio-ca-root-cert   1      70m

$ kubectl create ns demo
namespace/demo created
$ kubectl get cm -A -l  istio.io/config=true
NAMESPACE      NAME                 DATA   AGE
default        istio-ca-root-cert   1      71m
demo           istio-ca-root-cert   1      10s
devops         istio-ca-root-cert   1      71m
istio-system   istio-ca-root-cert   1      71m

```
it does not require to enble istio injection
it is smart to not copy this config map in basic namespaces like *kube-system*

```bash
$ kubectl get ns 
NAME              STATUS   AGE
default           Active   79m
demo              Active   4m24s
devops            Active   75m
istio-system      Active   76m
kube-node-lease   Active   79m
kube-public       Active   79m
kube-system       Active   79m
```

* The moment the services are created in the namespace and sidecars are injected; the injector mounts the istio cert to istio agent and proxy as well to provide access to root cert.
* Istio-agent genrates a private key and sends CSR (Certificate Signing Requests) to Istiod.
* Istiod generates a private key in memory and this key never leaves the pod
* the certifiate expires in 24 hours
* but is renewed every 12 hours
* Configuration is done by pilot-agent env var:
  * SECRET_GRACE_PERIOD_RATIO (default is 0.5)


## CA Certs : Custom

1. In the top-level directory of the Istio installation package, create a directory to hold certificates and keys:
```
$ mkdir -p certs
$ pushd certs
```

2. Generate the root certificate and key:
```
$ make -f ../tools/certs/Makefile.selfsigned.mk root-ca
```
This will generate the following files:
  * root-cert.pem: the generated root certificate
  * root-key.pem: the generated root key
  * root-ca.conf: the configuration for openssl to generate the root certificate
  * root-cert.csr: the generated CSR for the root certificate

3. For each cluster, where you want to use the istio service mesh separately, generate an intermediate certificate and key for the Istio CA. The following is an example for cluster1:
```
$ make -f ../tools/certs/Makefile.selfsigned.mk cluster1-cacerts
```
This will generate the following files in a directory named cluster1:

  * ca-cert.pem: the generated intermediate certificates
  * ca-key.pem: the generated intermediate key
  * cert-chain.pem: the generated certificate chain which is used by istiod
  * root-cert.pem: the root certificate

You can replace *cluster1* with a string of your choice. 

If you are doing this on an offline machine, copy the generated directory to a machine with access to the clusters.

4. In each cluster, create a secret cacerts including all the input files ca-cert.pem, ca-key.pem, root-cert.pem and cert-chain.pem. For example, for cluster1:
```
$ kubectl create namespace istio-system
$ kubectl create secret generic cacerts -n istio-system \
      --from-file=cluster1/ca-cert.pem \
      --from-file=cluster1/ca-key.pem \
      --from-file=cluster1/root-cert.pem \
      --from-file=cluster1/cert-chain.pem
```
5. Return to the top-level directory of the Istio installation:
```
$ popd
```

Now when you install istio in the cluster, Istiod would not create its own certificates.


## Certification Verification

To verify its working we need to deploy istio and run some services for certificates to be generated.

So

#### 1. Deploy Istio using the demo profile.

Istio’s CA will read certificates and key from the secret-mount files.
```
$ istioctl install --set profile=demo
```

#### 2. Running an example service

**Note:** Since we have not enabled the injection the side cars are injected directly via kubectl

* Deploy the httpbin and sleep sample services.
```
$ kubectl create ns foo
$ kubectl apply -f <(istioctl kube-inject -f samples/httpbin/httpbin.yaml) -n foo
$ kubectl apply -f <(istioctl kube-inject -f samples/sleep/sleep.yaml) -n foo
```
* Deploy a policy for workloads in the foo namespace to only accept mutual TLS traffic.
```bash
$ kubectl apply -n foo -f - <<EOF
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: "default"
spec:
  mtls:
    mode: STRICT
EOF
```

#### 3. Verification of the certificates

we will verify that workload certificates are signed by the certificates that we plugged into the CA. This requires to have **openssl** installed on machine.

* Sleep 20 seconds for the mTLS policy to take effect before retrieving the certificate chain of httpbin. ***As the CA certificate used here is self-signed, the verify error:num=19:self signed certificate in certificate chain error returned by the openssl command is expected.***

```bash
$ sleep 20; kubectl exec "$(kubectl get pod -l app=sleep -n foo -o jsonpath={.items..metadata.name})" -c istio-proxy -n foo -- openssl s_client -showcerts -connect httpbin.foo:8000 > httpbin-proxy-cert.txt
```
* Parse the certificates on the certificate chain.

```bash
$ sed -n '/-----BEGIN CERTIFICATE-----/{:start /-----END CERTIFICATE-----/!{N;b start};/.*/p}' httpbin-proxy-cert.txt > certs.pem
$ awk 'BEGIN {counter=0;} /BEGIN CERT/{counter++} { print > "proxy-cert-" counter ".pem"}' < certs.pem
```
* Verify the root certificate is the same as the one specified by the administrator:
```bash
$ openssl x509 -in certs/cluster1/root-cert.pem -text -noout > /tmp/root-cert.crt.txt
$ openssl x509 -in ./proxy-cert-3.pem -text -noout > /tmp/pod-root-cert.crt.txt
$ diff -s /tmp/root-cert.crt.txt /tmp/pod-root-cert.crt.txt
Files /tmp/root-cert.crt.txt and /tmp/pod-root-cert.crt.txt are identical
```

* Verify the CA certificate is the same as the one specified by the administrator:
```bash
$ openssl x509 -in certs/cluster1/ca-cert.pem -text -noout > /tmp/ca-cert.crt.txt
$ openssl x509 -in ./proxy-cert-2.pem -text -noout > /tmp/pod-cert-chain-ca.crt.txt
$ diff -s /tmp/ca-cert.crt.txt /tmp/pod-cert-chain-ca.crt.txt
Files /tmp/ca-cert.crt.txt and /tmp/pod-cert-chain-ca.crt.txt are identical
```

* Verify the certificate chain from the root certificate to the workload certificate:
```bash
$ openssl verify -CAfile <(cat certs/cluster1/ca-cert.pem certs/cluster1/root-cert.pem) ./proxy-cert-1.pem
./proxy-cert-1.pem: OK
```

## Kubernetes CSR Integration : When CA is not Istiod

* the private is not in cluster as a secret
* steps of K8s CSR
  1. CSR request by a Service/kubectl controller (requestor)
  2. Approver approves on basis of content and requestor
  3. Signer signs the request
  4. Signed certificate read by servce (requestor)

![CSR workflow](./images%26gifs/K8sCSRworkflow.png "Kubernetes CSR")

As you can see here in the picture

![CSR example](./images%26gifs/K8sCSR.png "The CSR")

#### Kubernetes CSR with cert manager
* In Istio it is almost same:
  1. CSR requested by service / kubectl controller by gRPC to istiod
  2. istiod creates CSR (requestor) object, approves request
  3. Instead of Kubernetes, cert-manager signs request
  4. Istiod reads signed certificate
  5. Istiod responds to Servie with signed certificate

**Note:** Kubernetes and Istio work on same plane so kubernetes signing the certificate can be problemetic

![CSR with Istio](./images%26gifs/CSRwithIstio.png "CSR when there is Istio")

here we have used a cert manager to sign the certificte other Istiod as CA with the yaml
```yaml
- name: K8S_SIGNER
  value: issuers.cert-manager.io/istio-system.istio-ca
```
the istio ca can be backed up some other issuer like vault by hashicorp.

## Istio-CSR : no involvement of Istiod

![IstioCSR](./images%26gifs/Istio-CSR.png)

## Using Vault

* run vault pod on your cluster
```bash
$ helm install vault hashicorp/vault --namespace vault --create-namespace
NAME: vault
LAST DEPLOYED: Mon Aug  8 14:27:28 2022
NAMESPACE: vault
STATUS: deployed
REVISION: 1
NOTES:
Thank you for installing HashiCorp Vault!

Now that you have deployed Vault, you should look over the docs on using
Vault with Kubernetes available here:

https://www.vaultproject.io/docs/


Your release is named vault. To learn more about the release, try:

  $ helm status vault
  $ helm get manifest vault

$ kubectl get pods -A
NAMESPACE     NAME                                    READY   STATUS    RESTARTS      AGE
kube-system   coredns-6d4b75cb6d-djq4p                1/1     Running   0             23m
kube-system   etcd-minikube                           1/1     Running   0             23m
kube-system   kube-apiserver-minikube                 1/1     Running   0             23m
kube-system   kube-controller-manager-minikube        1/1     Running   1 (23m ago)   23m
kube-system   kube-proxy-qmbz4                        1/1     Running   0             23m
kube-system   kube-scheduler-minikube                 1/1     Running   0             23m
kube-system   storage-provisioner                     1/1     Running   1 (22m ago)   23m
vault         vault-0                                 0/1     Running   0             7m4s
vault         vault-agent-injector-5d4c695bf4-4xttt   1/1     Running   0             7m4s

```
follow this page work: https://learn.hashicorp.com/tutorials/vault/kubernetes-cert-manager

* install jetstack certificate manager on cluster
```bash
$ helm repo add jetstack https://charts.jetstack.io
"jetstack" has been added to your repositories
$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "jetstack" chart repository
...Successfully got an update from the "hashicorp" chart repository
Update Complete. ⎈Happy Helming!⎈
$ helm upgrade --install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --set installCRDs=true --wait
Release "cert-manager" does not exist. Installing it now.
NAME: cert-manager
LAST DEPLOYED: Mon Aug  8 14:41:20 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
cert-manager v1.9.1 has been deployed successfully!

In order to begin issuing certificates, you will need to set up a ClusterIssuer
or Issuer resource (for example, by creating a 'letsencrypt-staging' issuer).

More information on the different types of issuers and how to configure them
can be found in our documentation:

https://cert-manager.io/docs/configuration/

For information on how to configure cert-manager to automatically provision
Certificates for Ingress resources, take a look at the `ingress-shim`
documentation:

https://cert-manager.io/docs/usage/ingress/
```

* Create secret of certificate authority  for your organistaion, here I am using the ***root-cert.pem*** created for custom CA using istiod in above ways to using CA using makefile.
```
$ kubectl create secret generic istio-root-certs --from-file=root-cert.pem=certs/root-cert  -n cert-manager
secret/istio-root-certs created
```
* Now we provide the issuer to cert manager to know how to talk to vault. First create namespace *istio-system*

**Note**: For token, we use the private key for pki, here I am using the root-key.pem created for istio. the *base64 -w 0* removes the formating for using as token.
```bash
$ cat root-key.pem | base64 -w 0; echo
```

```
$ kubectl create namespace istio-system
namespace/istio-system created
$ kubectl apply -f vault-secret.yaml 
secret/vault-token created
$ kubectl apply -f vault-issuer.yaml 
issuer.cert-manager.io/vault created
```

* Next installing istio csr configured to use vault as issuer

```bash
$ helm upgrade -i cert-manager-istio-csr jetstack/cert-manager-istio-csr --namespace cert-manager --values istio-csr-values.yaml
Release "cert-manager-istio-csr" has been upgraded. Happy Helming!
NAME: cert-manager-istio-csr
LAST DEPLOYED: Tue Aug  9 11:45:02 2022
NAMESPACE: cert-manager
STATUS: deployed
REVISION: 3
TEST SUITE: None
```
* Now we have istio configurations, giving istio-csr network address and disabling istiod as CA Server. 

```bash
$ istioctl install -f istio-config.yaml --verify -y
```
* Once Istio is up and running. we add Authentication rule

```bash
$ kubectl apply -f STRICTpeerauthentication.yaml 
```
* The we run the example application
```bash
$ kubectl create namespace sandbox
$ kubectl label namespace sandbox istio-injection=enabled
$ kubectl apply -f bookinfo.yaml 
```
* wait for pod to come up
```bash
$ kubectl wait --for=condition=ready pod -l app=productpage -n sandbox 
```

* find the certfication created
```
$ istioctl proxy-config secret -n sandbox $(kubectl get pods -n sandox -o jsonpath='{.items.metadata.name}' --selector app=productpage) -o json | jq -r '.dynamicActiveSecrets[0].secrets.tlsCertificate.certificateChain inlineBytes' | base64 --decode | openssl x509 --text -noout | vim --
```