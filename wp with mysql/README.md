# TP 7 : WordPress et MySQL

## Objectif

Déployer une application à deux tiers : un site WordPress qui s'appuie sur une base MySQL, chacune dans son propre Deployment.

Ce TP réunit les notions précédentes : Deployments, Services, namespace et volumes.

## Architecture

```
navigateur --> localhost:30088 --> wordpress-service (NodePort)
                                        |
                                   Pod WordPress
                                        |
                          mysql-service:3306 (ClusterIP)
                                        |
                                    Pod MySQL
```

## Contenu

| Fichier | Rôle |
|---|---|
| [mysql-deployment.yaml](mysql-deployment.yaml) | MySQL, avec la base `my-db` et l'utilisateur `user` |
| [mysql-service.yaml](mysql-service.yaml) | Service ClusterIP : la base n'est joignable que depuis le cluster |
| [wordpress-deployment.yml](wordpress-deployment.yml) | WordPress, configuré pour se connecter à `mysql-service` |
| [wordpress-service.yml](wordpress-service.yml) | Service NodePort sur le port 30088 |

WordPress trouve la base grâce à `WORDPRESS_DB_HOST: mysql-service` : dans le cluster, le nom d'un Service sert de nom DNS.

## Déployer

Le namespace `production` doit exister (TP 3).

```bash
kubectl apply -f "wp with mysql/"
```

## Vérifier

```bash
kubectl get deploy,pods,svc -n production
kubectl logs deploy/wordpress-deployment -n production
```

Les deux Pods doivent être en `Running`. Au premier démarrage, MySQL met quelques dizaines de secondes à s'initialiser : WordPress peut afficher une erreur de connexion à la base pendant ce temps.

Ouvre http://localhost:30088 : l'assistant d'installation de WordPress s'affiche.

## Ce que j'ai retenu

- La base reste en ClusterIP : seul WordPress a besoin d'y accéder.
- Les identifiants de la base sont répétés dans deux fichiers. Une modification oubliée d'un côté casse la connexion : c'est un des problèmes que les Secrets résolvent.
- MySQL n'a pas de volume dans ce TP : ses données sont perdues si son Pod est recréé. Le TP 6 montre comment corriger ça.

## Nettoyer

```bash
kubectl delete -f "wp with mysql/"
```
