# k8s-lab

Kubernetes lab with k3s and Calico CNI.

## Install k3s without a CNI

Server node. `calicoctl` will expect the config in *$HOME/.kube/config*

```shell
my_ip='192.168.128.TODO'
curl -sfL https://get.k3s.io | sh -s - server \
  --cluster-init \
  --flannel-backend=none \
  --disable-network-policy \
  --cluster-cidr=172.16.0.0/16 \
  --disable=traefik \
  --node-ip $my_ip \
  --node-external-ip $my_ip \
  --write-kubeconfig-mode=0644

mkdir -p $HOME/.kube && sudo cat /etc/rancher/k3s/k3s.yaml > $HOME/.kube/config
sudo cat /var/lib/rancher/k3s/server/node-token
```

Agent nodes

```shell
token='REDACTED'
server_ip='192.168.128.TODO'
my_ip='192.168.128.TODO'

curl -sfL https://get.k3s.io | \
  K3S_URL=https://$server_ip:6443 sh -s - agent \
  --token $token \
  --node-ip $my_ip \
  --node-external-ip $my_ip
```

Additional server nodes

```shell
token='REDACTED'
server_ip='192.168.128.TODO'
my_ip='192.168.128.TODO'

curl -sfL https://get.k3s.io | \
  K3S_URL=https://$server_ip:6443 sh -s - server \
  --token $token \
  --node-ip $my_ip \
  --node-external-ip $my_ip \
  --write-kubeconfig-mode=0644

mkdir -p $HOME/.kube && sudo cat /etc/rancher/k3s/k3s.yaml > $HOME/.kube/config
```

## Install Calico CNI
Refer to the [latest docs](https://docs.tigera.io/calico/latest/getting-started/kubernetes/k3s/quickstart).

Install the Calico operator and CRDs

```shell
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.31.2/manifests/operator-crds.yaml
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.31.2/manifests/tigera-operator.yaml
```

Install Calico, changing the manifest with the correct pod IP. After all the pods show the status of `Running`, you can exit `watch`.

```shell
export pod_network="172.16.0.0/14"

curl -s -o custom-resources.yaml \
  https://raw.githubusercontent.com/projectcalico/calico/v3.31.2/manifests/custom-resources.yaml && \
  sed -i "s|192\.168\.0\.0/16|${pod_network}|g" custom-resources.yaml
kubectl create -f custom-resources.yaml && watch kubectl get pods --all-namespaces
```

***

## 0 - Install `calicoctl`

Install `calicoctl` on every server node

```shell
curl -L https://github.com/projectcalico/calico/releases/download/v3.31.2/calicoctl-linux-amd64 -o calicoctl && \
    chmod +x ./calicoctl && \
    sudo mv ./calicoctl /usr/local/bin
```

Useful calicoctl commands

```shell
calicoctl ipam show --show-blocks
calicoctl get bgpconfiguration
```

## Enable BGP with filter

```shell
calicoctl apply -f - <<EOF
---
# Calico BGP configuration
apiVersion: projectcalico.org/v3
kind: BGPConfiguration
metadata:
  name: default
spec:
  serviceClusterIPs:
    - cidr: 192.168.1.0/24  # Advertise service IPs via BGP
  asNumber: 64555  # Your cluster's ASN
---
# create a BGP filter
apiVersion: projectcalico.org/v3
kind: BGPFilter
metadata:
  name: my-filter
spec:
  exportV4:
    - action: Accept
      matchOperator: In
      cidr: 192.168.1.0/24
      prefixLength:
        min: 24
        max: 32
    - action: Reject
      matchOperator: In
      cidr: 172.16.0.0/12
---
# BGP Peer to your firewall
apiVersion: projectcalico.org/v3
kind: BGPPeer
metadata:
  name: fortigate
spec:
  peerIP: 192.168.128.1  # Your firewall's IP
  asNumber: 64550
  filters:
    - my-filter
EOF
```
