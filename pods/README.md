# TP 1 : Premier Pod

## Objectif

Déployer un premier Pod et accéder à l'application qu'il fait tourner.

Un **Pod** est la plus petite unité dans Kubernetes : il fait tourner un ou plusieurs conteneurs qui partagent le même réseau.

## Contenu

[pod.yaml](pod.yaml) lance l'application `mmumshad/simple-webapp-color`. La variable d'environnement `APP_COLOR` définit la couleur de fond de la page, ici **rouge**.

## Déployer

```bash
kubectl apply -f pods/pod.yaml
```

- `apply` crée la ressource décrite dans le fichier, ou la met à jour si elle existe déjà.
- `-f` indique le chemin du fichier YAML.

## Vérifier

```bash
kubectl get pods -o wide
```

La colonne `STATUS` doit afficher `Running`. Si elle affiche `ContainerCreating`, l'image est en cours de téléchargement : patiente quelques secondes et relance la commande.

## Accéder à l'application

Un Pod n'est pas accessible depuis l'extérieur du cluster. On ouvre donc un tunnel entre la machine et le Pod :

```bash
kubectl port-forward pod-simple-webapp-color 8080:8080
```

`8080:8080` relie le port 8080 de la machine au port 8080 du conteneur. La commande reste active tant que le terminal est ouvert (`Ctrl + C` pour l'arrêter).

Ouvre http://localhost:8080 : la page s'affiche avec un fond rouge.

## Ce que j'ai retenu

- Un Pod seul n'est pas recréé s'il est supprimé ou s'il plante : c'est le rôle du Deployment (TP 2).
- `port-forward` sert au test et au débogage, pas à exposer une application durablement : c'est le rôle des Services (TP 4).

## Nettoyer

```bash
kubectl delete -f pods/pod.yaml
```
