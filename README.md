# ATELIER FROM IMAGE TO CLUSTER

## Objectif

L'objectif de cet atelier est d'industrialiser le cycle de vie d'une
application simple en utilisant des outils d'Infrastructure as Code.\
Nous avons créé une image Docker personnalisée avec Packer (basée sur
Nginx) contenant un fichier HTML personnalisé, puis nous avons déployé
cette application sur un cluster Kubernetes (k3d).

------------------------------------------------------------------------

## Technologies utilisées

-   Docker
-   Packer
-   Kubernetes (k3d)
-   Ansible
-   GitHub Codespaces

------------------------------------------------------------------------

## Architecture

-   Image Docker personnalisée (Nginx + index.html)
-   Cluster Kubernetes (1 master + 2 workers)
-   Déploiement sur Kubernetes
-   Accès via port-forward

------------------------------------------------------------------------

## Étapes réalisées

### 1. Création du Codespace

Depuis GitHub : - Cliquer sur **Code** - Ouvrir un **Codespace** - Accès
à un terminal Linux prêt à l'emploi

------------------------------------------------------------------------

### 2. Installation des outils

sudo apt update\
sudo apt install -y ansible

Installation de Packer :

wget
https://releases.hashicorp.com/packer/1.10.0/packer_1.10.0_linux_amd64.zip\
unzip packer_1.10.0_linux_amd64.zip\
sudo mv packer /usr/local/bin/

------------------------------------------------------------------------

### 3. Création du cluster Kubernetes (k3d)

curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh \|
bash

k3d cluster create lab --servers 1 --agents 2

kubectl get nodes

------------------------------------------------------------------------

### 4. Création de l'image avec Packer

Fichier nginx.pkr.hcl :

packer { required_plugins { docker = { version = "\>= 1.0.0" source =
"github.com/hashicorp/docker" } } }

source "docker" "nginx" { image = "nginx:latest" commit = true }

build { name = "nginx-custom" sources = \["source.docker.nginx"\]

provisioner "file" { source = "index.html" destination =
"/usr/share/nginx/html/index.html" } }

Commande :

packer init .\
packer build .

------------------------------------------------------------------------

### 5. Import dans k3d

k3d image import nginx-custom:latest -c lab

------------------------------------------------------------------------

### 6. Déploiement sur Kubernetes

kubectl create deployment nginx-custom --image=nginx-custom:latest

kubectl patch deployment nginx-custom -p
'{"spec":{"template":{"spec":{"containers":\[{"name":"nginx-custom","imagePullPolicy":"Never"}\]}}}}'

kubectl expose deployment nginx-custom --type=ClusterIP --port=80

------------------------------------------------------------------------

### 7. Accès à l'application

kubectl port-forward svc/nginx-custom 8080:80

------------------------------------------------------------------------

## Résultat

L'application est accessible via le navigateur et affiche la page HTML
personnalisée.



------------------------------------------------------------------------

## Conclusion

Ce projet montre comment automatiser : - La création d'image avec
Packer - Le déploiement Kubernetes - L'accès à une application web
