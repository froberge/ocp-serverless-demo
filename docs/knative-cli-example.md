# Exemple de commande avec Knative CLI


## Déploiement d'un service

* Deploiment _kn service create [SERVICE_NAME] --image [IMAGE]_
    ```
    kn service create greeter \
    --image quay.io/rhdevelopers/knative-tutorial-greeter:quarkus
    ```

* Récupérons url du service.
    ``` 
    export SVC_URL=`oc get rt greeter -o jsonpath={.status.url`
    ```
---
## Update d'un service 

* Update du service de greeting
    ```
    kn service update greeter --env "MESSAGE_PREFIX=Namaste"
    ```

* Lister les révision
    ```
    kn revision list -s greeter
    ```

    Result
    ```
    NAME            SERVICE   TRAFFIC   TAGS   GENERATION   AGE    CONDITIONS   READY   REASON
    greeter-00002   greeter   100%             2            86m    3 OK / 4     True
    greeter-00001   greeter                    1            109m   3 OK / 4     True
    ```
---

* Tag une révision
    ```
    kn service update greeter --tag=greeter-00002=latest
    ```


:construction: __CLEAN UP__
```
kn service delete greeter
```
