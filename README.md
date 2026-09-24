# Formation Kubernetes – Premiers pas

Ce guide montre comment lancer un cluster Kubernetes en local, y déployer des applications, les ranger dans un namespace et les exposer avec un Service.

## Organisation du dépôt

| Dossier | Contenu |
|---|---|
| [pods/](pods/) | Un Pod simple (application web à fond rouge) |
| [first deployment/](first%20deployment/) | Un Deployment nginx avec 2 replicas |
| [namespaces/](namespaces/) | Création du namespace `production` |
| [gestion de reseau/](gestion%20de%20reseau/) | Deux Pods (rouge et bleu) exposés par un Service NodePort |

> **Remarque :** certains dossiers contiennent des espaces. Dans le terminal, mets le chemin entre guillemets, par exemple `kubectl apply -f "gestion de reseau/pod-red.yaml"`.

Toutes les commandes ci-dessous se lancent depuis la racine du dépôt.

---

## 0. Prérequis

- **[OrbStack](https://orbstack.dev/)** installé (il fournit un cluster Kubernetes local sur macOS).
- **kubectl**, l'outil en ligne de commande pour piloter Kubernetes (fourni avec OrbStack).

Vérifier que kubectl est disponible :

```bash
kubectl version --client
```

---

## 1. Démarrer le cluster et s'y connecter

```bash
orb start k8s
```

Démarre le cluster Kubernetes d'OrbStack.

```bash
kubectl get nodes
```

Affiche les machines (nœuds) du cluster. Si un nœud apparaît avec le statut `Ready`, le cluster fonctionne et le terminal y est bien connecté.

---

## 2. Déployer un Pod

Le fichier [pods/pod.yaml](pods/pod.yaml) décrit un **Pod** : la plus petite unité dans Kubernetes, qui fait tourner un ou plusieurs conteneurs. Ici, il lance l'application `simple-webapp-color` avec un fond **rouge** (variable `APP_COLOR`).

```bash
kubectl apply -f pods/pod.yaml
```

- `apply` : crée (ou met à jour) la ressource décrite dans le fichier.
- `-f` : indique le chemin du fichier YAML à utiliser (chemin local, depuis le dossier où tu te trouves).

### Vérifier que le Pod tourne

```bash
kubectl get pods -o wide
```

Liste les Pods. L'option `-o wide` ajoute des infos (adresse IP, nœud…).

Résultat attendu : la colonne `STATUS` affiche `Running`. Si elle affiche `ContainerCreating`, patiente quelques secondes (l'image est en cours de téléchargement) et relance la commande.

### Accéder à l'application

Par défaut, un Pod n'est pas accessible depuis l'extérieur du cluster. On crée donc un « tunnel » entre ta machine et le Pod :

```bash
kubectl port-forward pod-simple-webapp-color 8080:8080 --address 0.0.0.0
```

- `8080:8080` : le port **8080 de ta machine** est relié au port **8080 du conteneur**.
- `--address 0.0.0.0` : rend l'application accessible aussi depuis les autres machines du réseau (sans cette option, uniquement depuis ta machine).

> **Attention :** la commande reste active tant que le terminal est ouvert. Pour l'arrêter : `Ctrl + C`.

Ouvre **http://localhost:8080** : tu dois voir une page avec un fond rouge.

---

## 3. Premier Deployment : nginx

Le fichier [first deployment/pod-nginx.yaml](first%20deployment/pod-nginx.yaml) ne décrit pas un simple Pod mais un **Deployment** : Kubernetes s'occupe de maintenir en permanence **2 copies (replicas)** du serveur web nginx. Si un Pod plante, il est automatiquement recréé.

Déployer :

```bash
kubectl apply -f "first deployment/pod-nginx.yaml"
```

Vérifier le Deployment et ses Pods :

```bash
kubectl get deployments
kubectl get pods -l app=nginx
```

`-l app=nginx` filtre les Pods ayant le label `app: nginx` (défini dans le fichier). Tu dois voir 2 Pods en `Running`.

Pour constater la recréation automatique, supprime un des Pods (remplace `<nom-du-pod>` par un nom affiché par la commande précédente) puis relance `kubectl get pods -l app=nginx` :

```bash
kubectl delete pod <nom-du-pod>
```

Un nouveau Pod apparaît aussitôt pour revenir à 2 replicas.

Accéder à nginx (port 80 dans le conteneur, 8081 sur ta machine pour ne pas entrer en conflit avec l'étape 2) :

```bash
kubectl port-forward deployment/deployment-nginx 8081:80
```

Puis ouvre **http://localhost:8081** : la page « Welcome to nginx! » doit s'afficher.

---

## 4. Namespaces

Un **namespace** est un espace de rangement à l'intérieur du cluster. Il permet de séparer les ressources (par exemple `production` et `developpement`) : deux ressources peuvent porter le même nom si elles sont dans des namespaces différents.

Jusqu'ici, tout a été créé dans le namespace `default`. Le fichier [namespaces/namespace.yaml](namespaces/namespace.yaml) crée un namespace nommé `production` :

```bash
kubectl apply -f namespaces/namespace.yaml
```

Lister les namespaces :

```bash
kubectl get namespaces
```

`production` doit apparaître avec le statut `Active`.

Pour voir les ressources d'un namespace précis, on ajoute `-n <namespace>` aux commandes :

```bash
kubectl get pods -n production
```

Pour l'instant, la liste est vide (`No resources found`).

---

## 5. Gestion du réseau : Service NodePort

Un Pod a une adresse IP qui change à chaque recréation, et le `port-forward` ne vise qu'un seul Pod à la fois. Un **Service** résout ces deux problèmes : il offre un point d'accès fixe et répartit les requêtes entre tous les Pods qui portent un label donné.

Le dossier [gestion de reseau/](gestion%20de%20reseau/) contient :

- [pod-red.yaml](gestion%20de%20reseau/pod-red.yaml) et [pod-blue.yaml](gestion%20de%20reseau/pod-blue.yaml) : deux Pods `simple-webapp-color`, l'un rouge, l'autre bleu. Ils portent tous les deux le label `app: web` et sont placés dans le namespace `production`.
- [service-nodeport-web.yaml](gestion%20de%20reseau/service-nodeport-web.yaml) : un Service de type **NodePort** qui envoie le trafic vers les Pods ayant le label `app: web`.

Les trois ports du Service :

| Champ | Valeur | Rôle |
|---|---|---|
| `nodePort` | 30008 | Port ouvert sur le nœud, accessible depuis ta machine (plage autorisée : 30000 à 32767) |
| `port` | 80 | Port du Service à l'intérieur du cluster |
| `targetPort` | 8080 | Port du conteneur vers lequel le trafic est envoyé |

> **Important :** le namespace `production` doit exister avant de créer ces ressources (étape 4). Sinon, `kubectl` renvoie l'erreur `namespaces "production" not found`.

Créer les Pods et le Service :

```bash
kubectl apply -f "gestion de reseau/"
```

Donner un dossier à `-f` applique tous les fichiers YAML qu'il contient.

Vérifier :

```bash
kubectl get pods -n production --show-labels
kubectl get service -n production
kubectl get endpoints service-nodeport-web -n production
```

- `--show-labels` affiche les labels : les deux Pods doivent porter `app=web`.
- La colonne `PORT(S)` du Service affiche `80:30008/TCP`.
- Les `endpoints` listent les adresses IP des Pods reliés au Service : il doit y en avoir deux. Si la liste est vide, le `selector` du Service ne correspond aux labels d'aucun Pod.

### Tester la répartition de charge

Avec OrbStack, les Services NodePort sont accessibles directement sur `localhost`. Ouvre **http://localhost:30008** : la page s'affiche en rouge ou en bleu selon le Pod qui a répondu.

Le navigateur garde souvent la même connexion ouverte, et donc le même Pod. Pour bien voir l'alternance, envoie plusieurs requêtes depuis le terminal :

```bash
for i in $(seq 1 10); do curl -s http://localhost:30008 | grep -o "Hello from [^!]*"; done
```

Les réponses doivent alterner entre `red-pod-simple-webapp-color` et `blue-pod-simple-webapp-color`.

---

## 6. Nettoyer

Supprimer les ressources créées :

```bash
kubectl delete -f pods/pod.yaml
kubectl delete -f "first deployment/pod-nginx.yaml"
kubectl delete -f "gestion de reseau/"
kubectl delete -f namespaces/namespace.yaml
```

> **À savoir :** supprimer un namespace supprime aussi tout ce qu'il contient. `kubectl delete namespace production` suffirait donc à retirer les Pods rouge et bleu et le Service.

Arrêter le cluster :

```bash
orb stop k8s
```

---

## Récapitulatif des commandes

| Action | Commande |
|---|---|
| Démarrer le cluster | `orb start k8s` |
| Voir les nœuds | `kubectl get nodes` |
| Créer une ressource | `kubectl apply -f <fichier>.yaml` |
| Créer toutes les ressources d'un dossier | `kubectl apply -f <dossier>/` |
| Voir les Pods | `kubectl get pods -o wide` |
| Voir les Pods d'un namespace | `kubectl get pods -n <namespace>` |
| Voir les labels des Pods | `kubectl get pods --show-labels` |
| Voir les Deployments | `kubectl get deployments` |
| Voir les namespaces | `kubectl get namespaces` |
| Voir les Services | `kubectl get service -n <namespace>` |
| Voir les Pods reliés à un Service | `kubectl get endpoints <service> -n <namespace>` |
| Accéder à une app | `kubectl port-forward <pod> <port-local>:<port-conteneur>` |
| Supprimer une ressource | `kubectl delete -f <fichier>.yaml` |
| Arrêter le cluster | `orb stop k8s` |
