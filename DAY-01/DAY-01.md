# TAKIMA 1

## Somaire

- [TAKIMA 1](#takima-1)
  - [Somaire](#somaire)
  - [Kubeconfig](#Kubeconfig)
  - [Premières commandes](#premières-commandes)
- [Etape 1 : Premières ressources : Pods/Replicaset/Deployment](#etape-1--premières-ressources--podsreplicasetdeployment)
  - [ReplicaSet](#ReplicaSet)
  - [Deployment](#Deployment)
  - [Faire un Rollback](#Faire-un-Rollback)
  - [Mettre à l'échelle](#mettre-à-léchelle)
  - [Mettre en standby un deploiement](#Mettre-en-standby-un-deploiement)
- [Etape 2 : Publication Service/Ingress](#etape-2--publication-serviceingress)
- [Etape 3 : ConfigMap/Secret](#etape-3--configmapsecret)

## Kubeconfig

### Quelles sont les informations que l'on retrouve dans ce fichier ?

Le fichier kubeconfig contient les informations de configuration de l'authentification et de l'accès au cluster Kubernetes. Il contient notamment les informations suivantes :

## Premières commandes

### Quelle est la différence ?
#### Entre `kubectl get pods -n votre_namespace` et `kubectl get pods -n default`

Il n'y a pas de différence car la configuration que l'on a donné à Kubernetes défini le namespace `votre_namespace` comme namespace par défaut.

## Etape 1 : Premières ressources : Pods/Replicaset/Deployment

### Quelles sont les propriétés principales que l'on retrouve ?

- apiVersion
- kind
- metadata
- spec

### Que se passe-t-il lors de cette deuxième création en imperatif ?

Lors du deuxième `kubectl run mynginx --image registry.takima.io/school/proxy/nginx` une erreur est levée car le pod `mynginx` existe déjà. En effet, le nom du pod doit être unique.

### Que se passe-t-il lors de cette deuxième création en déclaratif ?

```bash
❯ kubectl apply -f mynginx.yml

Warning: resource pods/mynginx is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
pod/mynginx configured
```

Lors du lancement en déclaratif, au lieu de recréer un Pod, Kubernetes met à jour le Pod existant. 

## ReplicaSet

### Exemple

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: unicorn-front-replicaset
  labels:
    app: unicorn-front
spec:
  template:
    metadata:
      name: unicorn-front-pod
      labels:
        app: unicorn-front
    spec:
      containers:
      - name: unicorn-front
        image: registry.takima.io/school/proxy/nginx
  replicas: 3
  selector:
    matchLabels:
      app: unicorn-front
```

### Que remarquez-vous dans la description des properties `spec: template` ? 

Le `spec: template` contient la description du pod qui sera créé par le ReplicaSet.

### À quoi sert le `selector: matchLabels` ?

Le `selector: matchLabels` permet de sélectionner les pods qui seront gérés par le ReplicaSet. 
Ici, le ReplicaSet va gérer les pods qui ont le label `app: unicorn-front`.

### Combien y a-t'il de pods déployés dans votre namespace ?

```bash
❯ kubectl get pods -n mathis-medard

NAME                             READY   STATUS    RESTARTS   AGE
unicorn-front-replicaset-7bl5p   1/1     Running   0          89s
unicorn-front-replicaset-84rxq   1/1     Running   0          89s
unicorn-front-replicaset-r4z4q   1/1     Running   0          89s
```

Trois pods ont été déployés dans le namespace `mathis-medard`.

### Maintenant, supprimez un Pod. Que se passe-t'il ?

```bash
❯ kubectl delete pod unicorn-front-replicaset-7bl5p

pod "unicorn-front-replicaset-7bl5p" deleted

❯ kubectl get pods -n mathis-medard

NAME                             READY   STATUS    RESTARTS   AGE
unicorn-front-replicaset-84rxq   1/1     Running   0          3m34s
unicorn-front-replicaset-h294b   1/1     Running   0          4s
unicorn-front-replicaset-r4z4q   1/1     Running   0          3m34s
```

Le Pod a bien été supprimé et un nouveau Pod a été créé par le ReplicaSet pour le remplacer.

### Supprimez le ReplicaSet. Que se passe-t'il ?

```bash
❯ kubectl delete replicaset unicorn-front-replicaset
replicaset.apps "unicorn-front-replicaset" deleted

❯ kube get pods
No resources found in mathis-medard namespace.
```

Le ReplicaSet a été supprimé et les Pods qu'il gérait ont aussi été supprimés.

## Deployment

La ressource Deployment est très proche de celle d’un ReplicaSet :
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: unicorn-front-deployment
  labels:
    app: unicorn-front
spec:
  replicas: 3
  selector:
    matchLabels:
      app: unicorn-front
  template:
    metadata:
      labels:
        app: unicorn-front
    spec:
      containers:
      - name: unicorn-front
        image: registry.takima.io/school/proxy/nginx:1.7.9
        ports:
        - containerPort: 80
```

### Quels sont les changements par rapport au ReplicaSet ?

Il n'y a pas de changement par rapport au ReplicaSet. 
Le Deployment est une ressource de plus haut niveau qui permet de gérer les ReplicaSets.

### Déployez ce Deployment. Combien y a-t'il de ReplicaSet ? De Pods ?

```bash
❯ kubectl apply -f unicorn-front-deployment.yml
deployment.apps/unicorn-front-deployment created

❯ kube get pods                                                     
NAME                                       READY   STATUS    RESTARTS   AGE
unicorn-front-deployment-644bd5bb5-b4xfd   1/1     Running   0          9s
unicorn-front-deployment-644bd5bb5-kvvz7   1/1     Running   0          9s
unicorn-front-deployment-644bd5bb5-ldpjq   1/1     Running   0          9s
```

Il y a un seul ReplicaSet et trois Pods.

```bash
❯ kubectl get all
NAME                                           READY   STATUS    RESTARTS   AGE
pod/unicorn-front-deployment-644bd5bb5-b4xfd   1/1     Running   0          3m51s
pod/unicorn-front-deployment-644bd5bb5-kvvz7   1/1     Running   0          3m51s
pod/unicorn-front-deployment-644bd5bb5-ldpjq   1/1     Running   0          3m51s

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/unicorn-front-deployment   3/3     3            3           3m52s

NAME                                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/unicorn-front-deployment-644bd5bb5   3         3         3       3m52s

```

### Une fois terminé, combien y a-t-il de replicaset ? 

Il existe deux ReplicaSets. Mais le premier est orphelin car il ne gère plus aucun Pod.

### Combien y a-t-il de Pods ? 

Il y a toujours trois Pods.

### Allez voir les logs des événements du déploiement avec `kubectl describe deployments. Qu’observez vous ?

```bash
❯ kubectl describe deployments
Name:                   unicorn-front-deployment
Namespace:              mathis-medard
CreationTimestamp:      Mon, 27 Nov 2023 11:25:46 +0100
Labels:                 app=unicorn-front
Annotations:            deployment.kubernetes.io/revision: 2
                        kubernetes.io/change-cause:
                          kubectl set image deployment/unicorn-front-deployment unicorn-front=registry.takima.io/school/proxy/nginx:1.9.1 --record=true
Selector:               app=unicorn-front
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=unicorn-front
  Containers:
   unicorn-front:
    Image:        registry.takima.io/school/proxy/nginx:1.9.1
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  unicorn-front-deployment-644bd5bb5 (0/0 replicas created)
NewReplicaSet:   unicorn-front-deployment-66f8d44ccf (3/3 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  11m    deployment-controller  Scaled up replica set unicorn-front-deployment-644bd5bb5 to 3
  Normal  ScalingReplicaSet  5m17s  deployment-controller  Scaled up replica set unicorn-front-deployment-66f8d44ccf to 1
  Normal  ScalingReplicaSet  5m15s  deployment-controller  Scaled down replica set unicorn-front-deployment-644bd5bb5 to 2
  Normal  ScalingReplicaSet  5m15s  deployment-controller  Scaled up replica set unicorn-front-deployment-66f8d44ccf to 2
  Normal  ScalingReplicaSet  5m13s  deployment-controller  Scaled down replica set unicorn-front-deployment-644bd5bb5 to 1
  Normal  ScalingReplicaSet  5m13s  deployment-controller  Scaled up replica set unicorn-front-deployment-66f8d44ccf to 3
  Normal  ScalingReplicaSet  5m11s  deployment-controller  Scaled down replica set unicorn-front-deployment-644bd5bb5 to 0
```

On observe que le déploiement a été mis à jour. 
Le ReplicaSet `unicorn-front-deployment-644bd5bb5` a été vidé et le ReplicaSet `unicorn-front-deployment-66f8d44ccf` a été créé et rempli avec les nouveaux Pod.

## Faire un Rollback

Mettez à jour votre déploiement avec une nouvelle version de l’image nginx.

```bash
❯ kubectl set image deployment.v1.apps/unicorn-front-deployment unicorn-front=registry.takima.io/school/proxy/nginx:1.91-falseimage --record=true
```
Observez votre Deployment, vos Pods et vos ReplicaSets.

### Que se passe-t'il ? Pourquoi ?

```bash
❯ kubectl rollout status deployment.v1.apps/unicorn-front-deployment                                            
Waiting for deployment "unicorn-front-deployment" rollout to finish: 1 out of 3 new replicas have been updated...

❯ kubectl get all                                                                                                                                
NAME                                            READY   STATUS             RESTARTS   AGE
pod/unicorn-front-deployment-66f8d44ccf-5xnkg   1/1     Running            0          9m45s
pod/unicorn-front-deployment-66f8d44ccf-8vp55   1/1     Running            0          9m47s
pod/unicorn-front-deployment-66f8d44ccf-wxspl   1/1     Running            0          9m49s
pod/unicorn-front-deployment-6c4646748d-x6sv2   0/1     ImagePullBackOff   0          2m5s

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/unicorn-front-deployment   3/3     1            3           16m

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/unicorn-front-deployment-644bd5bb5    0         0         0       16m
replicaset.apps/unicorn-front-deployment-66f8d44ccf   3         3         3       9m49s
replicaset.apps/unicorn-front-deployment-6c4646748d   1         1         0       2m5s
```

Le nouveau ReplicaSet `unicorn-front-deployment-6c4646748d` a été créé mais les Pods ne sont pas prêts car l'image n'a pas pu être téléchargée.

#### Dans ce cas de figure, l'objectif est de revenir à une version stable de `Deployment`

Pour commencer, vérifiez les révisions de ce déploiement :

```bash
❯ kubectl rollout history deployment.v1.apps/unicorn-front-deployment

deployment.apps/unicorn-front-deployment 
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment/unicorn-front-deployment unicorn-front=registry.takima.io/school/proxy/nginx:1.9.1 --record=true
3         kubectl set image deployment.v1.apps/unicorn-front-deployment unicorn-front=registry.takima.io/school/proxy/nginx:1.91-falseimage --record=true
```

### Combien y a-t-il de révisions ? À quoi correspond le champ CHANGE-CAUSE ?

Il y a trois révisions. Le champ `CHANGE-CAUSE correspond à la commande qui a été exécutée pour mettre à jour le déploiement.

#### Pour revenir à la révision 2, vous pouvez utiliser la commande suivante :

```bash
❯ kubectl rollout undo deployment.v1.apps/unicorn-front-deployment --to-revision=2

deployment.apps/unicorn-front-deployment rolled back
```

## Mettre à l'échelle

Pour scaler le serveur nginx à 5:
    
```bash
❯ kubectl scale deployment.v1.apps/unicorn-front-deployment --replicas=5

deployment.apps/unicorn-front-deployment scaled
```

### Combien y a-t'il de Pods?

```bash
❯ kubectl get all                                                                 

NAME                                            READY   STATUS    RESTARTS   AGE
pod/unicorn-front-deployment-66f8d44ccf-5xnkg   1/1     Running   0          17m
pod/unicorn-front-deployment-66f8d44ccf-6hvc6   1/1     Running   0          97s
pod/unicorn-front-deployment-66f8d44ccf-8vp55   1/1     Running   0          17m
pod/unicorn-front-deployment-66f8d44ccf-rkgbb   1/1     Running   0          97s
pod/unicorn-front-deployment-66f8d44ccf-wxspl   1/1     Running   0          17m

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/unicorn-front-deployment   5/5     5            5           24m

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/unicorn-front-deployment-644bd5bb5    0         0         0       24m
replicaset.apps/unicorn-front-deployment-66f8d44ccf   5         5         5       17m
replicaset.apps/unicorn-front-deployment-6c4646748d   0         0         0       9m58s
```

Il y a cinq Pods maintenant.

## Mettre en standby un deploiement

Mettez en pause un Deployment :
```bash
❯ kubectl rollout pause deployment.v1.apps/unicorn-front-deployment
deployment.apps/unicorn-front-deployment paused

❯ kubectl set image deployment.v1.apps/unicorn-front-deployment unicorn-front=registry.takima.io/school/proxy/nginx:1.20.2
deployment.apps/unicorn-front-deployment image updated
```

### Que se passe-t-il au niveau ReplicaSet ?

```bash
❯ kubectl get all
NAME                                            READY   STATUS    RESTARTS   AGE
pod/unicorn-front-deployment-66f8d44ccf-5xnkg   1/1     Running   0          30m
pod/unicorn-front-deployment-66f8d44ccf-6hvc6   1/1     Running   0          14m
pod/unicorn-front-deployment-66f8d44ccf-8vp55   1/1     Running   0          30m
pod/unicorn-front-deployment-66f8d44ccf-rkgbb   1/1     Running   0          14m
pod/unicorn-front-deployment-66f8d44ccf-wxspl   1/1     Running   0          30m

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/unicorn-front-deployment   5/5     0            5           36m

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/unicorn-front-deployment-644bd5bb5    0         0         0       36m
replicaset.apps/unicorn-front-deployment-66f8d44ccf   5         5         5       30m
replicaset.apps/unicorn-front-deployment-6c4646748d   0         0         0       22m
```

Il ne se passe rien au niveau des ReplicaSets car ils sont en pause.

```bash
❯ kubectl rollout resume deployment.v1.apps/unicorn-front-deployment
```

### Que se passe-t-il au niveau ReplicaSet ?

```bash
❯ kubectl get all
NAME                                            READY   STATUS    RESTARTS   AGE
pod/unicorn-front-deployment-6d6c86587c-9f55w   1/1     Running   0          64s
pod/unicorn-front-deployment-6d6c86587c-ff556   1/1     Running   0          64s
pod/unicorn-front-deployment-6d6c86587c-g59dm   1/1     Running   0          64s
pod/unicorn-front-deployment-6d6c86587c-hmhns   1/1     Running   0          62s
pod/unicorn-front-deployment-6d6c86587c-t5ltp   1/1     Running   0          62s

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/unicorn-front-deployment   5/5     5            5           5m39s

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/unicorn-front-deployment-644bd5bb5    0         0         0       36m
replicaset.apps/unicorn-front-deployment-6c4646748d   0         0         0       30m
replicaset.apps/unicorn-front-deployment-66f8d44ccf   5         5         5       22m
```

## Etape 2 : Publication Service/Ingress

### Mise en situation

### Que se passe-t'il ? Pourquoi ?

```bash
❯ kube get all                                  
NAME                                             READY   STATUS         RESTARTS   AGE
pod/simple-website-deployment-5566cdf4b8-hrfjr   0/1     ErrImagePull   0          4s
pod/simple-website-deployment-5566cdf4b8-lr4v2   0/1     ErrImagePull   0          4s
pod/simple-website-deployment-5566cdf4b8-pssbm   0/1     ErrImagePull   0          4s

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/simple-website-deployment   0/3     3            0           4s

NAME                                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/simple-website-deployment-5566cdf4b8   3         3         0       4s
```

Les Pods ne sont pas prêts car l'image n'a pas pu être téléchargée.

### Décrivez ce que répond la Web App ? Actualisez votre page avec CTRL + F5. Que se passe-t-il ?

Une page avec un fond coloré et le texte 
```
Je suis le Taki-Pod
situé sur le noeud

avec l'IP
```

En actualisant la page, la couleur du fond change

Maintenant, nous allons ajouter des configs de variables d'environement.

### Que constatez-vous sur le navigateur ?

Maintenant, en plus de la couleur de fond, nous avons le texte
```
Je suis le Taki-Pod simple-website-deployment-794f8d4c95-wf5w5
situé sur le noeud ip-10-30-7-190.eu-west-3.compute.internal

avec l'IP 10.30.7.70
```

Et en actualisant la page, nous changeons de Pod, de Noeud et d'IP

## Etape 3 : ConfigMap/Secret

FINI