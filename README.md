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

```
oceanprotocol-provider
├── charts
│   ├── ipfs
│   │   ├── Chart.yaml
│   │   ├── templates
│   │   │   ├── deployment.yaml
│   │   │   ├── ingress.yaml
│   │   │   ├── pvc.yaml
│   │   │   └── service.yaml
│   │   └── values.yaml
│   ├── operator-api
│   │   ├── Chart.yaml
│   │   ├── templates
│   │   │   ├── deployment.yaml
│   │   │   └── service.yaml
│   │   └── values.yaml
│   ├── operator-engine
│   │   ├── Chart.yaml
│   │   ├── templates
│   │   │   ├── deployment.yaml
│   │   │   ├── role-binding.yaml
│   │   │   ├── role.yaml
│   │   │   └── service-account.yaml
│   │   └── values.yaml
│   ├── postgres
│   │   ├── Chart.yaml
│   │   ├── templates
│   │   │   ├── deployment.yaml
│   │   │   ├── pvc.yaml
│   │   │   └── service.yaml
│   │   └── values.yaml
│   └── provider
│       ├── Chart.yaml
│       ├── templates
│       │   ├── deployment.yaml
│       │   ├── ingress.yaml
│       │   └── service.yaml
│       └── values.yaml
├── Chart.yaml
├── README.md
├── templates
│   ├── NOTES.txt
│   └── secrets.yaml
└── values.yaml

12 directories, 31 files
```

## Install

Create a values file adapted to your kubernetes cluster configuration:

```
# Default values for the deployment of oceanprotocol-provider in Arsys DCD Managed Kubernetes.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

keys:
  privateProviders: |
    { "100": "0x0..." }
  publicProviders: |
    [ "0x0..." ]
  privateOperator: "0x0..."
  publicOperator: "0x0..."

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

provider:
  image:
    tag: "v2.1.3"
  ipfsGateway: "https://ipfs.ocean.arlabdevelopments.com"
  ingress:
    annotations:
      cert-manager.io/cluster-issuer: letsencrypt-prod
    host: "provider.ocean.arlabdevelopments.com"
    tls: true
    secretName: provider-tls

operator-engine:
  image:
    repository: "rogargon/operator-engine"
    tag: "gke-gpu"
  description: "ArsysLab Data Room"
  storageClassname: "ionos-enterprise-hdd"
  confContainer: "rogargon/pod-configuration:timeout"
  priceMinute: "0"
  ipfsOutputPrefix: ""
  ipfsAdminLogsPrefix: ""
  operatorPrivateKey: ""

```

And after creating the file execute this command:

```
druiz@LLL-178708:~/git$ helm upgrade --install --namespace dataspace --create-namespace --values ./oceanprotocol-arsys.yaml arsys-c2d oceanprotocol-provider
Release "arsys-c2d" does not exist. Installing it now.
NAME: arsys-c2d
LAST DEPLOYED: Fri Jul  5 13:07:23 2024
NAMESPACE: dataspace
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Thank you for installing oceanprotocol-provider.

The published services for arsys-c2d are:

* IPFS: https://ipfs.ocean.arlabdevelopments.com/
* Provider: https://provider.ocean.arlabdevelopments.com/

Now wait until everything is `Running` to initialise the PostgreSQL database with the following command:

    $ kubectl run --namespace dataspace --attach --rm --restart=Never \
        --image curlimages/curl pgsqlinit -- \
        curl -X POST -H "accept: application/json" \
            -H "Admin: postgresadmin" \
            "http://arsys-c2d-operator-api.dataspace:8050/api/v1/operator/pgsqlinit" 

And then you can list the computational environments available using this URL: https://provider.ocean.arlabdevelopments.com/api/services/computeEnvironments
```
