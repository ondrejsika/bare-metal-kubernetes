[Ondrej Sika](https://ondrejsika.com)

# Bare Metal Kubernetes

    Ondrej Sika <ondrej@ondrejsika.com>
    https://github.com/ondrejsika/bare-metal-kubernetes

---

## About Course

### Kubernetes training in Czech Republic

- [ondrej-sika.cz/skoleni/kubernetes](https://ondrej-sika.cz/skoleni/kubernetes?_s=gh-dte)
- [skoleni-kubernetes.cz](https://skoleni-docker.cz/?_s=gh-kibm)

### Docker training in Europe

- [ondrej-sika.com/training/kubernetes](https://ondrej-sika.com/training/kubernetes?_s=gh-kibm)

### Related Courses

- Introduction to Kubernetes - [ondrejsika/kubernetes-training](https://github.com/ondrejsika/kubernetes-training) (on Github)

### Any Questions?

Write me mail to <ondrej@ondrejsika.com>

---

## Install Docker

```
curl -fsSL https://ins.oxs.cz/docker.sh | sudo sh
```

## Install Kubernetes (kubeadm, kubectl)

```
curl -fsSL https://ins.oxs.cz/kubernetes.sh | sudo sh
```

## Install NFS client (for NFS Persistant Volume Claims)

```
sudo apt install nfs-common
```

## Create a Master

Kubernetes doesn't run if you have swap on. You can disable it using:

```
swapoff -a
```

Now you can setup master

```
kubeadm init --pod-network-cidr=10.244.0.0/16
```

## Configure kubect

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## Join nodes

You can also disable swap, if it's on:

```
swapoff -a
```

```
kubeadm join <ip>:6443 --token <token> --discovery-token-ca-cert-hash <ca-hash>
```

## Master only

If you want run cluster with just one server (master only), you have to allow to run containers on master, which is not recommended.

```
kubectl taint nodes --all node-role.kubernetes.io/master-
```


## Install Flannel network

```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml
```

or

```
kubectl apply -f kube-flannel.yml
```

## Check Nodes

```
kubectl get nodes
```

## Install Dashboard

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
```

or

```
kubectl apply -f kubernetes-dashboard.yaml
```

## Add Dashboard User

```
kubectl apply -f https://raw.githubusercontent.com/ondrejsika/kubernetes-install-bare-metal/master/dashboard-user.yml
```

```
kubectl apply -f dashboard-user.yml
```

## Get Dashboard Token

```
kubectl -n kube-system describe serviceaccounts admin-user
kubectl -n kube-system describe secret <admin-user-token>
```

Or in one line

```
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | cut -f1 -d ' ') | grep -E '^token' | cut -f2 -d':' | tr -d '\t'
```

## Run kubectl proxy

```
kubectl proxy
```

Open <http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/login>

## Create Traefik LoadBalancer (for Ingress)

```
kubectl apply -f https://raw.githubusercontent.com/containous/traefik/v1.7/examples/k8s/traefik-rbac.yaml
kubectl apply -f https://raw.githubusercontent.com/containous/traefik/v1.7/examples/k8s/traefik-ds.yaml
```

or

```
kubectl apply -f traefik-rbac.yaml
kubectl apply -f traefik-ds.yaml
```

## Install Helm Client

Docs <https://github.com/helm/helm/blob/master/docs/install.md>

Or oneliner for Linux:

```
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get | bash
```

Or on Windows:

```
choco install kubernetes-helm
```

## Create Service Account for Tiller

```
kubectl create serviceaccount --namespace kube-system tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'
```

## Install NFS Client Provisioner (using Helm)

```
helm install stable/nfs-client-provisioner --set nfs.server=<nfs-server> --set nfs.path=<exported-path>
```
