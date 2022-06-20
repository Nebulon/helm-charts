Helm chart for [Nebulon](https://nebulon.com/) CSI Driver


# Prerequisites

## NebCloud Credentials
A secret named **nebulon-credentials** must be created. It must exist in the same namespace as Nebulon CSI driver. Use the following script to create it. 

`This script creates secret in the default namespace. If you want it in a different namespace then replace the string 'default' with the name of your namespace`

```
YOUR_UCAPI_USERNAME=name
YOUR_UCAPI_PASSWORD=password
NAMESPACE=default

echo 'apiVersion: v1
kind: Secret
metadata:
  name: nebulon-credentials
type: Opaque
stringData:
    api_endpoint: "https://ucapi.nebcloud.nebuloninc.com"
    username: '$YOUR_UCAPI_USERNAME'
    password: '$YOUR_UCAPI_PASSWORD | kubectl apply -n $NAMESPACE -f -
```

## Snapshot CRDs and RBACs

Certain CRDs and RBACs are required for supporting snapshots. Nebulon CSI driver chart assumes that these are already installed in the cluster by Kubernetes vendor. If that is not the case please run the following commands to install them:
  
### For version
```
kubectl create -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/master/client/config/crd/snapshot.storage.k8s.io_volumesnapshotclasses.yaml
kubectl create -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/master/client/config/crd/snapshot.storage.k8s.io_volumesnapshotcontents.yaml
kubectl create -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/master/client/config/crd/snapshot.storage.k8s.io_volumesnapshots.yaml
kubectl create -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/master/deploy/kubernetes/snapshot-controller/rbac-snapshot-controller.yaml
kubectl create -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/master/deploy/kubernetes/snapshot-controller/setup-snapshot-controller.yaml
```

# How to install Nebulon CSI driver using GitHub as Helm repository
`Notes:`

  `Snapshots are not enabled by default. If your environment meets the prerequisites then pass '--set snapshotclass.enabled=true' to helm to enable it.`

  `Example below installs chart in namespance 'nebulon'. To install in default namespace try the command without '-n nebulon'.`
## Create a repo named 'nebulon' in helm:
  
  ```
  $ helm repo add nebulon https://nebulon.github.io/helm-charts
  ```

## List charts from nebulon repo just created:
  
  ```
  $ helm search repo nebulon
  ```

  You will see the latest chart version:

  ```
  NAME                     	CHART VERSION	APP VERSION	DESCRIPTION
  ```

  ```
  nebulon/csi-nebulon      	0.1.11        	0.1.11      	Nebulon CSI Driver
  ```

## Install chart in a namespace named 'nebulon':

  ```
  helm install nebulon/csi-nebulon --create-namespace -n nebulon --generate-name
  ```

  Or install chart with snapshot support:
  
  ```
  helm install nebulon/csi-nebulon --create-namespace -n nebulon --set snapshotclass.enabled=true --generate-name
  ```

# How to install using local files (Only for testing)
Clone <a url="https://github.com/Nebulon/helm-charts"> this</a> repository and change directory to topmost folder.

## To install in namespace 'nebulon':
```
helm install -g --create-namespace -n nebulon .
```

## Install with snapshot support:
```
helm install -g --set snapshotclass.enabled=true --create-namespace -n nebulon .
```