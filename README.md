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

Para testar o acesso ao dashboard GUI siga as etapas abaixo:

1. Criar um ServiceAccount:

```
$ vim dashboard-adminuser.yml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system

$ kubectl apply -f dashboard-adminuser.yml
serviceaccount/admin-user created
```

2. Criar um ClusterRoleBinding:

```
$ vim admin-role-binding.yml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system

$ kubectl apply -f admin-role-binding.yml
clusterrolebinding.rbac.authorization.k8s.io/admin-user created
```

3. Agora crie um token para o "admin-user" e guarde para usar na etapa de acesso ao dashboard (7):

```
$ kubectl -n kube-system  create token admin-user
```
![image](https://github.com/zecaoliveira/kubespray-pve-k8s/assets/42525959/2c4fc2a8-87ba-4ecd-a4c2-d8f9ef8fe330)

4. Faça uma conexão da sua máquina para a do servidor "controller" do cluster Kubernetes e inicie o proxy:

```
$ ssh -L 8001:localhost:8001 sysops@172.16.0.41
$ sudo su -
# kubectl proxy
Starting to serve on 127.0.0.1:8001
```

5. Configure o proxy da sua máquina local com as configurações abaixo:

![image](https://github.com/zecaoliveira/kubespray-pve-k8s/assets/42525959/d18a152d-4b30-4965-b6d7-0d8e0cb3041f)

6. Copie e cole o endereço abaixo no mesmo navegador em que configurou o proxy da etapa número 5:
```
http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#/workloads?namespace=default
```

7. Na janela que vai surgir cole o token que você criou na etapa de número 3:

![image](https://github.com/zecaoliveira/kubespray-pve-k8s/assets/42525959/2b58236b-aba0-45a3-8704-565a80468db8)

8. E sucesso. Se tudo estiver funcionando você verá o dashboard do Kubernetes:

![image](https://github.com/zecaoliveira/kubespray-pve-k8s/assets/42525959/431462e4-4eb3-431b-9a3b-d254f6293203)

Enquanto o proxy for mantido ativo, conforme fizemos na etapa 4, o acesso ao dashboard vai funcionar.

Até o próximo laboratório.
