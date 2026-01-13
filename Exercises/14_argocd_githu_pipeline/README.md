Exercice 14 – Déployer une application Flask + Postgres avec Argo CD (GitOps)
# 🎯 Objectifs pédagogiques

Construire et publier une image Docker

Déployer une application via Argo CD

Comprendre le rôle de chaque fichier Kubernetes

Mettre en œuvre une démarche GitOps

Observer la synchronisation automatique Argo CD → Cluster



🔹 PARTIE A – Construire et publier l’image Docker

Remplace 'ton-pseudo' par ton vrai nom d'utilisateur Docker Hub

docker build -t ton-pseudo/flask-app:v1 .
### 1. Connexion à ton compte (entre ton login et ton mot de passe/token)
docker login

### 2. On donne un nouveau nom à l'image locale pour Docker Hub
docker tag flask-app:v1 ton-pseudo/flask-app:v1

### 3. On envoie l'image sur le Cloud
docker push ton-pseudo/flask-app:v1


### 🧩 B – Installation d'ArgoCD sur le cluster

Créer le namespace:

kubectl create namespace argocd

Installer ArgoCD (version stable):

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

Attendre que tous les pods soient prêts:

kubectl wait --for=condition=Ready pods --all -n argocd --timeout=300s

Vérifier que tous les pods sont en Running:
kubectl get pods -n argocd

Devrait montrer tous les pods en Running/Ready

Appliquer ce script: 
```bash
# 1. Mise à jour de l'application-controller (StatefulSet)
kubectl patch statefulset argocd-application-controller -n argocd --type strategic -p '
spec:
  template:
    spec:
      dnsPolicy: "None"
      dnsConfig:
        nameservers:
          - 10.96.0.10
          - 8.8.8.8
          - 8.8.4.4
        searches:
          - argocd.svc.cluster.local
          - svc.cluster.local
          - cluster.local
        options:
          - name: ndots
            value: "2"
'

# 2. Mise à jour du repo-server (Deployment)
kubectl patch deployment argocd-repo-server -n argocd --type strategic -p '
spec:
  template:
    spec:
      dnsPolicy: "None"
      dnsConfig:
        nameservers:
          - 10.96.0.10
          - 8.8.8.8
          - 8.8.4.4
        searches:
          - argocd.svc.cluster.local
          - svc.cluster.local
          - cluster.local
        options:
          - name: ndots
            value: "2"
'

# 3. Redémarrage des composants pour appliquer les changements
kubectl rollout restart statefulset argocd-application-controller -n argocd
kubectl rollout restart deployment argocd-repo-server -n argocd

# 4. Vérification du statut du déploiement
kubectl rollout status statefulset argocd-application-controller -n argocd
kubectl rollout status deployment argocd-repo-server -n argocd

[!IMPORTANT] Vérifiez que l'IP 10.96.0.10 correspond bien à l'IP du service kube-dns ou coredns dans votre cluster:
(kubectl get svc -n kube-system).

```

### Récupération du mot de passe admin de ArgoCD:

kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d

### Vérifier que le pod argocd est capable de résoudre le domain github.com

par exemple, le mien ou le votre (rempalcer argocd argocd-repo-server-6d8f9bc87b-rwt4c) :
kubectl exec -n argocd argocd-repo-server-6d8f9bc87b-rwt4c --   git ls-remote https://github.com/adell2024/mon-flask-app.git

Defaulted container "argocd-repo-server" out of: argocd-repo-server, copyutil (init)

bade6e6aa901f04120ab28376b75bcbb7a5e0caa        HEAD

bade6e6aa901f04120ab28376b75bcbb7a5e0caa        refs/heads/main


###🔹 PARTIE C – Déploiement GitOps avec Argo CD
🧩 C1 – Manifeste Argo CD
📄 argo-app-definition.yaml
