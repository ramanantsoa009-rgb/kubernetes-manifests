# TP 5 : Ingress avec Traefik

## Objectif

Exposer une application par un nom d'hôte, sur le port HTTP standard, au lieu d'un NodePort.

Un **Ingress** décrit des règles de routage HTTP : « les requêtes pour tel nom d'hôte et tel chemin vont vers tel Service ». Ces règles ne font rien seules : il faut un **Ingress controller** dans le cluster pour les appliquer. Ici, c'est Traefik.

## Contenu

[ingresscontrollers.yaml](ingresscontrollers.yaml) crée l'Ingress `ingresscontroller-web` dans le namespace `production` :

| Règle | Valeur |
|---|---|
| Nom d'hôte | `webapp.example.com` |
| Chemin | `/` (type `Prefix` : toutes les URL) |
| Destination | Service `service-clusterip-web`, port 80 |

## Prérequis

- Traefik installé dans le cluster
- Les Pods rouge et bleu et le Service ClusterIP du [TP 4](../network%20management/) déployés

## Déployer

```bash
kubectl apply -f "ingress controllers/ingresscontrollers.yaml"
```

## Vérifier

```bash
kubectl get ingress -n production
kubectl describe ingress ingresscontroller-web -n production
```

La section `Rules` de `describe` doit montrer `webapp.example.com` relié à `service-clusterip-web:80`, avec les adresses IP des deux Pods.

## Tester

`webapp.example.com` n'existe pas dans le DNS. On passe donc le nom d'hôte à la main dans l'en-tête `Host` :

```bash
curl -H "Host: webapp.example.com" http://localhost
```

Pour tester dans le navigateur, ajouter la ligne `127.0.0.1 webapp.example.com` au fichier `/etc/hosts`, puis ouvrir http://webapp.example.com.

## Ce que j'ai retenu

- Le Service derrière un Ingress peut rester en ClusterIP : seul l'Ingress controller est exposé.
- Un seul point d'entrée peut servir plusieurs applications, distinguées par le nom d'hôte ou le chemin.

## Nettoyer

```bash
kubectl delete -f "ingress controllers/ingresscontrollers.yaml"
```
