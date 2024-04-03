# Kubernetes Homelab usando Kubespray, Ubuntu 22.04 e Proxmox

Dica de configuração / ajuste:

Durante o processo de configuração ocorreu uma falha no modprobe do IPVS.

Isso ocorre porque no Ubuntu Cloud Image não tem o IPVS, conforme explicado aqui: https://github.com/kubernetes-sigs/kubespray/issues/10602

Para corrigir basta alterar o kube_proxy_mode: iptables

```
vim inventory/mycluster/group_vars/k8s_cluster/k8s-cluster.yml
# Kube-proxy proxyMode configuration.
# Can be ipvs, iptables <-
# kube_proxy_mode: ipvs # Original version
kube_proxy_mode: iptables # Custom version: 04/02/2024

```

E sucesso: cluster configurado!!!

![image](https://github.com/zecaoliveira/kubespray-pve-k8s/assets/42525959/b514a70e-ed84-40fe-9fdd-234b59dbb8d6)
