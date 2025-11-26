# TP DevOps : CI/CD, GitOps, Orchestration et Monitoring

## 1. Contexte et Objectifs

Ce TP vise à mettre en place une **chaîne DevOps complète** pour automatiser le cycle de vie d’une application statique (CV One Page) déployée sur une VM Ubuntu Server 24.04 (**DEVOPS-LAB**).

### Objectifs :
- **Automatisation de l’infrastructure** : Ansible, Terraform  
- **Pipeline CI/CD** : Jenkins  
- **Orchestration & GitOps** : K3s + Argo CD  
- **Monitoring** : Grafana Cloud  
- **Gestion des images Docker et déploiement automatisé**

---

## 2. Architecture du projet

```
├── ansible/
│   ├── ansible.cfg
│   ├── inventory.yml
│   ├── playbooks.yml
│   └── roles/
│       ├── common/tasks/main.yaml
│       ├── docker/tasks/main.yaml
│       ├── jenkins/tasks/main.yaml
│       └── terraform/tasks/main.yaml
├── cvApp/
│   ├── Dockerfile
│   ├── index.html
│   └── Jenkinsfile
├── k8s/
│   ├── deployment.yaml
│   ├── node_exporter-1.6.1.linux-amd64/
│   ├── node_exporter-1.6.1.linux-amd64.tar.gz
│   └── service.yaml
└── terraform/
    ├── main.tf
    └── terraform.tfstate
```

---

## 3. Préparation de l’environnement

1. **Création de la VM** : Ubuntu Server 24.04 (**DEVOPS-LAB**)  
2. **Configuration SSH** avec clé publique pour Jenkins et administration  
3. **Mise à jour du système** :
```bash
sudo apt update && sudo apt upgrade -y
```

---

## 4. Automatisation avec Ansible

### Étapes :
- Mise à jour du système :
```bash
ansible-playbook -i ansible/inventory.yml ansible/playbooks.yml --tags update
```
- Installation de Docker :
```bash
ansible-playbook -i ansible/inventory.yml ansible/playbooks.yml --tags docker
```
- Installation de Terraform :
```bash
ansible-playbook -i ansible/inventory.yml ansible/playbooks.yml --tags terraform
```
- Installation de Jenkins :
```bash
ansible-playbook -i ansible/inventory.yml ansible/playbooks.yml --tags jenkins
```

---

## 5. Pipeline CI/CD avec Jenkins

### Fonctionnalités :
- **Checkout du code** depuis GitHub  
- **Détection des changements** toutes les 5 minutes  
- **Build de l’image Docker**  
- **Push sur Docker Hub**  
- **Mise à jour GitOps** des manifests Kubernetes  
- **Notifications Slack** en cas de succès/échec  
- **Nettoyage** des images et conteneurs obsolètes  

#### Exemple de commandes :
```bash
docker build -t azouztarek/moncv:${BUILD_NUMBER} .
docker tag azouztarek/moncv:${BUILD_NUMBER} azouztarek/moncv:latest
docker push azouztarek/moncv:${BUILD_NUMBER}
docker push azouztarek/moncv:latest

# Mise à jour GitOps
sed -i "s|image: .*|image: azouztarek/moncv:${BUILD_NUMBER}|g" k8s/deployment.yaml
git add deployment.yaml
git commit -m "Update image to azouztarek/moncv:${BUILD_NUMBER}"
git push
```

---

## 6. Déploiement avec Terraform

- Déploiement d’un conteneur Docker **moncv** basé sur l’image `azouztarek/moncv`  
- Port exposé : **8585**

```bash
cd terraform
terraform init
terraform apply
```

**Test d’accès :**
```bash
curl http://<IP_VM>:8585
```

---

## 7. Orchestration Kubernetes avec K3s et Argo CD

- **Installation K3s (Single Node)** :
```bash
curl -sfL https://get.k3s.io | sh -
```

- **Déploiement via Argo CD** :
  - Deployment : 2 replicas  
  - Service : NodePort  

**Test d’accès :**
```
http://<IP_VM>:<NODEPORT>
```

### GitOps :
Déploiement automatique à chaque mise à jour de l’image Docker.

---

---

## 🔹 Gestion multi-projets avec Git Submodules

Ce projet utilise **Git Submodules** pour regrouper plusieurs dépôts tout en conservant leur indépendance.

### 📂 Structure des submodules
- `cvApp` : Application statique + Dockerfile + Jenkinsfile
- `k8s` : Manifests Kubernetes (GitOps)
- `ansible` : Automatisation de l’infrastructure
- `terraform` : Infrastructure as Code

---

### ✅ Cloner le repo avec ses submodules
Pour récupérer le repo parent et tous les submodules :
```bash
git clone --recurse-submodules https://github.com/azouztarek/devops-lab.git

