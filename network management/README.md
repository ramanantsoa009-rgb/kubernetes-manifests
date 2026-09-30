# TP 4 : Gestion du réseau avec les Services

## Objectif

Donner un point d'accès stable à un groupe de Pods et répartir les requêtes entre eux.

L'adresse IP d'un Pod change à chaque recréation, et `port-forward` ne vise qu'un seul Pod. Un **Service** résout ces deux problèmes : il sélectionne des Pods par leurs labels et leur envoie le trafic.

## Contenu

| Fichier | Rôle |
|---|---|
| [pod-red.yaml](pod-red.yaml) | Pod `simple-webapp-color` rouge, label `app: web` |
| [pod-blue.yaml](pod-blue.yaml) | Pod `simple-webapp-color` bleu, label `app: web` |
| [service-nodeport-web.yaml](service-nodeport-web.yaml) | Service **NodePort** : accessible depuis l'extérieur du cluster |
| [service-clusterip-web.yaml](service-clusterip-web.yaml) | Service **ClusterIP** : accessible uniquement depuis le cluster, utilisé par l'Ingress du TP 5 |

Toutes les ressources sont dans le namespace `production` (TP 3).

Les ports du Service NodePort :

| Champ | Valeur | Rôle |
|---|---|---|
| `nodePort` | 30008 | Port ouvert sur le nœud (plage autorisée : 30000 à 32767) |
| `port` | 80 | Port du Service dans le cluster |
| `targetPort` | 8080 | Port du conteneur qui reçoit le trafic |

## Déployer

```bash
kubectl apply -f "network management/"
```

Donner un dossier à `-f` applique tous les fichiers YAML qu'il contient.

## Vérifier

```bash
kubectl get pods -n production --show-labels
kubectl get service -n production
kubectl get endpoints service-nodeport-web -n production
```

- Les deux Pods doivent porter `app=web`.
- Le Service NodePort affiche `80:30008/TCP`.
- Les `endpoints` doivent lister deux adresses IP. Une liste vide signifie que le `selector` ne correspond aux labels d'aucun Pod.

## Tester la répartition de charge

Avec OrbStack, les NodePort sont accessibles sur `localhost`. Le navigateur réutilise souvent la même connexion, donc le même Pod : le terminal montre mieux l'alternance.

```bash
for i in $(seq 1 10); do curl -s http://localhost:30008 | grep -o "Hello from [^!]*"; done
```

Les réponses alternent entre `red-pod-simple-webapp-color` et `blue-pod-simple-webapp-color`.

## Ce que j'ai retenu

- Un Service ne connaît pas ses Pods par leur nom mais par leurs labels.
- ClusterIP (le type par défaut) sert à la communication interne. NodePort ouvre un port sur chaque nœud, pratique pour tester mais peu adapté à la production.
- Dans le cluster, un Service est joignable par son nom DNS (`service-clusterip-web`), ce qui sera utilisé au TP 7.

## Nettoyer

```bash
kubectl delete -f "network management/"
```

À garder si tu enchaînes avec le TP 5, qui utilise le Service ClusterIP.
