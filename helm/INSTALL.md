# Step by Step Instructions to Install and try the examples

## Assumptions
- A Kubernetes cluster is available and KUBECONFIG is correct configured
- kubectl cli is installed
- Helm v3 is installed

## Steps 

### Create namespace cert-manager

```
kubectl create ns cert-manager
```

### Generate a CA keypair

- Generate a KeyPair

```
openssl genrsa -out ca.key 2048
export COMMON_NAME=sidecarinjector.sidecarinjector.svc
openssl req -x509 -new -nodes -key ca.key -subj "/CN=${COMMON_NAME}" -days 3650 -reqexts v3_req -extensions v3_ca -out ca.crt -config /usr/local/etc/openssl/openssl.cnf
```

- Create a Secret containg the CA Keypair
Note this secret must be present in the cert-manager namespace where the CertManager will be isntalledd

```
kubectl create secret tls ca-key-pair    --cert=ca.crt    --key=ca.key    --namespace=cert-manager
```

### Install Cert-Manager 

- Label the namespace so that injection is disabled

```
kubectl label ns cert-manager sidecar-injection=disabled
```

- Install Cert-Manager using Helm

```
Helm install   cert-manager jetstack/cert-manager   --namespace cert-manager   --version v1.0.3 --set installCRDs=true
```

### Install Generic Sidecar Injector

- Clone the repo 

```
git clone git@github.com:salesforce/generic-sidecar-injector.git
```

- Install Sidecar injector using Helm

```
Helm install helm/sidecarinjector/  --generate-name
kubectl get pods -n sidecarinjector
```

### Verify sidecar injector functionality

- Try some examples

```
cd Helm/examples
kubectl apply -f pod.yaml
kubectl get pods 
```

- verify the pod has a new container injected called simple-sidecar

```
 kubectl get po -o jsonpath='{range .items[*]}{"pod: "}{.metadata.name}{"\n"}{range .spec.containers[*]}{"\tname: "}{.name}{"\n"}{end}'
pod: busybox
```

### Cleanup
- list Helm releases

```
Helm list
```

- delete the Helm release for sidecar

```
Helm delete sidecarinjector-1602990110
```

- delete the Helm release for the Cert Manager

```
Helm --namespace cert-manager delete cert-manager
```

- delete the pod 

```
kubectl delete pod busybox1
```

