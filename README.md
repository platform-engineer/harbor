# harbor
install harbor on linux fedora


+ [goharbor/harbor-helm: The helm chart to deploy Harbor](https://github.com/goharbor/harbor-helm)


+ proxmox
+ fedora server 

## install helm

Since Fedora 35, helm is available on the official repository. You can install helm with invoking:

+ [Helm - Installing Helm](https://helm.sh/docs/intro/install/)
+ [Harbor docs - Deploying Harbor with High Availability via Helm](https://goharbor.io/docs/2.8.0/install-config/harbor-ha-helm/)
+ [goharbor/harbor-helm: The helm chart to deploy Harbor](https://github.com/goharbor/harbor-helm)

update
```bash
sudo dnf update -y
```

install
```bash
sudo dnf install -y helm
```



## instal k3c for rancher

+ [K3s](https://k3s.io/)

```bash
curl -sfL https://get.k3s.io | sh - 
```

Check for Ready node, takes ~30 seconds 
```bash
sudo k3s kubectl get node
```

```bash
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
```


## install harbor

Add Helm repository
Download Harbor helm chart:
```bash
helm repo add harbor https://helm.goharbor.io
helm fetch harbor/harbor --untar
```

update your Helm repositories.
```bash
helm repo update
```


Install the Harbor helm chart with a release name my-release:
```bash
helm install my-release harbor/harbor
```



```bash
helm install my-release harbor/
```


```bash
helm repo list
```

## Generate a Certificate Authority 

+ [Harbor docs | Configure HTTPS Access to Harbor](https://goharbor.io/docs/2.0.0/install-config/configure-https/)


```bash
openssl genrsa -out ca.key 4096
```

```bash
openssl req -x509 -new -nodes -sha512 -days 3650 \
 -subj "/C=DE/ST=BERLIN/L=Berlin/O=example/OU=Personal/CN=harbor.fritz.box" \
 -key ca.key \
 -out ca.crt
```

The certificate usually contains a .crt file and a .key file, for example, yourdomain.com.crt and yourdomain.com.key.

Generate a private key.

```bash
openssl genrsa -out yourdomain.com.key 4096
```
Generate a certificate signing request (CSR).
Adapt the values in the -subj option to reflect your organization. If you use an FQDN to connect your Harbor host, you must specify it as the common name (CN) attribute and use it in the key and CSR filenames.

```bash
openssl req -sha512 -new \
    -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=yourdomain.com" \
    -key yourdomain.com.key \
    -out yourdomain.com.csr
```

Generate an x509 v3 extension file.

```
cat > v3.ext <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1=harbor.fritz.box
DNS.2=harbor
DNS.3=localhost
EOF
```



Use the v3.ext file to generate a certificate for your Harbor host.

Replace the yourdomain.com in the CRS and CRT file names with the Harbor host name.

```
openssl x509 -req -sha512 -days 3650 \
    -extfile v3.ext \
    -CA ca.crt -CAkey ca.key -CAcreateserial \
    -in harbor.fritz.box.csr \
    -out harbor.fritz.box.crt
```





+ [Deploying Harbor to Kubernetes | VMware Tanzu Developer Center](https://tanzu.vmware.com/developer/guides/harbor-gs/)


To install cert-manager, this guide uses Helm, for uniformity. As with installing Project Contour, Helm will not create a namespace, so the first step is to create one.
```bash
kubectl create namespace cert-manager
```



```bash
helm repo add jetstack https://charts.jetstack.io
```


```bash
helm repo update
```


```bash
helm install cert-manager jetstack/cert-manager --namespace cert-manager --version v1.0.2 --set installCRDs=true
```

Finally, wait for the Pods to become READY before moving on to the next step. This should only take a minute or so.

```bash
watch kubectl get pods -n cert-manager
```

setup “challenge record" in the solvers section, which allows Let’s Encrypt to verify that the certificate it is issuing is really controlled by you.

```
cat << EOF > letsencrypt-staging.yaml
apiVersion: cert-manager.io/v1alpha2
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    email: $EMAIL_ADDRESS
    privateKeySecretRef:
      name: letsencrypt-staging
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    solvers:
    - http01:
        ingress:
          class: contour
EOF
```


Now apply the file.
```bash      
kubectl apply -f letsencrypt-staging.yaml
```

```bash      
kubectl get clusterissuers.cert-manager.io
```

After staging certificate, you can install Harbor, for which this guide uses Helm in combination with the Bitnami repo set up earlier. As with your other install steps, the first thing to do is create the namespace.
```
kubectl create namespace harbor
``

