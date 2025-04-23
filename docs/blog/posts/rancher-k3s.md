---
date:
  created: 2024-10-23
draft: true
---

# Como instalar o Rancher

Este tutorial irá lhe guiar nos passos necessários para a instalação e configuração de um cluster Kubernetes baseado na solução da SUSE.

Você será guiado na instalação dos seguintes componentes, todos homologados pelo Cloud Native Computing Foundation (CNCF):

1. ![cert-manager](../../assets/images/cert-manager-logo.svg){width="18em"} **cert-manager** - Gerenciador de certificados para K8s.  
2. ![k3s](../../assets/images/icon-k3s.svg){.k3s} **k3s** - Distribuição Kubernetes leve da SUSE.  
3. ![Longhorn](../../assets/images/icon-longhorn.svg){.longhorn} **Longhorn** - Solução distribuída de armazenamento em blocos para Kubernetes da SUSE.  
4. ![Rancher](../../assets/images/rancher-logo-cow-blue.svg){.rancher} **Rancher** - Plataforma de gerenciamento de clusters Kubernetes da SUSE.  

<!-- more -->

!!! warning "Atenção"

    O ambiente montado através dos passos mostrados neste tutorial não deve ser visto como pronto para uso em produção.  
    O objetivo deste tutorial é viabilizar a apresentação da solução da SUSE, permitindo que se teste seus recursos na prática.

## Público alvo

Profissionais da área de desenvolvimento de sistemas e infraestrutura de TIC.

Para melhor aproveitamento do tutorial, recomenda-se os seguintes conhecimentos prévios:

1. Contêineres;
2. DevOps;
3. Linux.

## Ambiente

### Versões

Tanto o Longhorn quanto o Rancher rodam sobre o k3s, que por sua vez será instalado sobre o RHEL.

Para definir as versões de cada componente, iniciaremos pelo **Rancher**, cuja versão estável é a **v2.9.4**.

Ela está homologada para uso com o **Longhorn v1.6.2**.

Ambos estão homologados para uso com **RHEL v9** e **k3s v1.30**.

