---
layout: post
title: "Use Contiv-VPP CNI on K3s"
published: true
tags: [Kubernetes, Networking]
categories: [Kubernetes]
---

Contiv-VPP is an alternative to have use-space data plane for K3s. And I tried to install Contiv-VPP on K3s with default configuration. When I finished installation, realized Pods on K3s failed to create Pod networking interface with Contiv-VPP CNI. After some investigation on Contiv-VPP along with source code. I realized `containerd` as default container runtime of K3s was not supported by Contiv-VPP due to naming of network namespace is different between `containerd` and `docker` ([here](https://github.com/rancher/k3s/blob/03f05f93370f636fd3c5162a06fee54e40f9dd91/vendor/github.com/containerd/cri/pkg/netns/netns_unix.go#L72) and [here](https://github.com/contiv/vpp/blob/c6ed55900e77dd14b8705dc6fa6d90f7a8b70b56/plugins/ipnet/pod.go#L1128)). So `docker` is required as container runtime if you would like to enable Contiv-VPP as CNI. By default, `fannel` is used as CNI plugin on K3s. K3s provides a confiugration during installation: `--flannel-backend=none`. So the command to install K3s should be:
```
$ curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server --flannel-backend=none" sh -s - --docker
```
    
Since Contiv-VPP uses DPDK as its underlying library for process packets from NICs. Needs to make sure hugepages is enabled on host. The following command could be used to enable it (need to restart K3s):
```
$ sudo sysctl -w vm.nr_hugepages=512
```

Now, we are ready to install Contiv-VPP. Since the [node selector](https://github.com/contiv/vpp/blob/c6ed55900e77dd14b8705dc6fa6d90f7a8b70b56/k8s/contiv-vpp.yaml#L214) in `contiv-vpp.yaml` from Contiv-VPP didn't match [what K3s provides](https://github.com/rancher/k3s/blob/03f05f93370f636fd3c5162a06fee54e40f9dd91/pkg/server/server.go#L441). We need to change it before applying YAML file:
```
$ curl -sfL https://raw.githubusercontent.com/contiv/vpp/master/k8s/contiv-vpp.yaml \
    | sed 's#node-role.kubernetes.io/master: ""#node-role.kubernetes.io/master: "true"#' \
    | sudo /usr/local/bin/k3s kubectl apply -f -
```    

Few miniutes later, check the pods running on your K3s, and it should look like the following:
```
$ sudo /usr/local/bin/k3s kubectl get pods -n kube-system
NAME                                     READY   STATUS      RESTARTS   AGE
contiv-etcd-0                            1/1     Running     0          14h
contiv-crd-tb289                         1/1     Running     0          14h
contiv-vswitch-mfstf                     1/1     Running     0          14h
contiv-ksr-xl689                         1/1     Running     0          14h
helm-install-traefik-hzrts               0/1     Completed   0          14h
coredns-7944c66d8d-jv4rr                 1/1     Running     0          14h
local-path-provisioner-6d59f47c7-8s7qp   1/1     Running     0          14h
metrics-server-7566d596c8-l6hlh          1/1     Running     0          14h
svclb-traefik-8cfcf                      2/2     Running     0          14h
traefik-758cd5fc85-jv6j4                 1/1     Running     0          14h
```

# References
  - <https://rancher.com/docs/k3s/latest/en/installation/network-options/#custom-cni>
  - <https://rancher.com/docs/k3s/latest/en/advanced/#using-docker-as-the-container-runtime>
  - <https://github.com/contiv/vpp/blob/master/docs/setup/MANUAL_INSTALL.md>
