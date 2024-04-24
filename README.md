# CipherTrust Transparent Encryption for Kubernetes (CTE-K8s)

Arquivos de configuração básica para integração do CTE-K8s a um nó do K8s.

O CTE-K8s utiliza o CTE-Userspace (CTE-U).

Os arquivos definidos neste repositório são utilizados para configurar os elementos que participam no deployment do CTE-K8s em um ambiente cujo Volume Persistente(PV) é um compartilhamento NFS.

Resumidamente, o CTE-K8s expõe o seu próprio Storage Class(SC) para os pods finais. Isso significa que o PV original deve ser apontado para o CTE-K8s. Dessa forma, qualquer operação de criptografia e descriptografia vai passar pelo agente, e será possível realizar o controle de acesso.

## Recursos Adicionais

* [Acervo de recursos em vídeo utilizado pela suporte da fabricante para auxílio na configuração de ambientes de laboratório/POC](https://vimeo.com/user22518097)

## Setup K8s

O host para os efeitos desse repositório deve conter o seguinte software:

* Servidor NFS
* Docker
* Helm
* kubectl
* minikube

```bash
systemctl enable --now docker
minikube start
```

## Setup CTE-K8s

Com o cluster K8s funcionando, faça o deploy do agente CTE-K8s:

```bash
git clone https://github.com/thalescpl-io/ciphertrust-transparent-encryption-kubernetes
cd ciphertrust-transparent-encryption-kubernetes/
./deploy.sh
```

## Setup NFS

1. Instale os pacotes que provêem o servidor nfs:

* nfs-utils (RHEL)
* nfs-kernel-server (Ubuntu)

Consulte os repositórios da distro.

2. Configure os diretórios a serem exportados via NFS. Edite o arquivo:`/etc/exports`):
```
/nfs *(rw, async, no_root_squash, no_subtree_check)
```

3. Exporte os diretórios:

```bash
sudo exportfs -a
```

4. Habilite e execute o serviço NFS:

```bash
Ubuntu: systemctl enable --now nfs-kernel-server
RHEL: systemctl enable --now nfs-server
```

## Setup CipherTrust Manager

1. Crie um K8s Storage Group
```
Name: k8s-cte-sg
K8s Namespace: default
K8s StorageClass: k8s-cte-sc
```

2. Crie uma politica `k8s-cte-policy` do tipo **CTE for Kubernetes** e o atribua ao grupo `k8s-cte-sg` em GuardPolicy

3. Crie um registration token ilimitado


## Uso

1. Faça as alterações necessárias em `pv.yaml` para configurar o acesso ao NFS pelo K8s e aplique a configuração no K8s:
```bash
kubectl apply -f pv.yaml
kubectl apply -f pvc.yaml
```
Isso irá criar o PV e o PVC no cluster K8s.

2. Coloque o registration token da sua CM em `cte-csi-cmtoken.yaml` e aplique a configuração no K8s:
```bash
kubectl apply -f cte-csi-cmtoken.yaml
```
**O registration token deve ser inserido no arquivo em Base64**
Isso irá criar o Secret no cluster K8s.

3. Faça as alterações necessárias nos campos do `cte-storageclass.yaml` e a configuração do SC do CTE:
```bash
kubectl apply -f cte-storageclass.yaml
```
Isso irá criar o SC no cluster K8s.

4. Faça as alterações ncessárias em `cte-csi-claim.yaml` e aplique a configuração:
```bash
kubectl apply -f cte-csi-claim.yaml
```

Isso irá criar o PVC que será utilizado pelos pods para acesso ao volume com criptografia

5. Crie o pod com acesso ao volume criptogrado:

```bash
kubectl apply -f ububuntu-pod.yaml
```
