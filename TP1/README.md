# TP1 : Création d'un déploiement sur Kubernetes

Ce TP a pour but de vous faire interagir avec Kubernetes en y déployant des ressources.

Dans ce TP, nous allons :

- Déployer une application nginx
- Exposer cette application afin de pouvoir y accéder en interne
- Accéder au service web

## Déroulé du TP

1. Connexion à la plateforme Killercoda

    Killercoda est une plateforme gratuite permettant de tester plusieurs environnements. Nous allons utiliser cette plateforme pour tester Kubernetes sans avoir à mettre en place tout le cluster.

    Pour lancer un lab Kubernetes, accéder à [ce lien](https://killercoda.com/playgrounds/scenario/kubernetes) puis se connecter avec votre email, votre compte GitHub ou autre.

    > **Warning**  
    > Il est important de noter que **l'environnement donné se réinitialise 1 heure après** lancement.

2. Interagir avec kubectl

    Le lab lancé vous donne accès à une machine avec `kubectl` d'installé. Pour le vérifier, on peut exécuter la commande suivante :

    ```bash
    $ kubectl version
    WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.
    Client Version: version.Info{Major:"1", Minor:"27", GitVersion:"v1.27.1", GitCommit:"4c9411232e10168d7b050c49a1b59f6df9d7ea4b", GitTreeState:"clean", BuildDate:"2023-04-14T13:21:19Z", GoVersion:"go1.20.3", Compiler:"gc", Platform:"linux/amd64"}
    Kustomize Version: v5.0.1
    Server Version: version.Info{Major:"1", Minor:"27", GitVersion:"v1.27.1", GitCommit:"4c9411232e10168d7b050c49a1b59f6df9d7ea4b", GitTreeState:"clean", BuildDate:"2023-04-14T13:14:42Z", GoVersion:"go1.20.3", Compiler:"gc", Platform:"linux/amd64"}
    ```

3. Créer un nouveau déploiement **nginx**

    On créé (`create`) une ressource de type `deployment` nommée `nginx` avec l'image `easylinux/kubernetes:nginx` :

    ```bash
    $ kubectl create deployment nginx --image=easylinux/kubernetes:nginx
    deployment.apps/nginx created
    ```

    Une fois la commande exécutée, on peut vérifier que le déploiement a été initié (on `get` les ressources de type `deployment`) :

    ```bash
    $ kubectl get deployment
    NAME    READY   UP-TO-DATE   AVAILABLE   AGE
    nginx   1/1     1            1           8s
    ```

    Le déploiement créé alors ce que l'on appelle un `pod` avec l'image `easylinux/kubernetes:nginx`. On peut vérifier cela avec la commande suivante :

    ```bash
    $ kubectl get pods 
    NAME                    READY   STATUS    RESTARTS   AGE
    nginx-c68cd5494-tgl8r   1/1     Running   0          22s
    ```

    > **Note**  
    > On remarquera que notre pod ne s'appelle pas `nginx` mais `nginx-<random>`. Cela est dû au fait qu'on passe par un déploiement pour déployer notre pod (nous verrons la raison plus en détail plus tard).

4. Exposer le port 80 de notre application nginx

    Maintenant que notre application nginx est lancé, on peut l'exposer avec la commande suivante :

    ```bash
    $ kubectl expose deployment nginx --type=NodePort --port 80
    service/nginx exposed
    ```

    Cette commande va assigner un port du cluster (`NodePort`) à notre application nginx. Pour récupérer le port assigné, on effectue la commande suivante :

    ```bash
    $ kubectl get services
    NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
    kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP        26d
    nginx        NodePort    10.102.223.164   <none>        80:30096/TCP   112s
    ```

    > **Note**  
    > La notion de `NodePort` peut porter à confusion, mais c'est bien un port du cluster qui est assigné au service. Par conséquent, on peut interroger n'importe quel noeud du cluster sur ce port et tomber sur le même service.

    Le port assigné est le port `30096`.

5. Récupérer les membres du cluster

    Pour tester notre application, nous allons récupérer les membres du cluster Kubernetes avec la commande suivante :

    ```bash
    $ kubectl get nodes --o wide
    NAME           STATUS   ROLES           AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
    controlplane   Ready    control-plane   26d   v1.27.1   172.30.1.2    <none>        Ubuntu 20.04.5 LTS   5.4.0-131-generic   containerd://1.6.12
    node01         Ready    <none>          26d   v1.27.1   172.30.2.2    <none>        Ubuntu 20.04.5 LTS   5.4.0-131-generic   containerd://1.6.12
    ```

    > **Note**  
    > L'option `-o wide` donne plus de détail dans le résultat de la commande exécutée.

6. Récupérer la page nginx sur tous les noeuds du cluster

    Les services de type `NodePort` exposent un port du cluster pour une application. Pour le prouver, nous allons interroger les deux membres de notre cluster sur le même port avec la commande `curl` :

    ```bash
    curl 172.30.1.2:30096
    curl 172.30.2.2:30096
    ```

    Les deux commandes nous retournent la même page nginx.

On peut maintenant supprimer toutes les ressources créées avec les commandes suivantes :

```bash
kubectl delete deployment nginx
kubectl delete service nginx
```
