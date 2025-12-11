# k8s-lab

Kubernetes lab with kubeadm and Calico CNI

## Running manifests inline

```shell
calicoctl create -f - <<EOF
# yaml content here
EOF
```

## calicoctl commands

calicoctl ipam show --show-blocks
calicoctl get bgpconfiguration

***

## 0 - Install `calicoctl`

Install `calicoctl`

```shell
curl -L https://github.com/projectcalico/calico/releases/download/v3.31.2/calicoctl-linux-amd64 -o calicoctl && \
    chmod +x ./calicoctl && \
    sudo mv ./calicoctl /usr/local/bin
```

Save the kubeconfig to your home directory

```shell
sudo cat /etc/rancher/k3s/k3s.yaml > $(HOME)/.kube/config
```


## 1 - Enable BGP with filter

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

## 2 - NAT outgoing pod connections

```shell
calicoctl apply -f - <<EOF
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
  name: default-ipv4-ippool
spec:
  cidr: 192.168.0.0/16
  natOutgoing: true
EOF
```
