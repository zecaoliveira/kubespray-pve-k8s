# Kubernetes Homelab usando Kubespray, Ubuntu 22.04 e Proxmox

Seguindo a documentação do Pradeep Kumar funcionou: https://www.linuxtechi.com/install-kubernetes-using-kubespray/

Dica de configuração / ajuste:

Durante o processo de configuração ocorreu uma falha no modprobe com referência ao IPVS.

Isso ocorre porque no Ubuntu Cloud Image não tem o IPVS, conforme explicado aqui: https://github.com/kubernetes-sigs/kubespray/issues/10602

Para corrigir basta alterar o "kube_proxy_mode" de "ipvs" para "iptables":

```
$ vim inventory/mycluster/group_vars/k8s_cluster/k8s-cluster.yml
# Kube-proxy proxyMode configuration.
# Can be ipvs, iptables <-
# kube_proxy_mode: ipvs # Original version
kube_proxy_mode: iptables # Custom version: 04/02/2024

```

E sucesso: cluster configurado e funcionando!!!

![image](https://github.com/zecaoliveira/kubespray-pve-k8s/assets/42525959/b514a70e-ed84-40fe-9fdd-234b59dbb8d6)

Verificando a configuração a partir do "controller":

![image](https://github.com/zecaoliveira/kubespray-pve-k8s/assets/42525959/02a1d250-a9bc-400a-92f4-e1e1465342c9)

Implantei uma aplicação NGINX para testar o cluster:

```
$ kubectl create deployment demo-nginx-kubespray --image=nginx --replicas=2
$ kubectl expose deployment demo-nginx-kubespray --type NodePort --port=80
$ kubectl get  deployments.apps
$ kubectl get pods
$ kubectl get svc demo-nginx-kubespray
```

Resultado:

![image](https://github.com/zecaoliveira/kubespray-pve-k8s/assets/42525959/f8641d5b-cd89-4389-a2cc-58b4f28d0c43)

![image](https://github.com/zecaoliveira/kubespray-pve-k8s/assets/42525959/0ed46dd9-aaa3-443b-963e-bdac3fb35fff)




