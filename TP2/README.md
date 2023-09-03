# TP2 : Déployer les ressources du TP1 de manière déclarative

Ce TP a pour but de vous faire découvrir la manière déclarative de déployer des ressources dans Kubernetes

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

2. 