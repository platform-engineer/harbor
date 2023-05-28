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



## instal k3c

+ [K3s](https://k3s.io/)

```bash
curl -sfL https://get.k3s.io | sh - 
```

Check for Ready node, takes ~30 seconds 
```bash
sudo k3s kubectl get node
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

```

```bash

```
