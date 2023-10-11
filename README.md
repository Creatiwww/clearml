# ClearML Helm Chart Installation guide

## Requirements

* Set up a Kubernetes Cluster - for setting up Kubernetes on various platforms refer to the Kubernetes [getting started guide](http://kubernetes.io/docs/getting-started-guides/).
* Install Helm - Helm is a tool for managing Kubernetes charts. To install Helm, refer to the [Helm install guide](https://github.com/helm/helm#install) and ensure that the `helm` binary is in the `PATH` of your shell.

## ClearML Server Installation

```bash
# local env (without ingress)
$ helm install clearml ./clearml --namespace=clearml --create-namespace -f ./clearml/values.yaml
# on provider https://cloud.ru/ (without ingress)
$ helm install clearml ./clearml --namespace=clearml --create-namespace \
                                 -f ./clearml/values.yaml \
                                 -f ./clearml/values-prod-sensitive.yaml \
                                 --set elasticsearch.volumeClaimTemplate.storageClassName="csi-sbercloud-nd"
# on provider https://cloud.ru/ (WITH ingress)
# create values file with hosts:
$ touch ./clearml/values-prod-sensitive.yaml
# populate file with the following data:
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
                                 --set elasticsearch.volumeClaimTemplate.storageClassName="csi-sbercloud-nd"
```


please notice that required mongodb image is architecture sensitive, so use the correct one. File: clearml/charts/mongodb/values.yaml
```yaml
image:
  registry: docker.io
  #  repository: bitnami/mongodb   <-- amd64
  repository: arm64v8/mongo      # <-- arm
  tag: 6.0.10                    # <-- arm
  #  tag: 5.0.10-debian-11-r3      <-- amd64
```

## Clean up

```bash
$ helm uninstall clearml -n clearml
```