> **Matriz de compatibilidade**  
> [Longhorn v1.6.x](https://www.suse.com/suse-longhorn/support-matrix/all-supported-versions/longhorn-v1-6-x/)  
> [Rancher Manager v2.9.x](https://www.suse.com/suse-rancher/support-matrix/all-supported-versions/rancher-v2-9-2/)  
> [k3s v1.30.x](https://www.suse.com/suse-k3s/support-matrix/all-supported-versions/k3s-v1-30/)  

### Requisitos

#### Hardware

Para criação do cluster Kubernetes, utilizaremos três nós físicos com a mesma configuração de hardware:

|        Modelo        |                   CPU                   |          GPU           | Memória |  HDD/SSD  |
| :------------------: | :-------------------------------------: | :--------------------: | :-----: | :-------: |
| HP Elite Mini 800 G9 | 12th Gen Intel i5-12500 (12) @ 4.600GHz | Intel UHD Graphics 770 |  16GB   | 1TB/256GB |

#### Software

##### Sistema operacional

Utilizaremos quatro nós com sistema operacional Red Hat Enterprise Linux 9.

!!! info "Observação"

    A instalação do sistema operacional não faz parte do escopo deste tutorial.

##### Configurações de rede

|  #  |  Hostname   |     Description     |      IP       |
| :-: | :---------: | :-----------------: | :-----------: |
|  1  | jumpbox.lab | Administration host | 10.5.132.6/24 |
|  2  |  node1.lab  |   Kubernetes node   | 10.5.132.7/24 |
|  3  |  node2.lab  |   Kubernetes node   | 10.5.132.8/24 |
|  4  |  node3.lab  |   Kubernetes node   | 10.5.132.9/24 |

!!! warning "Atenção"

    Os IPs devem ser fixos.

### Configuração dos nós

#### :material-desktop-tower: Nó 1

``` shell linenums="1" title="Os comandos a seguir devem ser executados pelo usuário root."
rhc connect
dnf --assumeyes --refresh update
dnf --assumeyes install sPACKAGESs
cat << EOF >> /etc/hosts
sNODE1_IPs node1.lab rancher.lab
sNODE2_IPs node2.lab rancher.lab
sNODE3_IPs node3.lab rancher.lab
EOF
cat << EOF > /etc/NetworkManager/conf.d/k3s-cni-flannel.conf
[keyfile]
unmanaged-devices=interface-name:flannel*
EOF
systemctl enable --now iscsid.service
systemctl disable --now firewalld.service
systemctl reboot
```

```asciinema-player
{ "file": "./assets/asciinema/prep.cast", "cols": 132, "rows": 24, "auto_play": true }
```

``` shell linenums="1" title="Os comandos a seguir devem ser executados pelo usuário root."
curl -sfL https://get.k3s.io | \
INSTALL_K3S_CHANNEL=sINSTALL_K3S_CHANNELs \
INSTALL_K3S_SKIP_START=sINSTALL_K3S_SKIP_STARTs \
K3S_CLUSTER_INIT=sK3S_CLUSTER_INITs \
K3S_TOKEN=sK3S_TOKENs sh -
systemctl --now enable k3s.service & journalctl --unit k3s.service --follow
```

#### :material-desktop-tower: Nó 2

``` shell linenums="1" title="Os comandos a seguir devem ser executados pelo usuário root."
rhc connect
dnf --assumeyes --refresh update
dnf --assumeyes install sPACKAGESs
cat << EOF >> /etc/hosts
sNODE1_IPs node1.lab rancher.lab
sNODE2_IPs node2.lab rancher.lab
sNODE3_IPs node3.lab rancher.lab
EOF
cat << EOF > /etc/NetworkManager/conf.d/k3s-cni-flannel.conf
[keyfile]
unmanaged-devices=interface-name:flannel*
EOF
systemctl enable --now iscsid.service
systemctl disable --now firewalld.service
systemctl reboot
```

``` shell linenums="1" title="Os comandos a seguir devem ser executados pelo usuário root."
curl -sfL https://get.k3s.io | \
INSTALL_K3S_CHANNEL=sINSTALL_K3S_CHANNELs \
INSTALL_K3S_EXEC=sINSTALL_K3S_EXECs \
INSTALL_K3S_SKIP_START=sINSTALL_K3S_SKIP_STARTs \
K3S_TOKEN=xK3S_TOKENx \
K3S_URL=sK3S_URLs sh -
systemctl enable --now k3s.service & journalctl --unit k3s.service --follow
```

#### :material-desktop-tower: Nó 3

``` shell linenums="1" title="Os comandos a seguir devem ser executados pelo usuário root."
rhc connect
dnf --assumeyes --refresh update
dnf --assumeyes install sPACKAGESs
cat << EOF >> /etc/hosts
sNODE1_IPs node1.lab rancher.lab
sNODE2_IPs node2.lab rancher.lab
sNODE3_IPs node3.lab rancher.lab
EOF
cat << EOF > /etc/NetworkManager/conf.d/k3s-cni-flannel.conf
[keyfile]
unmanaged-devices=interface-name:flannel*
EOF
systemctl enable --now iscsid.service
systemctl disable --now firewalld.service
systemctl reboot
```

``` shell linenums="1" title="Os comandos a seguir devem ser executados pelo usuário root."
curl -sfL https://get.k3s.io | \
INSTALL_K3S_CHANNEL=sINSTALL_K3S_CHANNELs \
INSTALL_K3S_EXEC=sINSTALL_K3S_EXECs \
INSTALL_K3S_SKIP_START=sINSTALL_K3S_SKIP_STARTs \
K3S_TOKEN=xK3S_TOKENx \
K3S_URL=sK3S_URLs sh -
systemctl --now enable k3s.service & journalctl --unit k3s.service --follow
```

#### :material-laptop: Jumpbox

```shell linenums="1" title="Os comandos a seguir devem ser executados pelo usuário root."
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm completion bash > /etc/bash_completion.d/helm
```

```shell linenums="1" title="Os comandos a seguir devem ser executados pelo usuário root."
helm repo add longhorn https://charts.longhorn.io --force-update
helm install \
longhorn longhorn/longhorn \
--namespace longhorn-system \
--create-namespace \
--version 1.7.2
```

```asciinema-player
{
    "file": "./assets/asciinema/longhorn.cast",
    "cols": 120,
    "rows": 17,
    "auto_play": true
}
```

```shell linenums="1" title="Os comandos a seguir devem ser executados pelo usuário root."
kubectl --namespace longhorn-system rollout status deploy
```

```shell linenums="1" title="Os comandos a seguir devem ser executados pelo usuário root."
helm repo add jetstack https://charts.jetstack.io --force-update
helm install \
cert-manager jetstack/cert-manager \
--namespace cert-manager \
--create-namespace \
--set crds.enabled=true
```

```shell title="Resultado da instalação."
NAME: cert-manager
LAST DEPLOYED: Tue Sep 17 10:37:52 2024
NAMESPACE: cert-manager
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
cert-manager v1.15.3 has been deployed successfully!

In order to begin issuing certificates, you will need to set up a ClusterIssuer
or Issuer resource (for example, by creating a 'letsencrypt-staging' issuer).

More information on the different types of issuers and how to configure them
can be found in our documentation:

https://cert-manager.io/docs/configuration/

For information on how to configure cert-manager to automatically provision
Certificates for Ingress resources, take a look at the `ingress-shim`
documentation:

https://cert-manager.io/docs/usage/ingress/
```

```shell linenums="1" title="Os comandos a seguir devem ser executados pelo usuário root."
kubectl --namespace cert-manager rollout status deploy/cert-manager-webhook
```

```shell linenums="1" title="Os comandos a seguir devem ser executados pelo usuário root."
helm repo add rancher-stable https://releases.rancher.com/server-charts/stable --force-update
helm install rancher rancher-stable/rancher \
--namespace cattle-system \
--create-namespace \
--set hostname=rancher.lab \
--set bootstrapPassword=bootStrapAllTheThings
```

```asciinema-player
{ "file": "./assets/asciinema/rancher.cast", "cols": 132, "rows": 35, "auto_play": true }
```

```shell linenums="1" title="Os comandos a seguir devem ser executados pelo usuário root."
kubectl --namespace cattle-system get deploy --watch
```
