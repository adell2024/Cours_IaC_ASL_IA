# 🧪 Exercice 01 — Déployer une application de test sur Kubernetes

## 🎯 Objectif
Déployer une application simple (`nginx`) sur un cluster Kubernetes afin de vérifier
le bon fonctionnement du cluster, du réseau et des services.

---

## 📌 Contexte
Le cluster Kubernetes est opérationnel :
- Tous les nœuds sont en état `Ready`
- Le réseau (Flannel) est correctement déployé
- L’accès au cluster via `kubectl` est fonctionnel

Vous allez déployer votre **première application** sur le cluster.

---

## 🧩 Étapes

### Étape 1 — Créer un déploiement NGINX
Créer un déploiement `nginx` avec **3 réplicas** :

kubectl create deployment nginx --image=nginx --replicas=3

### Étape 2 — Vérifier les pods

Vérifier que les pods sont bien créés et en cours d’exécution :
kubectl get pods
### Étape 3 — Exposer l’application

Exposer le déploiement via un Service de type NodePort :
kubectl expose deployment nginx --port=80 --type=NodePort

### Étape 4 — Vérifier le service

Afficher les informations du service :
kubectl get svc nginx

### Étape 5 — Tester l’accès à l’application

Depuis votre navigateur ou votre terminal, accéder à l’application :
 curl http://10.0.0.11:31078
 
✔️ La page d’accueil NGINX doit s’afficher.

✅ Résultat attendu

Le déploiement nginx est présent

3 pods sont en état Running

Le service nginx est accessible depuis l’extérieur du cluster

🧠 Questions (optionnel)

Sur quels nœuds les pods nginx sont-ils déployés ?

Que se passe-t-il si vous supprimez un pod nginx ?

Quelle est la différence entre un Service ClusterIP et NodePort ?
