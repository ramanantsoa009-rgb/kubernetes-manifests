# TP 2 : Premier Deployment

## Objectif

Passer d'un Pod isolé à un **Deployment**, qui maintient en permanence un nombre donné de copies (replicas) d'une application.

## Contenu

[pod-nginx.yaml](pod-nginx.yaml) décrit un Deployment `deployment-nginx` :

| Champ | Valeur | Rôle |
|---|---|---|
| `replicas` | 2 | Nombre de Pods que Kubernetes maintient en vie |
| `selector.matchLabels` | `app: nginx` | Pods gérés par ce Deployment |
| `template` | image `nginx:latest`, port 80 | Modèle utilisé pour créer chaque Pod |

Le label du `template` doit correspondre au `selector`, sinon Kubernetes refuse le Deployment.

## Déployer

```bash
kubectl apply -f "first deployment/pod-nginx.yaml"
```

## Vérifier

```bash
kubectl get deployments
kubectl get pods -l app=nginx
```

`-l app=nginx` filtre les Pods qui portent ce label. Deux Pods doivent être en `Running`.

## Tester la recréation automatique

Supprime un des Pods (remplace `<nom-du-pod>` par un nom affiché par la commande précédente) :

```bash
kubectl delete pod <nom-du-pod>
kubectl get pods -l app=nginx
```

Un nouveau Pod apparaît aussitôt pour revenir à 2 replicas.

## Accéder à nginx

```bash
kubectl port-forward deployment/deployment-nginx 8081:80
```

Ouvre http://localhost:8081 : la page « Welcome to nginx! » s'affiche. Le port 8081 évite le conflit avec le TP 1.

## Ce que j'ai retenu

- On ne crée presque jamais de Pod à la main : on décrit un Deployment et Kubernetes gère les Pods.
- Le lien entre un Deployment et ses Pods passe uniquement par les labels.

## Nettoyer

```bash
kubectl delete -f "first deployment/pod-nginx.yaml"
```
