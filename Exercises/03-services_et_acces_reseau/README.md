# 🧪 Exercice 04 — Services Kubernetes et accès réseau

## 🎯 Objectif
Comprendre comment Kubernetes expose une application à l’intérieur et à l’extérieur du cluster
en utilisant différents types de Services (`ClusterIP`, `NodePort`).

---

## 📌 Contexte
Vous avez déjà un déploiement `nginx` actif avec plusieurs pods.  
L’objectif est maintenant de **rendre l’application accessible depuis le cluster et depuis l’extérieur**, et de comprendre le rôle des différents types de Service.

---

## 🧩 Étapes

### Étape 1 — Créer un Service ClusterIP (interne)

kubectl expose deployment nginx --port=80 --type=ClusterIP  // peut-être il faut commencer par faire: kubectl delete svc nginx

Vérifiez :

kubectl get svc nginx

CLUSTER-IP est visible

Pas d’EXTERNAL-IP (service interne uniquement)

### Étape 2 — Tester l’accès interne depuis un pod

kubectl run curlpod --image=radial/busyboxplus:curl -i --tty



