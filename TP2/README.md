# TP2 : Déployer les ressources du TP1 de manière déclarative

Ce TP a pour but de vous faire découvrir la manière déclarative de déployer des ressources dans Kubernetes.

Dans ce TP et comme dans le TP précédent, nous allons :

- Déployer et exposer une application nginx via un manifest YAML
- Accéder au service web

## Déroulé du TP

1. Réutiliser le même environnement Killercoda

    En théorie, vous avez effacé les ressources déployées dans le TP précédent. Si ce n'est pas le cas, exécutez les commandes suivantes :

    ```bash
    kubectl delete deployment nginx
    kubectl delete service nginx
    ```

2. Récupérer le manifest YAML de ce repo

    Sur le lab Killercoda, récupérer le manifest YAML de ce repo avec la commande suivante :

    ```bash
    wget https://raw.githubusercontent.com/ruihaut/k8s-diiage3/main/TP2/deployment.yaml
    ```

    Cette commande va télécharger le fichier `deployment.yaml`.

3. Appliquer le fichier téléchargé

    Exécuter la commande suivante pour appliquer la configuration renseignée dans le fichier :

    ```bash
    $ kubectl apply -f deployment.yaml
    deployment.apps/nginx created
    service/nginx created
    ```

    Alternativement, on peut appliquer le fichier directement sans avoir à le télécharger :

    ```bash
    kubectl apply -f https://raw.githubusercontent.com/ruihaut/k8s-diiage3/main/TP2/deployment.yaml
    deployment.apps/nginx created
    service/nginx created
    ```

    On peut vérifier que nos ressources ont bien été déployées comme ce fut le cas dans le TP1 :

    ```bash
    $ kubectl get deployment
    NAME    READY   UP-TO-DATE   AVAILABLE   AGE
    nginx   1/1     1            1           8s

    $ kubectl get services
    NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
    kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP        26d
    nginx        NodePort    10.102.223.164   <none>        80:30096/TCP   112s
    ```

Notre application nginx est maintenant déployée comme dans le TP1, à la seule différence qu'on a utilisé la méthode déclarative. Pour vérifier cela, vous pouvez **répéter les étapes 5 et 6 du TP1**.

## Explications sur le fichier YAML

Le fichier YAML contient 2 ressources :

- Une ressource de type `deployment`
- Une ressource de type `service`

Il est possible de renseigner ses deux ressources dans un unique fichier grâce au **triple-dash** (`---`). Le triple-dash permet de définir une fin de fichier au format YAML.

> **Note**  
> Bien que le triple-dash soit utile, on préfèrera renseigner chaque ressource dans son propre fichier. Cela facilite le templating des manifests, notamment avec [HELM](https://helm.sh/).

L'avantage d'utiliser des fichiers pour déployer des ressources est qu'**on peut définir toutes les options nécessaires**. On a ainsi une **vision globale et claire** de ce qu'on déploie.

### Section Deployment : options importantes

- **kind: Deployment** : défini le type de la ressource qu'on déploie
- **metadata.labels** : ensemble de clé / valeur. Les labels sont un concept important dans Kubernetes car ils permettent aux **selector** de pointer sur ces labels
- **spec.replicas** : défini le nombre de pod à créer
- **spec.selector** : sert au deployment à retrouver les pods quil a créé (en l'occurence, les pods avec le label "app: nginx")
- **spec.template** : défini le template de pod que le deployment doit créer (on y retrouve notamment les labels qu'on a défini dans `spec.selector`)

Plus de détails sur la [doc officielle de Kubernetes sur les deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/).

### Section Service : options importantes

- **kind: Service** : défini le type de la ressource qu'on déploie
- **spec.type** : défini le type de service qu'on veut déployer
- **spec.ports.port** : défini le port à exposer
- **spec.ports.targetPort** : défini le port du conteneur sur lequel le service doit taper
- **spec.selector** : sert au service pour qu'il puisse déterminer sur quel(s) pod(s) il doit rediriger ses requêtes (en l'occurence, les pods avec le label "app: nginx")

Plus de détails sur la [doc officielle de Kubernetes sur les services](https://kubernetes.io/docs/concepts/services-networking/service/).
