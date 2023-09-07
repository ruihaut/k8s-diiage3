# TP3 : Chartiser notre déploiement initial

Maintenant que vous avez vu comment déployer des ressources sur Kubernetes de manière déclarative, je vous propose de découvrir à travers ce TP l'utilisation des charts HELM.

Dans ce TP et comme dans le TP précédent, nous allons :

- Déployer et exposer une application nginx via Helm
- Accéder au service web

## Prérequis : Installer Helm

### De quoi est-ce qu'on parle?

Pour rappel, Helm est un outil permettant de templatiser un déploiement sur Kubernetes. Dans ce TP, il servira notamment à déployer les ressources des TP précédents.

**Important** : il est important de noter que **helm n'est pas lié à un cluster**. Il s'agit d'un exécutable qui permet de créer, tester et de déployer des charts. Pour déployer des ressources sur Kubernetes, il va s'appuyer sur le kubeconfig (`.kube/config`), comme `kubectl`.

### Installation de Helm sous Linux

Pour installer Helm sur Linux, il suffit d'exécuter les commandes suivantes :

```bash
# Télécharger le binaire archivé
wget https://get.helm.sh/helm-v3.12.3-linux-amd64.tar.gz

# Désarchiver le fichier
tar -zxf helm-v3.12.3-linux-amd64.tar.gz

# Déplacer le binaire
mv linux-amd64/helm /usr/local/bin/helm
```

Vous pouvez maintenant utiliser helm :

```bash
helm --help
```

