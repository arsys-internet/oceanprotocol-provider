# Ocean Protocol Compute to Data with Helm

## Documentation

Official [Helm documentation](https://helm.sh/docs/)

Official Ocean Protocol documentation of [Compute to data infrastructure](https://docs.oceanprotocol.com/infrastructure/compute-to-data-minikube) list of available container images:

 - Provider:
    - repository: oceanprotocol/provider-py
    - tags: latest / v2.1.6
 - API:
    - repository: oceanprotocol/operator-service
    - tag: v4main / latest
 - Operator Engine:
    - repository: oceanprotocol/operator-engine
    - tags: latest / v1.1.2 / v4main / nvidia / nvidia-test
 - POD Configuration:
    - repository: oceanprotocol/pod-configuration
    - tags: sanitize-url / v4main / v1.1.1 / latest
  - Operator Publishing:
    - repository: oceanprotocol/pod-publishing
    - tag: v4main

[OceanProtocol Provider on Kubernetes](https://github.com/rogargon/oceanprotocol-provider-kubectl/) by Roberto Garc&iacute;a list container image used:

 - Provider:
   - repository: oceanprotocol/provider-py
   - tags: v2.1.3
 - API:
   - repository: oceanprotocol/operator-service
   - tag: v4main
 - Operator Engine:
   - repository: rodargon/operator-engine
   - tag: gke-gpu
 - POD Configuration:
   - repository: rodargon/pod-configuration
   - tag: timeout
 - Operator Publishing:
   - repository: oceanprotocol/pod-publishing
   - tag: v4main

## Directory structure

```plain
.
├── charts
│   └── oceanprotocol-provider
│       ├── charts
│       │   ├── ipfs
│       │   │   ├── Chart.yaml
│       │   │   ├── templates
│       │   │   │   ├── deployment.yaml
│       │   │   │   ├── ingress.yaml
│       │   │   │   ├── pvc.yaml
│       │   │   │   └── service.yaml
│       │   │   └── values.yaml
│       │   ├── operator-engine
│       │   │   ├── Chart.yaml
│       │   │   ├── templates
│       │   │   │   ├── deployment.yaml
│       │   │   │   ├── role-binding.yaml
│       │   │   │   ├── role.yaml
│       │   │   │   └── service-account.yaml
│       │   │   └── values.yaml
│       │   ├── operator-service
│       │   │   ├── Chart.yaml
│       │   │   ├── templates
│       │   │   │   ├── deployment.yaml
│       │   │   │   └── service.yaml
│       │   │   └── values.yaml
│       │   ├── postgres
│       │   │   ├── Chart.yaml
│       │   │   ├── templates
│       │   │   │   ├── deployment.yaml
│       │   │   │   ├── pvc.yaml
│       │   │   │   └── service.yaml
│       │   │   └── values.yaml
│       │   └── provider
│       │       ├── Chart.yaml
│       │       ├── templates
│       │       │   ├── deployment.yaml
│       │       │   ├── ingress.yaml
│       │       │   └── service.yaml
│       │       └── values.yaml
│       ├── Chart.yaml
│       ├── templates
│       │   ├── NOTES.txt
│       │   └── secrets.yaml
│       └── values.yaml
├── LICENSE
└── README.md

14 directories, 32 files
```

## Install

### Create a repository

Add a new repository with this chart:

```console
helm repo add oceanprotocol-provider https://arsys-internet.github.io/oceanprotocol-provider/
```

If everything works fine you should get something like this:

```console
"oceanprotocol-provider" has been added to your repositories
```

### Personalize the deployment

Create a values file adapted to your kubernetes cluster configuration:

```yaml
# Default values for the deployment of oceanprotocol-provider in Arsys DCD Managed Kubernetes.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

networks:
  URL: |
    { "32456": "https://rpc.dev.pontus-x.eu",
      "32457": "https://rpc.test.pontus-x.eu" }
  privateProviders: |
    { "32456": "0x0...",
      "32457": "0x0..." }
  allowedProviders: |
    [ "0x0..." ]
  privateOperator: "0x0..."
  publicOperator: "0x0..."
  authorizedDecrypters: |
    { "0x0...",
      "0x0..." }

ipfs:
  storage:
    classname: "ionos-enterprise-hdd"
  ingress:
    annotations:
      cert-manager.io/cluster-issuer: letsencrypt-prod
    host: "ipfs.ocean.arlabdevelopments.com"
    tls: true
    secretName: ipfs-tls

postgres:
  storage:
    classname: "ionos-enterprise-ssd"

operator-engine:
  image:
    repository: "rogargon/operator-engine"
    tag: "gke-gpu"
  description: "ArsysLab Data Room"
  storageClassname: "ionos-enterprise-hdd"
  priceMinute: "0"
  ipfsOutputPrefix: "https://ipfs.ocean.arlabdevelopments.com/ipfs/"
  ipfsAdminLogsPrefix: "https://ipfs.ocean.arlabdevelopments.com/ipfs/"
  podConfContainer: "rogargon/pod-configuration:timeout"

provider:
  image:
    tag: "v2.1.3"
  ipfsGateway: "https://ipfs.ocean.arlabdevelopments.com"
  providerFeeToken: |
    { "32456": "0x8a4826071983655805bf4f29828577cd6b1ac0cb",
      "32457": "0xdd0a0278f6BAF167999ccd8Aa6C11A9e2fA37F0a" }
  aquariusURL: "https://aquarius.pontus-x.eu/"
  ingress:
    annotations:
      cert-manager.io/cluster-issuer: letsencrypt-prod
    host: "provider.ocean.arlabdevelopments.com"
    tls: true
    secretName: provider-tls
```

### Deploy the provider

Once the values are adjusted to our needs, install the provider using this command:

```console
helm upgrade --install --namespace dataspace --create-namespace --values ./oceanprotocol-arsys.yaml arsys-c2d oceanprotocol-provider/oceanprotocol-provider
```

If everything works fine you should get something like this:

```plain
NAME: arsys-c2d
LAST DEPLOYED: Tue Sep 17 14:03:53 2024
NAMESPACE: dataspace
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Thank you for installing oceanprotocol-provider.

The published services for arsys-c2d are:

* IPFS: https://ipfs.ocean.arlabdevelopments.com/
* Provider: https://provider.ocean.arlabdevelopments.com/

If it is a new install, and not an upgrade, wait until everything is `Running`
to initialise the PostgreSQL database with the following command:

$ kubectl run --namespace dataspace --attach --rm --restart=Never \
  --image curlimages/curl pgsqlinit -- \
  curl -X POST -H "accept: application/json" \
    -H "Admin: myAdminSecret" \
    "http://arsys-c2d-operator-service.dataspace:8050/api/v1/operator/pgsqlinit"

Once DB initialization it's done, then you can list the computational 
environments available using this URL:
  https://provider.ocean.arlabdevelopments.com/api/services/computeEnvironments
```

Wait until all the pods get into running state.

```console
$ kubectl --namespace dataspace get pods --watch
NAME                                          READY   STATUS    RESTARTS   AGE
arsys-c2d-ipfs-68f59bbbd7-qxdqg               0/1     Pending   0          9s
arsys-c2d-operator-engine-694d59dcd5-5r698    1/1     Running   0          9s
arsys-c2d-operator-service-56999659cb-tz5nk   1/1     Running   0          9s
arsys-c2d-postgres-7f6f9ddc4b-8nk4g           0/1     Pending   0          9s
arsys-c2d-provider-fd9b989f4-7ftqf            1/1     Running   0          9s
arsys-c2d-ipfs-68f59bbbd7-qxdqg               0/1     Pending   0          23s
arsys-c2d-postgres-7f6f9ddc4b-8nk4g           0/1     Pending   0          23s
arsys-c2d-ipfs-68f59bbbd7-qxdqg               0/1     ContainerCreating   0          23s
arsys-c2d-postgres-7f6f9ddc4b-8nk4g           0/1     ContainerCreating   0          23s
arsys-c2d-ipfs-68f59bbbd7-qxdqg               0/1     ContainerCreating   0          44s
arsys-c2d-ipfs-68f59bbbd7-qxdqg               1/1     Running             0          45s
arsys-c2d-postgres-7f6f9ddc4b-8nk4g           0/1     ContainerCreating   0          84s
arsys-c2d-postgres-7f6f9ddc4b-8nk4g           1/1     Running             0          85s
^C
```

If it's a first install then ensure to launch de database initialization command shown on the notes of helm.

```console
$ kubectl run --namespace dataspace --attach --rm --restart=Never \
  --image curlimages/curl pgsqlinit -- \
  curl -X POST -H "accept: application/json" \
    -H "Admin: myAdminSecret" \
    "http://arsys-c2d-operator-service.dataspace:8050/api/v1/operator/pgsqlinit"
If you don't see a command prompt, try pressing enter.
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
pod "pgsqlinit" deleted
```

## Services

For a normal working provider we have the following resources created on kubernetes:

```
$ $ kubectl get --namespace dataspace --show-labels all,jobs,persistentvolumeclaims,persistentvolumes,ingresses,secrets,configmaps,certificates
NAME                                              READY   STATUS    RESTARTS   AGE   LABELS
pod/arsys-c2d-ipfs-68f59bbbd7-qxdqg               1/1     Running   0          10m   app=arsys-c2d-ipfs,pod-template-hash=68f59bbbd7
pod/arsys-c2d-operator-engine-694d59dcd5-5r698    1/1     Running   0          10m   app=arsys-c2d-operator-engine,pod-template-hash=694d59dcd5
pod/arsys-c2d-operator-service-56999659cb-tz5nk   1/1     Running   0          10m   app=arsys-c2d-operator-service,pod-template-hash=56999659cb
pod/arsys-c2d-postgres-7f6f9ddc4b-8nk4g           1/1     Running   0          10m   app=arsys-c2d-postgres,pod-template-hash=7f6f9ddc4b
pod/arsys-c2d-provider-fd9b989f4-7ftqf            1/1     Running   0          10m   app=arsys-c2d-provider,pod-template-hash=fd9b989f4

NAME                                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE   LABELS
service/arsys-c2d-ipfs               ClusterIP   10.233.24.70    <none>        5001/TCP,8080/TCP   10m   app.kubernetes.io/managed-by=Helm
service/arsys-c2d-operator-service   ClusterIP   10.233.47.146   <none>        8050/TCP            10m   app.kubernetes.io/managed-by=Helm
service/arsys-c2d-postgres           ClusterIP   10.233.34.17    <none>        5432/TCP            10m   app.kubernetes.io/managed-by=Helm,app=arsys-c2d-postgres
service/arsys-c2d-provider           ClusterIP   10.233.40.18    <none>        8030/TCP            10m   app.kubernetes.io/managed-by=Helm

NAME                                         READY   UP-TO-DATE   AVAILABLE   AGE   LABELS
deployment.apps/arsys-c2d-ipfs               1/1     1            1           10m   app.kubernetes.io/managed-by=Helm,app=arsys-c2d-ipfs
deployment.apps/arsys-c2d-operator-engine    1/1     1            1           10m   app.kubernetes.io/managed-by=Helm,app=arsys-c2d-operator-engine
deployment.apps/arsys-c2d-operator-service   1/1     1            1           10m   app.kubernetes.io/managed-by=Helm,app=arsys-c2d-operator-service
deployment.apps/arsys-c2d-postgres           1/1     1            1           10m   app.kubernetes.io/managed-by=Helm
deployment.apps/arsys-c2d-provider           1/1     1            1           10m   app.kubernetes.io/managed-by=Helm,app=arsys-c2d-provider

NAME                                                    DESIRED   CURRENT   READY   AGE   LABELS
replicaset.apps/arsys-c2d-ipfs-68f59bbbd7               1         1         1       10m   app=arsys-c2d-ipfs,pod-template-hash=68f59bbbd7
replicaset.apps/arsys-c2d-operator-engine-694d59dcd5    1         1         1       10m   app=arsys-c2d-operator-engine,pod-template-hash=694d59dcd5
replicaset.apps/arsys-c2d-operator-service-56999659cb   1         1         1       10m   app=arsys-c2d-operator-service,pod-template-hash=56999659cb
replicaset.apps/arsys-c2d-postgres-7f6f9ddc4b           1         1         1       10m   app=arsys-c2d-postgres,pod-template-hash=7f6f9ddc4b
replicaset.apps/arsys-c2d-provider-fd9b989f4            1         1         1       10m   app=arsys-c2d-provider,pod-template-hash=fd9b989f4

NAME                                       STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS           VOLUMEATTRIBUTESCLASS   AGE   LABELS
persistentvolumeclaim/arsys-c2d-ipfs       Bound    pvc-5c2c4013-0a8c-4f6f-b7a6-f100254ec2d8   1Gi        RWO            ionos-enterprise-hdd   <unset>                 10m   app.kubernetes.io/managed-by=Helm
persistentvolumeclaim/arsys-c2d-postgres   Bound    pvc-c6ad64b3-63fa-4f80-a1b5-fbecb8eb96ea   1Gi        RWO            ionos-enterprise-ssd   <unset>                 10m   app.kubernetes.io/managed-by=Helm

NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                                                                                                       STORAGECLASS           VOLUMEATTRIBUTESCLASS   REASON   AGE     LABELS
persistentvolume/pvc-5c2c4013-0a8c-4f6f-b7a6-f100254ec2d8   1Gi        RWO            Delete           Bound    dataspace/arsys-c2d-ipfs                                                                                                    ionos-enterprise-hdd   <unset>                          9m44s   <none>
persistentvolume/pvc-c6ad64b3-63fa-4f80-a1b5-fbecb8eb96ea   1Gi        RWO            Delete           Bound    dataspace/arsys-c2d-postgres                                                                                                ionos-enterprise-ssd   <unset>                          9m44s   <none>

NAME                                           CLASS   HOSTS                                  ADDRESS        PORTS     AGE   LABELS
ingress.networking.k8s.io/arsys-c2d-ipfs       nginx   ipfs.ocean.arlabdevelopments.com       5.250.184.10   80, 443   10m   app.kubernetes.io/managed-by=Helm
ingress.networking.k8s.io/arsys-c2d-provider   nginx   provider.ocean.arlabdevelopments.com   5.250.184.10   80, 443   10m   app.kubernetes.io/managed-by=Helm

NAME                                     TYPE                 DATA   AGE     LABELS
secret/arsys-c2d-networks                Opaque               7      10m     app.kubernetes.io/managed-by=Helm
secret/arsys-c2d-postgres                Opaque               6      10m     app.kubernetes.io/managed-by=Helm,app=arsys-c2d-postgres
secret/ipfs-tls                          kubernetes.io/tls    2      3h11m   controller.cert-manager.io/fao=true
secret/provider-tls                      kubernetes.io/tls    2      3h11m   controller.cert-manager.io/fao=true
secret/sh.helm.release.v1.arsys-c2d.v1   helm.sh/release.v1   1      10m     modifiedAt=1726574635,name=arsys-c2d,owner=helm,status=deployed,version=1

NAME                         DATA   AGE     LABELS
configmap/kube-root-ca.crt   1      3h11m   <none>

NAME                                       READY   SECRET         AGE   LABELS
certificate.cert-manager.io/ipfs-tls       True    ipfs-tls       10m   app.kubernetes.io/managed-by=Helm
certificate.cert-manager.io/provider-tls   True    provider-tls   10m   app.kubernetes.io/managed-by=Helm
```
