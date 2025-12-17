# redis-cluster-k8s

# Déploiement Redis Cluster sur Kubernetes

Ce guide décrit pas-à-pas comment déployer un **Redis Cluster** sur un cluster Kubernetes avec Helm.

---

## PRÉREQUIS À VÉRIFIER

Avant de commencer, assurez-vous que votre cluster est prêt et que les outils nécessaires sont installés.

### ÉTAPE 1️ - Vérifier que le cluster Kubernetes est opérationnel
```
kubectl cluster-info
kubectl get nodes
```

### ÉTAPE 2 - Vérifier kubectl et Helm
```
kubectl version --client
helm version
```

### ÉTAPE 3 - Vérifier la présence d'une StorageClass
```
kubectl get storageclass
```
---

## DÉPLOIEMENT REDIS CLUSTER

### ÉTAPE 1️ - Créer un namespace dédié
```
kubectl create namespace redis
```
### ÉTAPE 2️ - Ajouter le repo Helm Bitnami et mettre à jour
```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo redis-cluster
```

### ÉTAPE 3️ - Créer le fichier redis-values.yaml
```
image:
  registry: docker.io
  repository: bitnami/redis
  tag: latest

cluster:
  nodes: 3
  replicas: 3

auth:
  enabled: true
  password: "Password123"

persistence:
  enabled: true
  size: 10Gi

resources:
  requests:
    cpu: 250m
    memory: 1Gi
  limits:
    cpu: 500m
    memory: 2Gi

podAntiAffinityPreset: hard

metrics:
  enabled: true
```

### ÉTAPE 4️ - Installer Redis Cluster avec Helm
```
helm install redis-cluster bitnami/redis-cluster -n redis -f redis-values.yaml
```

### ÉTAPE 5️ - Vérifier que tous les pods sont en Running
```
kubectl get pods -n redis -w
```

### ÉTAPE 6️ - Vérifier les PersistentVolumeClaims (PVC)
```
kubectl get pvc -n redis
```

###  ÉTAPE 7️ - Récupérer le mot de passe Redis et vérifier le cluster
```
$RedisPassword = kubectl get secret redis-cluster -n redis -o jsonpath="{.data.redis-password}" | base64 -d
kubectl exec -it redis-cluster-master-0 -n redis -- redis-cli -a $RedisPassword cluster info
```

### ÉTAPE 8️ - Tester Redis  
```
kubectl exec -it redis-cluster-master-0 -n redis -- redis-cli -c -a $RedisPassword
```
Dans Redis CLI :
```
SET test "Hello from PowerShell"
GET test
```
Vérifications importantes
```
kubectl get svc -n redis
kubectl get pods -n redis -o wide
```

