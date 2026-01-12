# 🚀 Étape 1 : Préparation de l'Infrastructure (Template Proxmox)

Avant de déployer votre application avec Terraform ou Kubernetes, vous devez préparer l'image de base (**template**) sur votre nœud Proxmox.  

Nous utilisons un script d'automatisation qui configure une image **Ubuntu Noble 24.04 LTS** optimisée avec **cloud-init** et le **qemu-guest-agent**.

---

## 📋 Prérequis

- Un accès **SSH root** à votre nœud Proxmox.
- Votre **clé SSH publique** configurée sur le nœud pour une connexion sans mot de passe.
- Les **bridges réseaux vmbr0 et vmbr1** configurés sur Proxmox.

---

## 🛠️ Exécution du script de création

Le script `create_vm_template.sh` s'exécute depuis votre poste de travail local.  
Il va générer une clé SSH dédiée pour vos futures VMs, la transférer sur Proxmox, et piloter la création du template à distance.

1. Rendez le script exécutable :

```bash
chmod +x create_vm_template.sh

    Lancez la création du template : vous devez spécifier le type de stockage cible (local-lvm ou nfs) en argument.

# Exemple pour un stockage local-lvm
./create_vm_template.sh local-lvm

# Exemple pour un stockage NFS
./create_vm_template.sh nfs

🔍 Ce que fait le script

    Génération de clé : Crée une paire de clés SSH (id_rsa_proxmox_templates) sur votre machine pour sécuriser l'accès aux futures VMs.

    Provisioning Cloud-Init : Configure l'utilisateur par défaut (ubuntu), le mot de passe (azerty) et injecte votre clé publique.

    Optimisation : Installe automatiquement le qemu-guest-agent et effectue les mises à jour système (apt upgrade) au premier démarrage.

    Réseau : Prépare une configuration dual-stack (Management/Data) prête à être pilotée par Terraform.

    ⏱ Temps d'attente : Lors du premier déploiement d'une VM basée sur ce template, prévoyez 5 à 10 minutes pour que cloud-init termine les mises à jour et l'installation des paquets.

⚠️ Recommandations importantes

    Si vous disposez d’un partage NFS, il est recommandé de créer le template dessus (par exemple STORAGE='nfs') car :

        L’import du disque est plus rapide.

        Le clonage des VM depuis le template est également plus rapide.

    Le script ne convertit pas automatiquement la VM en template.
    Je préfère vérifier que la VM fonctionne correctement avant de la transformer manuellement en template via l’interface Proxmox ou la CLI :

qm template <VMID>

    Le template utilise DHCP par défaut sur les interfaces réseau.

    Les IP statiques doivent être définies par Terraform ou manuellement après le clonage.

    Le qemu-guest-agent sera automatiquement installé et activé.
