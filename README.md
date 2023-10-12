# Playground for ClearML MLOps tool

## ClearML Server Installation

### Requirements

* Set up a Kubernetes Cluster - for setting up Kubernetes on various platforms refer to the Kubernetes [getting started guide](http://kubernetes.io/docs/getting-started-guides/).
* Install Helm - to install, refer to the [Helm install guide](https://github.com/helm/helm#install).

### Installation

```bash
# local env (without ingress)
$ helm install clearml ./clearml --namespace=clearml --create-namespace -f ./clearml/values.yaml

# on provider https://cloud.ru/ (without ingress)
$ helm install clearml ./clearml --namespace=clearml --create-namespace \
                                 -f ./clearml/values.yaml \
                                 --set elasticsearch.volumeClaimTemplate.storageClassName="csi-sbercloud-nd"

# on provider https://cloud.ru/ (WITH ingress)
#   create values file with hosts:
$ touch ./clearml/values-prod-sensitive.yaml
#   populate file with the following data:
    apiserver:
      ingress:
        hostName: "api.<your domain>"
    fileserver:
      ingress:
        hostName: "files.<your domain>"
    webserver:
      ingress:
        hostName: "app.<your domain>"
     
$ helm install clearml ./clearml --namespace=clearml --create-namespace \
                                 -f ./clearml/values.yaml \
                                 -f ./clearml/values-prod.yaml \
                                 -f ./clearml/values-prod-sensitive.yaml \
                                 --set elasticsearch.volumeClaimTemplate.storageClassName="csi-sbercloud-nd"
```

(!) please notice that required mongodb image is architecture sensitive, so use the correct one at file: _clearml/charts/mongodb/values.yaml_
```yaml
image:
  registry: docker.io
  #  repository: bitnami/mongodb   <-- amd64
  repository: arm64v8/mongo      # <-- arm
  tag: 6.0.10                    # <-- arm
  #  tag: 5.0.10-debian-11-r3      <-- amd64
```

## Jupyter notebook to test ClearML

### Installation
```bash
# install py package
pip install clearml
# configure credentials by copy-pasting the settings given on ClearML UI (Settings->Workspace->Create new credentials)
clearml-init
```
Configure ClearML connection at _jupyter/clearml_hello_world.ipynb_ according to the settings given on ClearML UI (Settings->Workspace->Create new credentials)
```
    "# configure ClearML connection\n",
    "%env CLEARML_WEB_HOST=http://app.<host>\n",
    "%env CLEARML_API_HOST=http://api.<host>\n",
    "%env CLEARML_FILES_HOST=http://files.<host>\n",
    "%env CLEARML_API_ACCESS_KEY=<access_key>\n",
    "%env CLEARML_API_SECRET_KEY=<secret_key>"
```
Now you can execute notebook the ordinar way

(!) script _clear_notebook_outputs_and_metadata_before_push.sh_ is used for clearing metadata and outputs of the notebook, before pushing it to git

## Clean up

```bash
$ helm uninstall clearml -n clearml
```