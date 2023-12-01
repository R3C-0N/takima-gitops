# TAKIMA 2

## SOMMAIRE

- [TAKIMA 2](#takima-2)
    - [Somaire](#sommaire)
    - [1ère étape : déployer l’API](#1ère-étape--déployer-lapi)
    - [2ᵉ étape : créer la base de données](#2ᵉ-étape--créer-la-base-de-données)
    - [3ᵉ étape : Faire pointer l’API sur la base de données](#3ᵉ-étape--faire-pointer-lapi-sur-la-base-de-données)
    - [4ᵉ étape : Rendre votre deployment parfait !](#4ᵉ-étape--rendre-votre-deployment-parfait-)
    - [5ᵉ étape : C'est au tour du Front.](#5ᵉ-étape--cest-au-tour-du-front)
    - [6ᵉ étape : La persistance dans Kubernetes](#6ᵉ-étape--la-persistance-dans-kubernetes)

## 1ère étape : déployer l’API

### Que se passe-t-il au niveau des Pods de l’API ? Vous pouvez jeter un oeil aux logs. (kubectl logs -f nomdupod)

```bash
► kube get all                    
        
NAME                                  READY   STATUS             RESTARTS      AGE
pod/api-deployment-58879987f4-5h5d2   0/1     CrashLoopBackOff   5 (21s ago)   3m37s
pod/api-deployment-58879987f4-l7kpl   0/1     CrashLoopBackOff   5 (24s ago)   3m37s
pod/api-deployment-58879987f4-xl465   0/1     CrashLoopBackOff   5 (21s ago)   3m37s

NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/api-service   ClusterIP   172.20.55.206   <none>        80/TCP    3m22s

NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/api-deployment   0/3     3            0           3m37s

NAME                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/api-deployment-58879987f4   3         3         0       3m37s
```

```bash
➜ kubectl logs -f pods/api-deployment-58879987f4-5h5d2 
Unknown argument: db.mathis-medard.takima.school
```

Les Pods de l'API ne peuvent pas se connecter à la base de données et ils crashent.

## 2ᵉ étape : créer la base de données

## 3ᵉ étape : Faire pointer l’API sur la base de données

### Quel est le nom du service de la base de données ?

```bash
➜ kubectl get services
NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
api-service   ClusterIP   172.20.55.206   <none>        80/TCP     39m
db-service    ClusterIP   172.20.7.20     <none>        5432/TCP   24m
```

Le service de la base de données est `db-service`

## 4ᵉ étape : Rendre votre deployment parfait !

## 5ᵉ étape : C'est au tour du Front.

### Pourquoi le computer a disparu ?

Il n'y a plus de computer car la base de données a été recréée et donc les données ont été perdues.
Il n'y a pas de volumes persistants.

### Vérifiez que le PVC est créé avec le PV. Quel est le nom du PV ?

```bash
➜ kubectl get pv

NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                STORAGECLASS   REASON   AGE
pvc-b6b2e04f-cc42-4f60-a9ec-ef896a9e7712   3Gi        RWO            Delete           Bound    mathis-medard/db-pvc gp2                     6m21s
```

Le nom du PV est `pvc-b6b2e04f-cc42-4f60-a9ec-ef896a9e7712`

## 6ᵉ étape : La persistance dans Kubernetes


## Bonus: CronJob

```bash
➜ kube logs pods/logical-backup-formation-cdb-manual-zsbzq-mathis-medard-672wl
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 17835    0 17835    0     0  17182      0 --:--:--  0:00:01 --:--:-- 17182
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 19642    0 19642    0     0  1475k      0 --:--:-- --:--:-- --:--:-- 1475k
+ dump
+ /usr/lib/postgresql/14/bin/pg_dumpall
+ compress
+ pigz
+ upload
+ case $LOGICAL_BACKUP_PROVIDER in
++ estimate_size
++ /usr/lib/postgresql/14/bin/psql -tqAc 'select sum(pg_database_size(datname)::numeric) from pg_database;'
+ aws_upload 7148808
+ declare -r EXPECTED_SIZE=7148808
++ date +%s
+ PATH_TO_BACKUP=s3://backup-pg-formation/spilo/formation-cdb/118b4050-9d2a-4057-bff7-fd39feb9fbca/logical_backups/1701188914.sql.gz
+ args=()
+ [[ ! -z 7148808 ]]
+ args+=("--expected-size=$EXPECTED_SIZE")
+ [[ ! -z '' ]]
+ [[ ! -z eu-west-3 ]]
+ args+=("--region=$LOGICAL_BACKUP_S3_REGION")
+ [[ ! -z AES256 ]]
+ args+=("--sse=$LOGICAL_BACKUP_S3_SSE")
+ aws s3 cp - s3://backup-pg-formation/spilo/formation-cdb/118b4050-9d2a-4057-bff7-fd39feb9fbca/logical_backups/1701188914.sql.gz --expected-size=7148808 --region=eu-west-3 --sse=AES256
+ [[ 0 != 0 ]]
+ [[ 0 != 0 ]]
+ [[ 0 != 0 ]]
+ set +x
```