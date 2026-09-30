# TP 8 : WordPress avec Helm

## Objectif

Déployer le même WordPress qu'au [TP 7](../wp%20with%20mysql/), mais avec **Helm** au lieu d'écrire chaque manifest à la main.

Helm est un gestionnaire de paquets pour Kubernetes. Un **chart** regroupe tous les manifests d'une application (Deployments, Services, volumes, Secrets). On ne modifie que les valeurs dont on a besoin, dans un fichier `values.yaml`.

## Contenu

[values.yaml](values.yaml) surcharge les valeurs par défaut du chart `bitnami/wordpress`, qui installe WordPress et une base MariaDB.

| Clé | Valeur | Rôle |
|-----|--------|------|
| `wordpressUsername` | `admin` | Identifiant du compte administrateur WordPress |
| `wordpressPassword` | `password` | Mot de passe du compte administrateur |
| `wordpressHost` | `localhost:30080` | Adresse utilisée par WordPress pour générer ses URLs |
| `mariadb.auth.rootPassword` | `rootpassword` | Mot de passe root de MariaDB (vide par défaut dans le chart) |
| `mariadb.auth.password` | `""` | Mot de passe de l'utilisateur applicatif (vide : généré par le chart) |
| `service.type` | `NodePort` | Expose WordPress hors du cluster (ClusterIP par défaut) |
| `service.nodePorts.http` | `30080` | Port fixe sur le nœud pour le HTTP |
| `service.nodePorts.https` | `""` | Port HTTPS attribué automatiquement |
| `livenessProbe.enabled` | `false` | Probe désactivée |
| `readinessProbe.enabled` | `false` | Probe désactivée |
| `startupProbe.enabled` | `false` | Probe désactivée |

### Pourquoi désactiver les probes

L'installation initiale de WordPress prend du temps et les probes du chart interrogent le Pod avec le mauvais scheme (HTTPS). Le Pod est alors redémarré en boucle avant la fin de l'installation. On les désactive pour ce TP uniquement.

## Déployer

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

helm install wordpress bitnami/wordpress -f helm/values.yaml
```

La release est installée dans le namespace `default`.

## Vérifier

```bash
helm list
kubectl get pods,svc,pvc
```

Les Pods WordPress et MariaDB doivent être en `Running`. Ouvre http://localhost:30080 et connecte-toi sur http://localhost:30080/wp-admin avec `admin` / `password`.

Pour appliquer une modification du `values.yaml` :

```bash
helm upgrade wordpress bitnami/wordpress -f helm/values.yaml
```

## Comparaison avec le TP 7

| | TP 7 (manifests) | TP 8 (Helm) |
|---|---|---|
| Fichiers écrits | 4 manifests | 1 fichier de valeurs |
| Stockage | Pas de volume pour la base | PVC créés par le chart |
| Mots de passe | En clair dans les Deployments | Stockés dans des Secrets générés par le chart |
| Mise à jour | `kubectl apply` fichier par fichier | `helm upgrade`, avec retour arrière possible (`helm rollback`) |

## Ce que j'ai retenu

- Helm évite de réécrire des manifests déjà maintenus par d'autres, mais il faut lire la documentation du chart pour savoir quelles valeurs existent (`helm show values bitnami/wordpress`).
- Les valeurs par défaut d'un chart ne sont pas toujours adaptées à un cluster local : ici, le type de Service et les probes.

## Nettoyer

```bash
helm uninstall wordpress
kubectl get pvc
```

`helm uninstall` ne supprime pas le PVC de MariaDB. Le retirer à la main si besoin avec `kubectl delete pvc <nom>`.

## Attention

Les mots de passe sont en clair dans ce fichier : c'est acceptable pour un TP local, pas en production. Pour aller plus loin, utiliser un Secret Kubernetes et les options `existingSecret` du chart.
