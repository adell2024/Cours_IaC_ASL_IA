# 🧪 Exercice 11 – Application PHP connectée à MariaDB (Secrets existants)
🎯 Objectif

Déployer une application PHP qui :

communique avec MariaDB via un Service Kubernetes

utilise un Secret déjà existant pour le mot de passe

respecte les bonnes pratiques de sécurité (aucun secret en clair)

📌 Pré-requis

Les ressources suivantes existent déjà :

Deployment mariadb

Service mariadb-service

Secret mariadb-pass (clé password)

Database mabase

👉 On ne recrée rien ici (sauf le serveur web)

🅰️ Étape 1 – ConfigMap + Deployment + Service  : en un seul fichier. séparés par "----"

Le fichier php-app.yaml:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: php-code
data:
  index.php: |
    <?php
    $host = 'mariadb-service';
    $db   = 'mabase';
    $user = 'root';
    $pass = getenv('DB_PASSWORD'); // 🔐 Mot de passe depuis le Secret

    try {
        $dsn = "mysql:host=$host;dbname=$db;charset=utf8mb4";
        $pdo = new PDO($dsn, $user, $pass);

        echo "<body style='font-family:sans-serif; text-align:center; padding-top:50px; background-color:#f0fff4;'>";
        echo "<h1 style='color:#2f855a;'>✅ Connexion Réussie !</h1>";
        echo "<p>PHP communique correctement avec MariaDB.</p>";
        echo "<p><b>Service DB :</b> $host</p>";
        echo "<p><b>IP Pod PHP :</b> " . $_SERVER['SERVER_ADDR'] . "</p>";
        echo "</body>";
    } catch (PDOException $e) {
        echo "<body style='font-family:sans-serif; text-align:center; padding-top:50px; background-color:#fff5f5;'>";
        echo "<h1 style='color:#c53030;'>❌ Erreur de Connexion</h1>";
        echo "<p>" . $e->getMessage() . "</p>";
        echo "</body>";
    }
    ?>

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: php-web
  template:
    metadata:
      labels:
        app: php-web
    spec:
      containers:
      - name: php
        image: php:8.0-apache
        command: ["sh", "-c", "docker-php-ext-install pdo pdo_mysql && apache2-foreground"]
        ports:
        - containerPort: 80
        env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mariadb-pass
              key: password
        volumeMounts:
        - name: code-volume
          mountPath: /var/www/html/index.php
          subPath: index.php
      volumes:
      - name: code-volume
        configMap:
          name: php-code

---
apiVersion: v1
kind: Service
metadata:
  name: php-service
spec:
  type: ClusterIP
  selector:
    app: php-web
  ports:
    - port: 80
      targetPort: 80
```

### 🅱️ Étape 2 : Déploiement

kubectl apply -f php-app.yaml

### 🅴 Étape 3 – Test

kubectl run curlpod --rm -it \
  --image=curlimages/curl \
  --restart=Never -- \
  curl http://php-service

