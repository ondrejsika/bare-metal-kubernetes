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
