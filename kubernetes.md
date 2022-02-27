---
title: Kubernetes
author: Thibault Ayanides
theme: white
highlightTheme: stackoverflow-dark
width: 1920
revealOptions:
  transition: "none"
  slideNumber: true
  width: 1920
  height: 1080
---

<link href="css/style.css" rel="stylesheet">

# Kubernetes

<img src="img/kubernetes_logo.png"/>

---

## Sommaire

1. Contexte
2. Architecture
3. Workloads

---

## Contexte
<!-- .slide: class="big-slide" -->

- Créé par Google en 2014 à partir de leur propre solution, Borg
- Sert de base de création à la CNCF (<em>Cloud Native Computing Foundation</em>)
- Écrit en Golang
- Proposé dans des solutions de cloud publics (GCP, AWS, Azure, OVH, ...) et de clouds privés (Openshift, Rancher, ...)
- Plateforme d'orchestration de conteneur

<iframe data-src="https://www.cncf.io/projects/" width=100% height=100%></iframe>

---

## Orchestration de conteneurs, kesako ?

L'orchestration c'est :
- l'automatisation du déploiement
- l'automatisation de la montée en charge
- la mise en œuvre de conteneurs applicatifs sur des clusters de serveurs

---
## Ce que peut faire Kubernetes

- Répartisseur de charge et découverte de services
- Orchestration de stockage
- Rollback et rollout de nouvelles versions automatiques
- Optimisation des ressources
- Auto-réparation
- Gestion de configuration et de secrets applicatifs

---
# Architecture

---

## Control Plane
<!-- .slide: class="big-slide" -->

<img src="img/architecture.svg"/>

- Nodes master (abstraits dans les offres cloud publics managées)
- Nodes worker

---

## Composition du control plane
<!-- .slide: class="big-slide" -->

- <b>Etcd</b>
    - Persistence des données du cluster : clé/valeur distribuée, protocole RAFT
- <b>APIServer</b>
    - Serveur qui permet la configuration d’objet Kubernetes (Pods, Service, Replication Controller, ...)
    - Tous les communications du cluster passent par l’API Server
    - Gère l’authentification, l’autorisation, la validation de la demande, le contrôle d’admission
- <b>Controller Manager</b>
    - Gère les différents contrôleurs du cluster (NodeController, ReplicationController, EndpointController, ...)
- <b>Scheduler</b>
    - S’occupe de trouver un noeud pour déployer un POD

---

## Composition des nodes
<!-- .slide: class="big-slide" -->

- <b>Kubelet</b>
    - Crée et gère les conteneurs. Contacté par l’API Server
- <b>Kube-proxy</b>
    - Gère les règles réseaux sur chaque noeud
    - Forwarding TCP/UDP et Load balancing entre les services et les backends
- <b>Container Runtime Interface (CRI)</b>
    - Exécute les conteneurs : Containerd (default), Cri-o, gVisor, Kata, Docker (Deprecated)
- <b>Container Network Interface (CNI)</b>
    - Allocation des adresses IP des conteneurs
    - Gestion de l’interface réseau qui va porter le conteneur
    - Cilium (GKE v2), Flannel, Weave, ...
- <b>Container Storage Interface (CSI)</b>
    - Création, redimensionnement, attachement, montage [...] des volumes
    - Secrets

---

## Architecture

<img src="img/architecture2.png" width=50%/>

---

# Installation d'un cluster Kubernetes de test

---
## Outillages

- Kubectl
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
mkdir -p ~/bin
mv ./kubectl ~/bin/kubectl
# Append ~/bin to your PATH
# Install bash or zsh completions
kubectl completion bash
```
- Vagrant
```bash
curl -LO "https://releases.hashicorp.com/vagrant/2.2.19/vagrant_2.2.19_linux_amd64.zip"
unzip vagrant_2.2.19_linux_amd64.zip
chmod +x vagrant
mv vagrant ~/bin
rm vagrant_2.2.19_linux_amd64.zip
```

- Virtualbox

---

## Installation de Kubernetes

- Configuration Virtualbox
```bash
sudo mkdir -p /etc/vbox/
echo * 0.0.0.0/0 ::/0 | sudo tee -a /etc/vbox/networks.conf
```

- Installation via Vagrant
```bash
git clone https://github.com/scriptcamp/vagrant-kubeadm-kubernetes.git
cd vagrant-kubeadm-kubernetes
vagrant up
```

- Copie des creds
```bash
cp config ~/.kube/
```

---

## Vérification de l'installation

- Apiserver
```bash
vagrant ssh
kubectl cluster-info
kubectl get nodes
```

- Dashboard
```bash
TOKEN=$(kubectl -n kube-system describe secret default| awk '$1=="token:"{print $2}')
kubectl config set-credentials kubernetes-admin --token="${TOKEN}"
kubectl proxy
# http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/workloads?namespace=default et se logger avec .kube/config
```
