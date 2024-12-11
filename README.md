# kubernates
## Tutorial de Kurbenates
![image](https://github.com/Jilosofo/kubernates/assets/126982541/9f53f386-d0dd-4a47-8818-c6eead19b6a3)

*******
`Era da implantação tradicional:` No início, as organizações executavam aplicações em servidores físicos. Não havia como definir limites de recursos para aplicações em um mesmo servidor físico, e isso causava problemas de alocação de recursos. Por exemplo, se várias aplicações fossem executadas em um mesmo servidor físico, poderia haver situações em que uma aplicação ocupasse a maior parte dos recursos e, como resultado, o desempenho das outras aplicações seria inferior. Uma solução para isso seria executar cada aplicação em um servidor físico diferente. Mas isso não escalava, pois os recursos eram subutilizados, e se tornava custoso para as organizações manter muitos servidores físicos.

*******
`Era da implantação virtualizada:` Como solução, a virtualização foi introduzida. Esse modelo permite que você execute várias máquinas virtuais (VMs) em uma única CPU de um servidor físico. A virtualização permite que as aplicações sejam isoladas entre as VMs, e ainda fornece um nível de segurança, pois as informações de uma aplicação não podem ser acessadas livremente por outras aplicações.

A virtualização permite melhor utilização de recursos em um servidor físico, e permite melhor escalabilidade porque uma aplicação pode ser adicionada ou atualizada facilmente, reduz os custos de hardware e muito mais. Com a virtualização, você pode apresentar um conjunto de recursos físicos como um cluster de máquinas virtuais descartáveis.

Cada VM é uma máquina completa que executa todos os componentes, incluindo seu próprio sistema operacional, além do hardware virtualizado.

*******
`Era da implantação em contêineres:` Contêineres são semelhantes às VMs, mas têm propriedades de isolamento flexibilizados para compartilhar o sistema operacional (SO) entre as aplicações. Portanto, os contêineres são considerados leves. Semelhante a uma VM, um contêiner tem seu próprio sistema de arquivos, compartilhamento de CPU, memória, espaço de processo e muito mais. Como eles estão separados da infraestrutura subjacente, eles são portáveis entre nuvens e distribuições de sistema operacional.
*******
## Closter kuerbenates
### Control Plane:
* **etcd:** Guarda tudo em relação ao cluster tudo que informação sensivel do cluster Importante efetuar sempre backup onde salva todas informações do cluster (pensar em uma estrategia de redundacia.)
* **Controler manager:** Gerente do controler, ele tem acesso ao etcd ex:(ele verifica se aplicação realmente está rodando de acordo qualquer coisa ele reclama pro etcd) 
* **scherduler:** Responsavel verificar onde e melhor colocar cada container ele e resposavel verificar melhor qual hardware com melhor recurso. Pra onde vai seu container. 
* **apiserver** Unica aplicação responsavel de conversar com etcd somente ele conversa com etcd
### Wokers (Orquestrador do piano) 
![clusterkubernates](https://github.com/Jilosofo/kubernates/assets/126982541/6c0ef248-995f-487f-8166-e793480184a2)


## K3S

### Intalação server k3s
Deploy a Kubernetes Cluster with K3S
==================

Please check out the official [K3S Doccumentation](https://docs.k3s.io/) for more detailed instructions

Install the server
------------
Login to the server node and install K3S

```sh
$ ssh root@node1

# Make sure your nodes have unique hostnames, set the hostname if your nodes are all named localhost or something

$ hostnamectl set-hostname node1
$ echo node1 > /etc/hostname

# Install K3S using the install script
$ curl -sfL https://get.k3s.io | sh -
```


Verify the installation

```sh
# Make sure you have the /usr/local/bin directory in your path to directly access the installed binaries

$ export PATH=/usr/local/bin:$PATH
$ k3s kubectl get nodes

$ kubectl get nodes
$ kubectl get pods -A

```

Deploy test app
------------
```sh
$ kubectl apply -f web-app.yml

# Verify
$ kubectl get pods,svc
$ kubectl -n kube-system get ds
$ kubectl -n kube-system get pods
```

Customise the server configuration
------------
```sh
# Create the configuration file
$ cat <<EOF > /etc/rancher/k3s/config.yaml
cluster-init: true
EOF
# Restart k3s
$ systemctl restart k3s
$ kubectl get nodes
```

Install MetalLB
------------
Disable Service LB
```sh
# Edit the configuration file
$ cat /etc/rancher/k3s/config.yaml
cluster-init: true
disable: servicelb


# Restart k3s
$ systemctl restart k3s
```

Create the metallb manifest

```sh
$ cat <<EOF > /var/lib/rancher/k3s/server/manifests/metallb.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: metallb-system
---
apiVersion: helm.cattle.io/v1
kind: HelmChart
metadata:
  name: metallb
  namespace: metallb-system
spec:
  repo: https://metallb.github.io/metallb
  chart: metallb
  targetNamespace: metallb-system

---
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: pool-1
  namespace: metallb-system
spec:
  addresses:
  - 172.20.0.181-172.20.0.185

---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: k3s-l2
  namespace: metallb-system
spec:
  ipAddressPools:
  - pool-1
EOF

#Verify
$ kubectl -n metallb-system get pods
```

Delete and re-create web-app service

```sh
$ kubectl delete svc web-app
$ kubectl apply -f web-app.yml

# Verify metallb externalIP
$ kubectl get svc
$ curl 172.20.0.182
```

Install NGINX Ingress Controller
------------
Disable Traefik Ingress Controller
```sh
# Edit the configuration file
$ cat /etc/rancher/k3s/config.yaml
cluster-init: true
disable: servicelb
disable: traefik


# Restart k3s
$ systemctl restart k3s
```

Create the nginx ingress manifest

```sh
$ cat <<EOF > /var/lib/rancher/k3s/server/manifests/nginx-ingress.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: ingress-nginx
---
apiVersion: helm.cattle.io/v1
kind: HelmChart
metadata:
  name: ingress-nginx
  namespace: ingress-nginx
spec:
  repo: https://kubernetes.github.io/ingress-nginx
  chart: ingress-nginx
  targetNamespace: ingress-nginx
  valuesContent: |-
    controller:
      image:
        tag: "v1.8.1"
      service:
        type: LoadBalancer
EOF

#Verify
$ kubectl -n ingress-nginx get pods
```

Configure a Private Registry
------------
Create the registries configuration
```sh
# Create the registries.yaml file
$ cat <<EOF > /etc/rancher/k3s/registries.yaml
mirrors:
  registry.home-k8s.lab:
    endpoint:
      - "https://registry.home-k8s.lab"

configs:
  "registry.home-k8s.lab":
    auth:
      username: admin
      password: Harbor12345
    tls:
      insecure_skip_verify: true
EOF

# Restart k3s
$ systemctl restart k3s
```

Deploy test app to use the private registry

```sh
$ kubectl apply -f web-app2.yml

# Verify
$ kubectl get pods,svc
$ kubectl describe pod web-app2-k25r8
```


Configure default storage
------------
Refer to the local-path-provisioner [docs](https://github.com/rancher/local-path-provisioner/blob/master/README.md#usage) for more configuration options. 

Set the default local storage path
```sh
# Edit the configuration file
$ cat /etc/rancher/k3s/config.yaml
cluster-init: true
disable: servicelb
disable: traefik
default-local-storage-path: /mnt/disk1

# Restart k3s
$ systemctl restart k3s
```

Set additional configurations
```sh
# Edit the local-path-config configmap
$ kubectl -n kube-system edit configmap local-path-config

# Restart the local-path-provisioner deployment
$ kubectl -n kube-system rollout restart deploy local-path-provisioner
```

Test the storage
```sh
# Apply the example pvc and pod manifests
$ kubectl create -f pvc.yaml
$ kubectl create -f pod.yaml

#Verify
$ kubectl get pod
$ kubectl get pv
$ kubectl get pvc
```

Adding a control plane node
------------

Retrieve token from etcd capable server node
```sh
$ cat /var/lib/rancher/k3s/server/token
```

Login to new server node and install K3S
```sh
$ ssh root@node2


$ curl -fL https://get.k3s.io | sh -s - server --token <token> --disable-etcd --server https://node1:6443 
```

Persist the configuration by creating a config.yaml file
```sh
# Create the configuration file
$ cat <<EOF > /etc/rancher/k3s/config.yaml
disable: servicelb
disable: traefik
default-local-storage-path: /mnt/disk1
disable: etcd
EOF

# Restart k3s
$ systemctl restart k3s
```

Verify that the new node was added to the cluster
```sh
# Run kubectl get nodes on any server node
$ kubectl get nodes
```

Adding an agent node
------------

Retrieve token from etcd capable server node
```sh
$ cat /var/lib/rancher/k3s/server/token
```

Login to new server node and install K3S
```sh
$ ssh root@node3


$ curl -sfL https://get.k3s.io | K3S_URL=https://node1:6443 K3S_TOKEN=node1token sh -
```

Persist the configuration by creating a config.yaml file
```sh
# Create the configuration file
$ cat <<EOF > /etc/rancher/k3s/config.yaml
disable: servicelb
disable: traefik
default-local-storage-path: /mnt/disk1
EOF

# Restart k3s
$ systemctl restart k3s-agent
```

Verify that the new node was added to the cluster
```sh
# Run kubectl get nodes on any server node
$ kubectl get nodes
```

Uninstalling K3S
------------

Retrieve token from etcd capable server node
```sh
# On server nodes
$ /usr/local/bin/k3s-uninstall.sh

# On agent nodes
$ /usr/local/bin/k3s-agent-uninstall.sh
```




====================================
``curl -sfl https://get.k3s.io | sh -s - server \
--datastore-enpoint="mysql://k3s:Senha#123@tcp(195.154.76.101:51986)/k3s_db" --node-taint CriticalAddonsOnly=true:NoExecute --tls-san IP-loadbalace ou DNS``

--tls-san adicionar load balace <br>
--node-taint CriticalAddonsOnly=true:NoExecute => Esse server não vai executar os pods  


