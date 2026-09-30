# TP 3 : Namespaces

## Objectif

Ranger les ressources dans un espace dédié au lieu du namespace `default`.

Un **namespace** isole logiquement des ressources à l'intérieur d'un même cluster. Deux ressources peuvent porter le même nom si elles sont dans des namespaces différents.

## Contenu

[namespace.yaml](namespace.yaml) crée le namespace `production`. Il est utilisé par tous les TP suivants.

## Déployer

```bash
kubectl apply -f namespaces/namespace.yaml
```

## Vérifier

```bash
kubectl get namespaces
kubectl get pods -n production
```

`production` doit apparaître avec le statut `Active`. La liste des Pods est vide pour l'instant (`No resources found`).

## Ce que j'ai retenu

- Sans `-n <namespace>`, `kubectl` travaille dans `default`. Oublier `-n` est la cause la plus fréquente du message « ressource introuvable ».
- Le namespace doit exister avant les ressources qui y font référence.
- Supprimer un namespace supprime tout ce qu'il contient.

## Nettoyer

À faire en dernier, une fois les TP suivants terminés :

```bash
kubectl delete -f namespaces/namespace.yaml
```
