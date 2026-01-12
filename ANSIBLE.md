Déploiement Kubernetes avec Ansible

Ce guide présente l’ordre d’exécution des scripts et des rôles Ansible pour déployer un cluster Kubernetes sur vos VMs créées à partir du template Proxmox.

📋 Prérequis

Les VMs doivent être créées et accessibles via SSH.

Votre clé SSH publique doit être ajoutée au template ou aux VMs.

Le tunnel réseau peut être créé si les VMs sont sur un réseau non directement accessible.

🔌 Tester la connectivité aux VMs

Avant de lancer vos playbooks, assurez-vous que les VMs sont joignables :

Créer un tunnel SSH vers le réseau des VMs (si nécessaire) :
sshuttle -r root@172.16.200.XX 10.0.0.0/24

Si toutes les VMs répondent avec pong, vous pouvez procéder au déploiement.

🛠️ Ordre d’exécution des playbooks

Les playbooks sont exécutés dans l’ordre suivant :
| Ordre | Playbook               | Description                                                                                         |
| ----- | ---------------------- | --------------------------------------------------------------------------------------------------- |
| 1     | `01-prepare-nodes.yml` | Prépare les VMs : mise à jour système, installation des packages de base et configuration initiale. |
| 2     | `02-install-k8s.yml`   | Installe Kubernetes (kubeadm, kubelet, kubectl) sur toutes les VMs.                                 |
| 3     | `03-init-master.yml`   | Initialise le nœud maître Kubernetes et configure le réseau du cluster.                             |
| 4     | `04-join-workers.yml`  | Ajoute les nœuds workers au cluster maître.                                                         |

✅ Recommandations

Testez toujours la connectivité avant d’exécuter les playbooks.

Exécutez les playbooks dans l’ordre indiqué.

Vérifiez les journaux Ansible pour détecter toute erreur avant de passer au playbook suivant.

Les tâches critiques sont idempotentes : vous pouvez relancer un playbook sans risque de casser la configuration existante.

🚀 ÉTAPE 1 : PRÉPARATION DES NODES
ansible-playbook -i inventory/hosts.yml playbooks/01-prepare-nodes.yml
```

**Lancez cette commande et observez l'exécution.**

**Ce qui va se passer :**
- Installation des paquets système
- Désactivation du swap
- Configuration des modules kernel
- Configuration des paramètres réseau

**Durée estimée : 2-3 minutes**

---

## 📋 **Pendant que ça tourne, voici ce que vous pouvez observer :**

Vous allez voir des lignes comme :
```
TASK [Mettre à jour le cache APT] **********************
ok: [k8s-master]
ok: [k8s-worker1]
ok: [k8s-worker2]
ok: [k8s-worker3]

TASK [Désactiver le swap immédiatement] ****************
changed: [k8s-master]
changed: [k8s-worker1]
...
```

---

## ✅ **À la fin, vous devriez voir :**
```
PLAY RECAP *********************************************
k8s-master     : ok=XX   changed=XX   unreachable=0    failed=0
k8s-worker1    : ok=XX   changed=XX   unreachable=0    failed=0
k8s-worker2    : ok=XX   changed=XX   unreachable=0    failed=0
k8s-worker3    : ok=XX   changed=XX   unreachable=0    failed=0
Si vous voyez failed=0 partout, c'est parfait ! ✅
