# Helm Deployment

### Table of content
* [Install Helm](#Install-Helm)
  * [Helm config](#Helm-config)
* [Blueprint Helm Charts](#Blueprint-Helm-Charts)
  * [App Helm Chart](#App-Helm-Chart)
  * [Postgres Helm chart](#Postgres-Helm-chart)
  * [Ingress Helm Chart](#Ingress-Helm-Chart)
* [Create helm releases](#Create-helm-releases)
  * [Postgres](#Postgres)
  * [Adminer](#Adminer)
  * [Kanban-app](#Kanban-app)
  * [Kanban-ui](#Kanban-ui)
  * [Ingress](#Ingress)
* [Maintanance](#Maintanance)
* [References](#References)



## Install Helm
Installing Helm - https://helm.sh/docs/intro/install/

### Helm config

Add repository with official stable Helm charts:

```bash
$ helm repo add stable https://kubernetes-charts.storage.googleapis.com/
"stable" has been added to your repositories
```

## Blueprint Helm Charts

Within this project there are 3 types of Helm charts, which will be used to deploy all Kubernetes objects.

### App Helm Chart

This is a basic Helm chart, necessary to deploy applications (*adminer*, *kanban-ui* & *kanban-app*). In order to run those apps in the Kubernetes cluster we need to create Deployment & Service for each one of them. This chart is taking care of that - it provides a template for doing that.

It was created with following command:
```bash
$ helm create app
Creating app
```

After cleaning the unncessary templates and adding files for Deployment & Service, the chart looks as follows:
```bash
.
├── charts
├── Chart.yaml
├── .helmignore
├── templates
│   ├── deployment.yaml
│   └── service.yaml
└── values.yaml
```

In order to deploy specific application you need to create a YAML file which will override default  values located in the `values.yaml`:

```yaml
app:
  name: app
  group: app
  replicaCount: 1
  container:
    image: add-image-here
    port: 8080
    env: 
      - key: key
        value: value
  service:
    type: ClusterIP
    port: 8080
```

### Postgres Helm chart

This chart is used to create a PostgreSQL database and includes 4 types of Kubernetes objects: Deployment, Service, PersistentVolumeClaim & ConfigMap. 

All templates and defualt `values.yaml` file is located in the `./postgres` folder:

```bash
.
├── charts
├── Chart.yaml
├── .helmignore
├── templates
│   ├── config.yaml
│   ├── deployment.yaml
│   ├── pvc.yaml
│   └── service.yaml
└── values.yaml
```

Similarly to previous example, to create all these objects you need to prepare your own YAML file, which follow the structure from `values.yaml`, like for example it's done in `kanban-postgress.yaml` file. 

### Ingress Helm Chart

And the last chart, is responsible for creating and configurating Ingress Controller. 

It creates the default backend service - marked as `dependency` in `Chart.yaml`:

```yaml
dependencies:
  - name: nginx-ingress
    version: 1.36.0
    repository: https://kubernetes-charts.storage.googleapis.com/
```

And the configuration of Ingress Controller, which is made by combining of `/templates/ingress.yaml` and `values.yaml`. If you want to change the routing you need to override values from relevant file.

In order to download that dependency use the command:

```
$ helm dependency update ./ingress/
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈Happy Helming!⎈
Saving 1 charts
Downloading nginx-ingress from repo https://kubernetes-charts.storage.googleapis.com/
Deleting outdated charts
```

## Create helm releases

Here is the list of commands that needs to be executed to deploy applications on the cluster. 

### Postgres

```bash
$ helm install -f kanban-postgres.yaml postgres ./postgres
NAME: postgres
LAST DEPLOYED: Fri Apr 10 22:42:44 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

### Adminer

```bash
$ helm install -f adminer.yaml adminer ./app
NAME: adminer
LAST DEPLOYED: Fri Apr 10 22:43:19 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

### Kanban-app

```bash
$ helm install -f kanban-app.yaml kanban-app ./app
NAME: kanban-app
LAST DEPLOYED: Fri Apr 10 22:43:45 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

### Kanban-ui

```bash
$ helm install -f kanban-ui.yaml kanban-ui ./app
NAME: kanban-ui
LAST DEPLOYED: Fri Apr 10 22:44:16 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

### Ingress

```bash
$ helm install -f ingress.yaml ingress ./ingress
NAME: ingress
LAST DEPLOYED: Fri Apr 10 22:44:42 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

## Maintanance

In order to check the list of Helm releases:

```bash
$ helm list
NAME      	NAMESPACE	REVISION	UPDATED             STATUS  	CHART         	APP VERSION
adminer   	default  	1       	2020-04-10 22:43:19	deployed	app-0.1.0     	1.16.0     
ingress   	default  	1       	2020-04-10 22:44:42	deployed	ingress-0.1.0 	1.16.0     
kanban-app	default  	1       	2020-04-10 22:43:45	deployed	app-0.1.0     	1.16.0     
kanban-ui 	default  	1       	2020-04-10 22:44:16 deployed	app-0.1.0     	1.16.0     
postgres  	default  	1       	2020-04-10 22:42:44	deployed	postgres-0.1.0	1.16.0 
```

To update the release just replace the `install` key word in a command with `upgrade`, e.g.: 

```
$ helm upgrade -f ingress.yaml ingress ./ingress
```

To delete the release use:

```bash
$ helm uninstall ingress
```


