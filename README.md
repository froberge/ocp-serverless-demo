# Demo Red Hat Serverless on OpenShift ocp-serverless-demo

Voici un tutoriel sur  [Red Hat Serverless](https://www.redhat.com/en/technologies/cloud-computing/openshift/serverless)


## Red Hat Serverless, Knative sur Red Hat OpenShift.

Le modèle de cloud computing serverless fournit aux développeurs une environnement de développement d'applications cloud native moderne pour les clouds hybrides. Serverless permet aux développeurs de se concentrer sur leur code sans se soucier de l'infrastructure. `Red Hat OpenShift Serverless` vous permet d'éviter d'être enfermé dans un seul fournisseur et permet une approche multicloud.

 [Knative](https://knative.dev/docs/) est un project Open-Source de niveau d'entreprise qui permet de construire des applications Serverless et Event-Driven. `Knative` est une solution qui est plaforme agnostique pour rouler des applicaitons serverless.


### Pourquoi utiliser le Serverless.

Serverless fait référence à l'utilisation d'application et de processus qui roule dans le cloud. Serverless fonctionne en mode "as-used", ce qui veut dire que la compagnie paie seulement pour les ressources qui sont utilisées par l'exécution des applications. Serverless avec `Knative permet plusieurs avantages:
* Abstraction Simplifier: Simplifie la création des yaml grâce au Custom CRD.
* AutoScaling: Permet de scaler une applicaiton de 0 a N aux besoins
* Progressive Rollout: Permets de choisir la stratégie de "rollout" selon vos besoins.
* Event Integration": Permets l'intégration avec plusieurs types d'évents de plusieurs sources différentes.
* Handle Even: Grâce aux event broker, on peut manipuler les events
* _Kubernetes Native_ fait pour Kubernetes et est extensible comme celui-ci.


### Table des matière
* [Prérequis](#prérequis)
* [Installation de Serverless](#installation)
* [Composante de Knative](#composante-de-knative)
* [Demo Knative cli](docs/knative-cli-example.md)
* [Demo Knative Serving](docs/knative-serving.md)
* [Demo Knative Eventing](docs/knative-eventing.md)

##### Prérequis
* Un cluster OpenShift
* Un compte admin
* Un accès au cluster par CLI
* Accès au code de ce repository
* [Optionel] Knativve client 


#### Installation

L'installation de l'opérateur Serverless, avec le  script kustomize suivant.
```
until oc apply -k setup/overlays/demo
do
  sleep 20
done
```


#### Composante de Knative

`Knative` contient 2 composantes principales. Les 2 composantes fonctionnent ensemble pour automatiser et gérer les taches et applications des équipes qui travaillent dans un environnement cloud natif.

__Serving__ permet de rouler des application serverless dans un monde Kubernetes de façon simple, rapide et sans effort. Knative s'occupe des différents détails comme le réseautage, l'autoscaling ainsi que la supervision des révisions. Avec `serving` les équipes peuvent se concentrer sur la logique du code dans le langage de programmation de leur choix.

__Eventing__ Livraison, gestion et abonnement universel aux évènements. `Eventing` nous permettent de construire des applications modernes en attachant un "compute" à un flux de données (stream).

![architecture](docs/images/knative-architecture.png)