Pour les autres plateformes (Windows, Mac), suivez le [Guide d'installation officiel de Helm](https://helm.sh/docs/intro/install/).

## Créer une chart Helm à la main

### Créer une chart de base

Dans un premier temps, nous allons utiliser helm pour générer une chart de base.

1. Créer un environnement Killercoda.

    Pour suivre ce TP, je vous propose de refaire un nouvel environnement Killercoda.

    > **Note**  
    > Sur Killercoda, Helm est déjà installé.

2. Créer un répertoire dans lequel vous allez créer votre chart et vous déplacer dans ce répertoire

    ```bash
    mkdir charts
    cd charts
    ```

3. Créer une chart avec Helm

    Dans notre cas, étant donné qu'on veut créer une chart helm qui déploie un service nginx, on va appeler notre chart nginx :

    ```bash
    # helm create <nom_de_la_chart>
    helm create nginx
    ```

    On peut vérifier que les composants de notre chart ont bien été générés :

    ```bash
    $ ls -ls
    total 4
    4 drwxr-xr-x 4 root root 4096 Sep  7 19:35 nginx

    $ ls -ls nginx
    total 16
    4 -rw-r--r-- 1 root root 1141 Sep  7 19:35 Chart.yaml
    4 drwxr-xr-x 2 root root 4096 Sep  7 19:35 charts
    4 drwxr-xr-x 3 root root 4096 Sep  7 19:35 templates
    4 -rw-r--r-- 1 root root 1872 Sep  7 19:35 values.yaml
    ```

    Notre chart se compose de 3 éléments indispensables :

    - `Chart.yaml` - Contient tous les détails de la chart créée : **à modifier**
    - `templates` - Contient **tous les manifests Kubernetes** dont Helm va se servir : helm va remplacer toutes les variables contenues dans ces manifests par le contenu du fichier `values.yaml`
    - `values.yaml` - Contient toutes les variables nécessaires à la chart

4. Modification du fichier `values.yaml` pour déployer notre application nginx

    Tout se joue dans le `values.yaml`. C'est dedans qu'on va spécifier notamment **l'image** qu'on utilise pour notre déploiement, et **le service** (et notamment le **type de service**) qu'on va utiliser.

    On va commencer par l'éditer :

    ```bash
    cd nginx && nano values.yaml 
    ```

    On voit qu'il y a beaucoup de variables, mais on va s'attarder à l'une d'entre elles pour commencer : `image`.

    Dans le bloc `image`, on voit que notre chart va déployer des pods avec l'image nginx :

    ```yaml
    image:
        repository: nginx
        pullPolicy: IfNotPresent
        # Overrides the image tag whose default is the chart appVersion.
        tag: ""
    ```

    Dans nos TP précédents cependant, on avait pris une autre image. On fait la modification :

    ```yaml
    image:
        repository: easylinux/kubernetes
        pullPolicy: IfNotPresent
        # Overrides the image tag whose default is the chart appVersion.
        tag: "nginx"
    ```

    > **Note**  
    > easylinux/kubernetes:nginx de base se décompose comme suit : `repo = easylinux/kubernetes`, `tag = nginx`  
    > En passant, on aurait très bien pu utiliser une autre image qui n'a absolument rien à voir avec nginx

    Maintenant qu'on a défini l'image, au tour du service. Toujours dans le même fichier, rechercher le bloc de configuration `service` :

    ```yaml
    service:
        type: ClusterIP
        port: 80
    ```

    On le type en `NodePort` :

    ```yaml
    service:
        type: NodePort
        port: 80
    ```

    Sauvegarder le fichier. La chart est maintenant prête à être déployée.

### Déployer notre chart

Maintenant que la chart est prête, on peut la déployer avec la commande suivante :

```bash
helm install nginx . --values values.yaml
```

Notre chart installe s'installe. On peut faire un get des pods pour le constater :

```bash
$ kubectl get pods 
NAME                     READY   STATUS    RESTARTS   AGE
nginx-7b695585bb-w9lrm   1/1     Running   0          86s
```

Pour lister notre déploiement helm, on fait la commande suivante :

```bash
$ helm list 
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
nginx   default         1               2023-09-07 20:07:33.499031083 +0000 UTC deployed        nginx-0.1.0     1.16.0
```

### Déployer la chart dans un autre namespace

Finalement, on voudrait déployer notre appli sur un autre namespace `nginx-app`. Dans ce cas, on commence par la supprimer :

```bash
$ helm delete nginx
release "nginx" uninstalled
```

Et on redéploie avec la commande suivante :

```bash
helm install nginx . --values values.yaml --namespace nginx-app --create-namespace
```

Pour voir notre pod, on précise le namespace :

```bash
kubectl get pods -n nginx-app
```

### Upgrader notre chart

Finalement, 1 seul pod ne suffit plus.. on en veut 5. Dans ce cas, modifier la variable `replicaCount` du fichier `values.yaml` :

```yaml
replicaCount: 5 
```

On sauvegarde le fichier et on effectue un upgrade de notre application :

```bash
helm upgrade nginx . --values values.yaml --namespace nginx-app
```

On vérifie que notre appli s'est bien mise à jour :

```bash
$ kubectl get pods -n nginx-app
NAME                     READY   STATUS    RESTARTS   AGE
nginx-7b695585bb-5s429   1/1     Running   0          8m41s
nginx-7b695585bb-hq527   1/1     Running   0          3m45s
nginx-7b695585bb-k2z5f   1/1     Running   0          3m45s
nginx-7b695585bb-qc2l2   1/1     Running   0          3m45s
nginx-7b695585bb-qsclf   1/1     Running   0          3m45s
```

## La vraie force de helm : utiliser des charts existantes

Reste plus qu'à faire ça pour tous les services à installer sur kube...

Et si je vous disais que pour la plupart des services, **tout ça, quelqu'un l'avait déjà fait pour vous**? Vous ne me croyez pas? Rendez-vous sur [ce lien](https://artifacthub.io/) et recherchez ce que vous voulez (entre autres) :

- nginx
- keycloak
- rabbitmq

Tout y est. Et encore plus fort, pas besoin d'avoir la chart en local, tout se fait via des liens. Ainsi, toutes les commandes et options qu'on a vu plus haut (`--namespace nginx-app`, `--create-namespace` et autres) s'appliquent

### Récupérer les valeures par défaut d'une chart

Pour modifier les valeures d'une chart, il faut déjà les avoir. Pour cela, vous pouvez utiliser la commande suivante :

```bash
helm show values <chart>
# Exemple avec Keycloak
# helm show values oci://registry-1.docker.io/bitnamicharts/keycloak
```

> **Warning**  
> Sur Killercoda, la version de Helm installée est trop ancienne. Si vous tapez cette commande telle quel, ça ne va pas fonctionner. Vous pouvez cependant installer la dernière version sur le lab

Pour installer la dernière version de Helm sur le lab Killercoda, exécuter les commandes suivantes :

```bash
# Télécharger le binaire archivé
wget https://get.helm.sh/helm-v3.12.3-linux-amd64.tar.gz

# Désarchiver le fichier
tar -zxf helm-v3.12.3-linux-amd64.tar.gz

# Déplacer le binaire
mv linux-amd64/helm /usr/bin/helm
```

Réessayer la commande Helm :

```bash
helm show values oci://registry-1.docker.io/bitnamicharts/keycloak
```

Les values par défaut de la chart Keycloak devraient s'afficher.

Enfin, pour garder ces values pour les modifier, on peut récupérer le contenu comme suit :

```bash
helm show values oci://registry-1.docker.io/bitnamicharts/keycloak > values.yaml
```

Vous avez maintenant le values.yaml de la chart Keycloak que vous pouvez modifier comme vous voulez. Il ne vous reste plus qu'à déployer votre chart keycloak avec vos propres values.
