# Kubernetes : TP et manifests d'apprentissage

Dépôt de travail personnel, constitué pendant ma formation Kubernetes. Chaque dossier correspond à un TP : j'y écris les manifests à la main, je les déploie sur un cluster local et je documente ce que j'ai compris.

Le but n'est pas de livrer une application prête pour la production. Il s'agit d'apprendre les briques de base de Kubernetes, une par une, en les manipulant réellement.

## Objectifs

- Comprendre les objets fondamentaux de Kubernetes et leur rôle : Pod, Deployment, Namespace, Service, Ingress, PersistentVolume.
- Savoir écrire un manifest YAML sans générateur, le déployer et vérifier son état avec `kubectl`.
- Exposer une application à l'intérieur et à l'extérieur du cluster.
- Rendre des données persistantes au-delà de la vie d'un Pod.
- Déployer une application multi-conteneurs (WordPress et sa base de données), d'abord à la main, puis avec Helm.

## Technologies utilisées

| Outil | Usage dans ce dépôt |
|---|---|
| Kubernetes | Orchestration des conteneurs |
| [OrbStack](https://orbstack.dev/) | Cluster Kubernetes local sur macOS (un seul nœud) |
| kubectl | Déploiement et inspection des ressources |
| Helm | Installation de WordPress à partir du chart Bitnami |
| Traefik | Ingress controller pour le routage HTTP par nom d'hôte |
| Images Docker | `nginx`, `mysql`, `wordpress`, `mmumshad/simple-webapp-color` |
| Git / GitHub | Suivi de la progression, un commit par étape |

## Parcours des TP

Les TP se suivent dans cet ordre : chacun s'appuie sur les précédents.

| # | Dossier | Notions abordées |
|---|---|---|
| 1 | [pods/](pods/) | Premier Pod, variables d'environnement, `port-forward` |
| 2 | [first deployment/](first%20deployment/) | Deployment, replicas, recréation automatique des Pods |
| 3 | [namespaces/](namespaces/) | Isolation logique des ressources |
| 4 | [network management/](network%20management/) | Labels, selectors, Services NodePort et ClusterIP, répartition de charge |
| 5 | [ingress controllers/](ingress%20controllers/) | Ingress, routage par nom d'hôte avec Traefik |
| 6 | [pv and pvc/](pv%20and%20pvc/) | Volumes `hostPath`, PersistentVolume, PersistentVolumeClaim |
| 7 | [wp with mysql/](wp%20with%20mysql/) | Application à deux tiers, communication entre Services par nom DNS |
| 8 | [helm/](helm/) | Déploiement du même WordPress avec un chart Helm et un `values.yaml` |

Chaque dossier contient son propre README avec les commandes, les vérifications et le nettoyage.

## Prérequis

- OrbStack installé, avec Kubernetes activé
- `kubectl` (fourni par OrbStack)
- `helm` pour le TP 8

```bash
orb start k8s
kubectl get nodes    # le nœud doit être en statut Ready
```

Toutes les commandes des README se lancent depuis la racine du dépôt. Plusieurs dossiers contiennent des espaces : mets le chemin entre guillemets, par exemple `kubectl apply -f "network management/"`.

## Limites assumées

Ce dépôt sert à apprendre, certains choix sont donc volontairement simples :

- Les mots de passe sont écrits en clair dans les manifests. Ce sont des valeurs de démonstration, utilisées uniquement sur un cluster local.
- Les volumes utilisent `hostPath`, qui ne fonctionne que sur un cluster à un seul nœud.
- Les images ne sont pas figées à une version précise (`mysql`, `wordpress`, `nginx:latest`).

## Prochaines étapes

- Gestion des Secrets et des ConfigMaps, pour sortir les mots de passe des manifests
- Probes de santé (liveness, readiness)
- Limites de ressources (CPU, mémoire)

## Source

Les exercices suivent la formation Kubernetes d'[Eazytraining](https://eazytraining.fr/). Les manifests et les explications de ce dépôt sont les miens.

## Commandes de base

| Action | Commande |
|---|---|
| Démarrer le cluster | `orb start k8s` |
| Créer ou mettre à jour une ressource | `kubectl apply -f <fichier ou dossier>` |
| Lister des ressources | `kubectl get pods,svc,deploy -n <namespace>` |
| Voir le détail et les événements | `kubectl describe <type> <nom> -n <namespace>` |
| Lire les logs d'un Pod | `kubectl logs <pod> -n <namespace>` |
| Supprimer | `kubectl delete -f <fichier ou dossier>` |
| Arrêter le cluster | `orb stop k8s` |
