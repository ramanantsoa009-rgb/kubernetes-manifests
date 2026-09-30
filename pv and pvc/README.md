# TP 6 : Stockage persistant (volumes, PV et PVC)

## Objectif

Conserver les données d'une base MySQL quand son Pod est supprimé ou recréé.

Par défaut, tout ce qu'un conteneur écrit disparaît avec lui. Ce TP compare deux façons de garder les données :

1. Monter directement un dossier du nœud dans le Pod (`hostPath`).
2. Passer par un **PersistentVolume** (PV) et un **PersistentVolumeClaim** (PVC), qui séparent le stockage de l'application.

## Contenu

| Fichier | Rôle |
|---|---|
| [pod-mysql-volune.yaml](pod-mysql-volune.yaml) | Pod MySQL `mysql-volume`, données montées directement depuis `/data-volume` sur le nœud |
| [pv.yaml](pv.yaml) | PersistentVolume `mysql-pv` : 1 Gi, stocké dans `/data-pv` sur le nœud |
| [pvc.yaml](pvc.yaml) | PersistentVolumeClaim `mysql-pvc` : demande 100 Mi de stockage |
| [pod-mysql-pv.yaml](pod-mysql-pv.yaml) | Pod MySQL `mysql-pv`, qui utilise le PVC |

Dans les deux Pods, les données de MySQL (`/var/lib/mysql`) sont montées sur le volume.

## Approche 1 : volume hostPath

```bash
kubectl apply -f "pv and pvc/pod-mysql-volune.yaml"
kubectl get pod mysql-volume -n production
```

Simple, mais le Pod dépend d'un chemin précis sur un nœud précis.

## Approche 2 : PV et PVC

L'administrateur du cluster fournit le stockage (PV). L'application demande seulement « 100 Mi en lecture-écriture » (PVC), sans savoir où il se trouve.

```bash
kubectl apply -f "pv and pvc/pv.yaml"
kubectl apply -f "pv and pvc/pvc.yaml"
kubectl get pv,pvc -n production
```

Le PVC doit passer en statut `Bound`, relié à `mysql-pv`. On peut ensuite lancer le Pod :

```bash
kubectl apply -f "pv and pvc/pod-mysql-pv.yaml"
kubectl get pod mysql-pv -n production
```

## Tester la persistance

Créer une table, supprimer le Pod, le recréer, puis vérifier que la table est toujours là :

```bash
kubectl exec -it mysql-pv -n production -- mysql -u user -ppassword my-db \
  -e "CREATE TABLE test (id INT); SHOW TABLES;"

kubectl delete pod mysql-pv -n production
kubectl apply -f "pv and pvc/pod-mysql-pv.yaml"

kubectl exec -it mysql-pv -n production -- mysql -u user -ppassword my-db -e "SHOW TABLES;"
```

Attends que le Pod soit en `Running` avant la dernière commande.

## Ce que j'ai retenu

- `storageClassName: ""` dans le PVC est indispensable ici. Sans lui, le cluster utilise sa StorageClass par défaut et crée un nouveau volume au lieu de se lier au PV créé à la main.
- Un PVC se lie à un PV dont la capacité est au moins égale à la demande (1 Gi pour 100 Mi demandés) et dont le mode d'accès correspond.
- Un PV n'appartient à aucun namespace, contrairement au PVC. Le champ `namespace` de [pv.yaml](pv.yaml) est ignoré.
- `hostPath` ne convient qu'à un cluster à un seul nœud, comme celui d'OrbStack.

## Nettoyer

```bash
kubectl delete -f "pv and pvc/"
```

Supprimer le PV ne vide pas le dossier `/data-pv` sur le nœud.
