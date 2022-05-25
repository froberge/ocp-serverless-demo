# Démonstration  Eventing avec Kafka


## Prerequis
* Installation de Kafka sur OpenShift. voir [ocp-kafka-demo](https://github.com/froberge/ocp-kafka-demo) pour installation si c'est pas fait.


`CloudEvents` est une spécification pour décrire les données d'événement d'une manière commune. Un événement peut être produit par un nombre quelconque de sources (par exemple, Kafka, S3, GCP PubSub, MQTT) et en tant que développeur de logiciels, vous souhaitez une abstraction commune pour toutes les entrées d'événement.

## Pattern d'utilisation

Il existe 3 Pattern principale d'utilisation avec Knative Eventing

__Source to Sink__
Source to Service offre l'expérience de démarrage la plus simple avec Knative Eventing. Il fournit un seul récepteur, c'est-à-dire un service de réception d'événements, sans file d'attente, contre-pression et filtrage. La source vers le service ne prend pas en charge les réponses, ce qui signifie que la réponse du service Sink est ignorée. La responsabilité de la source de l'événement consiste simplement à transmettre le message sans attendre la réponse du récepteur.

![sourcetosink](images/source2sink.png)

__Channel & Subscription__
Avec le canal et subscription, le système Knative Eventing définit un canal, qui peut se connecter à divers backends tels que In-Memory, Kafka et GCP PubSub pour approvisionner les événements. Chaque canal peut avoir un ou plusieurs abonnés sous la forme de services Sink, qui peuvent recevoir les messages d'événement et les traiter selon les besoins. Chaque message du canal est formaté en tant que CloudEvent et envoyé plus haut dans la chaîne à d'autres abonnés pour un traitement ultérieur. Le modèle d'utilisation Canaux et abonnement n'a pas la capacité de filtrer les messages.

![channel-subscription](images/channel-subcription.png)

__Broker and Trigger__
Le broker et le trigger sont similaires au canal et subscription, sauf qu'ils prennent en charge le filtrage des événements. Le filtrage des événements est une méthode qui permet aux abonnés de manifester un intérêt pour un certain ensemble de messages qui transitent vers le courtier. Pour chaque courtier, Knative Eventing créera implicitement un canal Knative Eventing. Le trigger s'abonne au courtier et applique le filtre sur les messages de son courtier abonné. Les filtres sont appliqués sur les attributs Cloud Event des messages, avant de les livrer aux Sink Services (abonnés) intéressés.

![broker-trigger](images/broker-trigger.png)


## Installation de Knative Kafka

* Pour commencer nous devons faire une instance de Knative Kafka
    ```
    oc apply -f manifest/k8s/eventing/knative-kafka.yaml
    ```

    On devrait voir l'instance dans - >  Dans OpenShift, Install Operator, project knative-eventing, Knative Kafka
    ![Knative-kafka](images/knative-kafka.png)

* Céation d'un knative channel en Kafka. Par default les channel Knative sont InMemory.
Pour ce faire nous allons modifier le default Channel Config.
    ```
    oc apply -f manifest/k8s/eventing/default-channel-config.yaml
    ```

* Création d'un Knative channel. Comme le channel est de type Kafka, il va créer le kafka topic qui correspond.
    ```
    oc apply -f manifest/k8s/eventing/default-kafka-channel.yaml
    ```
    __Validation__
    Le channel Knative
    ```
     oc get -n demo-serverless channels
     ```
     ```
     NAME           URL                                                                AGE     READY   REASON
    my-events-ch   http://my-events-ch-kn-channel.demo-serverless.svc.cluster.local   3h17m   True
    ```

    Le channel kafka
    ```
     oc get -n demo-serverless kafkachannels
     ```
     ```
        NAME           READY   REASON   URL                                                                AGE
        my-events-ch   True             http://my-events-ch-kn-channel.demo-serverless.svc.cluster.local   7m35s
    ```

    Le topic
    ```
    oc get kafkatopics -n mykafka
    ```
    ![topic-list](images/topics-list.png)

:construction: __CLEAN UP__
```
oc -n demo-serverless delete  channels.messaging.knative.dev my-events-ch
```
---
    
## Connecter un source Kafka a un Sink

On sait maintenant que notre infrastructure fonctionne, on peut maintenant connecter un Knative serving servince a un Kafka.

* Deployer du service.
        ```
        oc apply -f manifest/k8s/eventing/eventing-hello-sink.yaml
        ```

* Valider la création du service.
    ```
    oc get ksvc -n demo-serverless
    ```

    ```
    NAME            URL                                                                                     LATESTCREATED         LATESTREADY           READY   REASON
    eventinghello   https://eventinghello-demo-serverless.apps.cluster-nrthl.nrthl.sandbox140.opentlc.com   eventinghello-00001   eventinghello-00001   True
    ```

* Regarger les logs du services avec stern
    ```
    stern eventinghello -c user-container
    ```

:warning: Assurez vous que le topics, my-topic a bien été créer a la création du kafka project.

* Faire la création d'une source Kafka
    ```
    oc apply -f manifest/k8s/eventing/kafka-source.yaml
    ```

    ![kafka-source-sink](images/kafka-source-sink.png)

* Tester
    ```
    kubectl -n mykafka run kafka-producer -ti \
    --image=quay.io/strimzi/kafka:0.26.1-kafka-3.0.0 \
    --rm=true --restart=Never \
    -- bin/kafka-console-producer.sh\
    --broker-list my-cluster-kafka-bootstrap:9092 \
    --topic my-topic
    ```
    
    _Au prompt on peut rentrer les messages qu'on veut._

* S'assurer de monitorer les logs sur sink pour voir la consomation des message.

___

## Auto Scaling avec Kafka

Maintenant faissons un test pour démontrer le auto-scalling car il y a trop de message qui entre.

* Deployer un application qui va agir comme un spammer.
    ```
    oc -n mykafka run kafka-spammer \
    --image=quay.io/rhdevelopers/kafkaspammer:1.0.2
    ```

* Un fois l'application deployé, entrer dans le pod
    ```
    oc -n mykafka exec -it kafka-spammer -- /bin/sh
    ```

* Spammer l'application
    ```
    curl localhost:8080/3
    ```

    __Ceci devrait forcer l'application eventinghello a scale up pods__
    ```
    oc get pods -n demo-serverless -w
    ```


:eyeglasses: Vous pouvez voir le scale up en monitorant les pods au niveau du command line ou dans l'interface web de OpenShift.

:construction: __CLEAN UP__
```

oc delete -n mykafka pod kafka-spammer
oc delete -n demo-serverless  -f manifest/k8s/eventing/kafka-source.yaml
oc delete -n demo-serverless  -f manifest/k8s/eventing/eventing-hello-sink.yaml
```
---