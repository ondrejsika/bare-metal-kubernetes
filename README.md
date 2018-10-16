# Kubernetes Install on Bare Metal

    Ondrej Sika <ondrej@ondrejsika.com>
    https://github.com/ondrejsika/kubernetes-install-bare-metal

## Install Docker

```
curl -fsSL https://ins.oxs.cz/docker.sh | sudo sh
```

## Install Kubernetes (kubeadm, kubectl)

```
curl -fsSL https://ins.oxs.cz/kubernetes.sh | sudo sh
```

## Create a Master

```
kubeadm init --pod-network-cidr=10.244.0.0/16
```

## Konfigure kubect

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HO
```

## Fix `/etc/kubernetes/manifests/kube-controller-manager.yaml`

Add to end of `spec.controller.command`

```
    - --allocate-node-cidrs=true
    - --cluster-cidr=10.244.0.0/16
```

and reboot

```
reboot
```

## Join nodes

```
kubeadm join <ip>:6443 --token <token> --discovery-token-ca-cert-hash <ca-hash>
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
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```

or

```
kubectl apply -f kubernetes-dashboard.yaml
```

## Add Dashboard User

```
kubectl apply -f dashboard-user.yml
```

## Get Dashboard Token

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
kubectl apply -f https://raw.githubusercontent.com/containous/traefik/master/examples/k8s/traefik-rbac.yaml
kubectl apply -f https://raw.githubusercontent.com/containous/traefik/master/examples/k8s/traefik-deployment.yaml
kubectl apply -f https://raw.githubusercontent.com/containous/traefik/master/examples/k8s/traefik-ds.yaml
```

or

```
kubectl apply -f traefik-rbac.yaml
kubectl apply -f traefik-deployment.yaml
kubectl apply -f traefik-ds.yaml
```