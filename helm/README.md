# WordPress avec Helm

Fichier `values.yaml` pour surcharger les valeurs par défaut du chart WordPress de Bitnami, dans le cadre du TP.

## Installation

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

helm install wordpress bitnami/wordpress -f values.yaml
```

Le site est ensuite accessible sur http://localhost:30080.

## Contenu du values.yaml

| Clé | Valeur | Rôle |
|-----|--------|------|
| `wordpressUsername` | `admin` | Identifiant du compte administrateur WordPress |
| `wordpressPassword` | `password` | Mot de passe du compte administrateur |
| `wordpressHost` | `localhost:30080` | Adresse utilisée par WordPress pour générer ses URLs |
| `mariadb.auth.rootPassword` | `rootpassword` | Mot de passe root de MariaDB (vide par défaut dans le chart) |
| `mariadb.auth.password` | `""` | Mot de passe de l'utilisateur applicatif (vide : généré par le chart) |
| `service.type` | `NodePort` | Expose WordPress hors du cluster (ClusterIP par défaut) |
| `service.nodePorts.http` | `30080` | Port fixe sur le noeud pour le HTTP |
| `service.nodePorts.https` | `""` | Port HTTPS attribué automatiquement |
| `livenessProbe.enabled` | `false` | Probe désactivée |
| `readinessProbe.enabled` | `false` | Probe désactivée |
| `startupProbe.enabled` | `false` | Probe désactivée |

## Pourquoi désactiver les probes

L'installation initiale de WordPress prend du temps et les probes du chart interrogent le pod avec le mauvais scheme (HTTPS). Le pod est alors redémarré en boucle avant la fin de l'installation. On les désactive pour ce TP uniquement.

## Commandes utiles

```bash
# Vérifier l'état des pods et du service
kubectl get pods,svc

# Appliquer une modification du values.yaml
helm upgrade wordpress bitnami/wordpress -f values.yaml

# Tout supprimer (le PVC de MariaDB reste, à supprimer à la main si besoin)
helm uninstall wordpress
kubectl get pvc
```

## Attention

Les mots de passe sont en clair dans ce fichier : c'est acceptable pour un TP local, pas en production. Pour aller plus loin, utiliser un Secret Kubernetes et les options `existingSecret` du chart.
