# Démonstration  Knative Cli


## Deploiment d'un service en utilisant le client `kn`

* Deploiment _kn service create [SERVICE_NAME] --image [IMAGE]_
    ex: Déployment du service greeting existant.
    ```
    kn service create greeter \
    --image quay.io/rhdevelopers/knative-tutorial-greeter:quarkus
    ```

* Récupérons url du service.
    ``` 
    export SVC_URL=`oc get rt greeter -o jsonpath={.status.url`
    ```

* Tester le service.
    ```
    curl $SVC_URL
    ```
    Result
    ```
    Hi  greeter => '9861675f8845' : 1
    ```

:eyeglasses: regardez ce qui arrive du côté OpenShift, scale up and down.

---
## Update d'un service en utilisant le client `kn`

* Update du service de greeting
    ```
    kn service update greeter --env "MESSAGE_PREFIX=Namaste"
    ```

* List the revision
```
 kn revision list -s greeter
```

Result
```
NAME            SERVICE   TRAFFIC   TAGS   GENERATION   AGE    CONDITIONS   READY   REASON
greeter-00002   greeter   100%             2            86m    3 OK / 4     True
greeter-00001   greeter                    1            109m   3 OK / 4     True
```


* Tester le service.
    ```
    curl $SVC_URL
    ```
    Result
    ```
    Namaste  greeter => '9861675f8845' : 1
    ```
---


:construction: __CLEAN UP__
```
kn service delete greeter
```
