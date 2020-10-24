---
layout: post
title: "Use Contiv-VPP CNI on K3s"
published: true
tags: [Kubernetes, Networking]
categories: [Kubernetes]
---

By default, `fannel` is used as CNI plugin on K3s. K3s provides a confiugration during installation: `--flannel-backend=none`. Contiv-VPP is an alternative to have use-space data plane for K3s. Unfortunately, it only supports `docker` as container runtime due to compatibility issue on naming of network namespace. So if you would like to use Contiv-VPP as CNI on K3s, you might need to use the following command for our installation:

    curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server --flannel-backend=none" sh -s - --docker
    
Since Contiv-VPP uses DPDK as its underlying library for process packets from NICs. Needs to make sure hugepages is enabled on host. The following command could be used to enable it:

    sysctl -w vm.nr_hugepages=512
    
Now, we are ready to install Contiv-VPP. Since the default node selector from Contiv-VPP didn't match what K3s provides. We need to change it before applying YAML file:

    curl -sfL https://raw.githubusercontent.com/contiv/vpp/master/k8s/contiv-vpp.yaml | sed 's#node-role.kubernetes.io/master: ""#node-role.kubernetes.io/master: "true"#' | sudo /usr/local/bin/k3s kubectl apply -f -
    
Few miniutes later, check the pods running on your K3s:
```
$ sudo /usr/local/bin/k3s kubectl get pods -n kube-system -o wide
NAME                                     READY   STATUS      RESTARTS   AGE   IP              NODE      NOMINATED NODE   READINESS GATES
contiv-etcd-0                            1/1     Running     0          14h   10.206.66.222   centos7   <none>           <none>
contiv-crd-tb289                         1/1     Running     0          14h   10.206.66.222   centos7   <none>           <none>
contiv-vswitch-mfstf                     1/1     Running     0          14h   10.206.66.222   centos7   <none>           <none>
contiv-ksr-xl689                         1/1     Running     0          14h   10.206.66.222   centos7   <none>           <none>
helm-install-traefik-hzrts               0/1     Completed   0          14h   10.1.1.3        centos7   <none>           <none>
coredns-7944c66d8d-jv4rr                 1/1     Running     0          14h   10.1.1.2        centos7   <none>           <none>
local-path-provisioner-6d59f47c7-8s7qp   1/1     Running     0          14h   10.1.1.4        centos7   <none>           <none>
metrics-server-7566d596c8-l6hlh          1/1     Running     0          14h   10.1.1.5        centos7   <none>           <none>
svclb-traefik-8cfcf                      2/2     Running     0          14h   10.1.1.6        centos7   <none>           <none>
traefik-758cd5fc85-jv6j4                 1/1     Running     0          14h   10.1.1.7        centos7   <none>           <none>
```

# References
  - https://rancher.com/docs/k3s/latest/en/installation/network-options/
  - https://rancher.com/docs/k3s/latest/en/advanced/
  - https://github.com/contiv/vpp/blob/master/docs/setup/MANUAL_INSTALL.md
