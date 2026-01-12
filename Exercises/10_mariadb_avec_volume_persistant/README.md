# 🧪 Exercice 04 — Déployer MariaDB avec stockage persistant
🎯 Objectifs pédagogiques

À la fin de cet exercice, vous devriez être capable de :

Déployer une base de données MariaDB dans Kubernetes

Utiliser un Secret pour gérer un mot de passe sensible

Mettre en place un stockage persistant via PV / PVC

Comprendre le rôle de nodeSelector avec un hostPath

Exposer la base via un Service interne (ClusterIP)

🧩 Étape A — Création du Secret

La base MariaDB nécessite un mot de passe root.

Pour des raisons de sécurité, celui-ci est stocké dans un Secret Kubernetes.

kubectl create secret generic mariadb-pass \
  --from-literal=password=supersecret
  
Vérification :

kubectl get secrets

kubectl describe secret mariadb-pass

📌 Pourquoi un Secret ?

Évite d’écrire des mots de passe en clair dans les fichiers YAML

Permet une gestion séparée des données sensibles

🧩 Étape B — Création du PersistentVolume (PV)

Fichier mariadb-pv.yaml

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mariadb-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data-mariadb"
```
Application :

kubectl apply -f mariadb-pv.yaml

kubectl get pv

📌 Point important

hostPath crée le stockage localement sur le nœud

Ce type de volume est pédagogique, pas recommandé en production

🧩 Étape C — Création du PersistentVolumeClaim (PVC)

Fichier mariadb-pvc.yaml :

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mariadb-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```
Application :

kubectl apply -f mariadb-pvc.yaml

kubectl get pvc

📌 Le PVC permet au pod de demander dynamiquement du stockage sans connaître le PV exact.

🧩 Étape D — Déploiement de MariaDB

Fichier mariadb-deploy.yaml :

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mariadb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mariadb
  template:
    metadata:
      labels:
        app: mariadb
    spec:
      nodeSelector:
        kubernetes.io/hostname: k8s-worker1
      containers:
      - name: mariadb
        image: mariadb:10.6
        env:
        - name: MARIADB_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mariadb-pass
              key: password
        - name: MARIADB_DATABASE
          value: mabase
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: storage
          mountPath: /var/lib/mysql
      volumes:
      - name: storage
        persistentVolumeClaim:
          claimName: mariadb-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: mariadb-service
spec:
  selector:
    app: mariadb
  ports:
    - port: 3306
      targetPort: 3306
```
Application :

kubectl apply -f mariadb-deploy.yaml

🔍 Vérifications

kubectl get pods -o wide

kubectl get svc

kubectl describe pod mariadb

Vérifier que :

Le pod est bien lancé sur k8s-worker1

Le PVC est bien monté

Le Service est en ClusterIP

🧪 Test de connexion à la base

Créer un pod client temporaire :

kubectl get secret mariadb-pass \
  -o jsonpath="{.data.password}" | base64 -d

kubectl run mariadb-client --rm -it \
  --image=mariadb:10.6 \
  --env="MYSQL_PWD=supersecret" \
  --restart=Never -- \
  mariadb -h mariadb-service -u root

📌 MariaDB utilise automatiquement MYSQL_PWD si elle existe.

🧠 Point pédagogique important (à mettre en évidence dans le cours)
🔹 Le Secret est scopé au pod

Un Secret :

❌ n’est pas global au cluster

❌ n’est pas partagé automatiquement

Il doit être :

monté

ou injecté
explicitement dans chaque pod

👉 Sécurité par défaut de Kubernetes

🧪 Vérification côté serveur (bonus)

kubectl exec -it deploy/mariadb -- env | grep MARIADB

Résultat attendu :

MARIADB_ROOT_PASSWORD=********

MARIADB_DATABASE=mabase

deploy/mariadb est un Deployment

kubectl exec attend un Pod

kubectl est censé :

résoudre le Deployment

trouver un Pod

s’y connecter

🔹 Toujours exécuter kubectl exec sur un Pod, pas un Deployment
1️⃣ Récupérer le nom exact du pod

kubectl get pods -l app=mariadb

Exemple de sortie :

mariadb-7c6c9b8d7f-abcde

2️⃣ Exécuter la commande correctement

kubectl exec -it mariadb-7c6c9b8d7f-abcde -- env | grep MARIADB

📌 À noter (important pour le cours)

kubectl exec deploy/xxx peut fonctionner sur certaines versions,
mais ce n’est pas fiable et peut provoquer des erreurs internes.

👉 Bonne règle pédagogique :

get / describe → Deployment, Service

logs / exec → Pod uniquement

🧠 À mettre sur GitHub (texte prêt)

Lors de l’utilisation de kubectl exec, la commande doit cibler un Pod et non un Deployment.
Dans certaines versions de kubectl, l’exécution directe sur un Deployment peut provoquer un panic interne du client (nil pointer dereference).
Il est donc recommandé de récupérer explicitement le nom du pod avant d’utiliser kubectl exec.
