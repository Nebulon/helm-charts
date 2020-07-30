# Kubernetes Wordpress Demo App w/Persistent Volumes

## Prerequisites
* A load balancer is required for the Wordpress site frontend in bare metal environments without an external load balancer. Instructions for installing MetalLB can be found here: https://metallb.universe.tf/
* * The Nebulon CSI driver must be installed for the persistent volumes to work. Installation instructions are available in the Nebulon Kubernetes Infrastructure Guide available on www.nebulon.com

## What gets built
This project will deploy two pods:

* wordpress -> a frontend pod with a 25GB PV for configuration data
* mysql -> backend database for wordpress content

## Deployment
First clone the repository to a machine that has kubectl installed and is configured to talk to a K8s cluster. 
Once you have the repository downloaded you can deploy the stateful set by running this from the root folder of the project: 

```
    $ kubectl apply -k ./
    storageclass.storage.k8s.io/nebulon unchanged
    namespace/wordpress created
    secret/mysql-pass-m68dhhmggh created
    service/wordpress-mysql created
    service/wordpress created
    deployment.apps/wordpress-mysql created
    deployment.apps/wordpress created
    persistentvolumeclaim/mysql-pv-claim created
    persistentvolumeclaim/wp-pv-claim created
```

The deployed pods:
```
    $ kubectl get pods -n wordpress
    NAME                               READY   STATUS    RESTARTS   AGE
    wordpress-774c7c6d68-kgdzj         1/1     Running   1          2m53s
    wordpress-mysql-596dc96584-64m5g   1/1     Running   0          2m53s
```

To get the IP address for the wordpress pod:
```
    $ kubectl get svc wordpress -n wordpress
    NAME        TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)        AGE
    wordpress   LoadBalancer   10.98.29.107   10.100.5.207   80:32713/TCP   3m19s
```


## Some Tips
Set the namespace for kubectl to "wordpress" so you don't have to append "-n wordpress" to every command. You can do this by running:
`kubectl config set-context $(kubectl config current-context) --namespace=wordpress`

To flip back to the default namespace:
`kubectl config set-context $(kubectl config current-context) --namespace=default`

Consider adding some helpful alias commands to your ~/.bashrc file to help with navigation. Once edits have been made run ". ~/.bashrc" to reload.

```
alias kgp="kubectl get pods -o wide"    
alias kgpv="kubectl get pv"
alias kgpvc="kubectl get pvc"
#kmysql alias -> executes 'df -h /var/lib/mysql' on the mysql pod to show details of the pod mount point and its connection its persistent volume
alias kmysql='kubectl exec $(kubectl get pod -l "tier=mysql" -n wordpress -o jsonpath='{.items[0].metadata.name}') -- df -h /var/lib/mysql'
#kwordpress alias -> executes 'df -h /var/www/html' on the wordpress pod to show details of the pod mount point and its connection its persistent volume
alias kwordpress='kubectl exec $(kubectl get pod -l "tier=frontend" -n wordpress -o jsonpath='{.items[0].metadata.name}') -- df -h /var/www/html'
#kgva alias -> lists volume attachments for the pods. 
alias kgva="kubectl get volumeattachment"   
alias kwgp="watch -n 5 kubectl get pods"    
alias nswordpress='kubectl config set-context $(kubectl config current-context) --namespace=wordpress'
alias nsdefault='kubectl config set-context $(kubectl config current-context) --namespace=default'
```

# Attribution
This sample application is a lightly modified version of an official K8s demo available here: https://kubernetes.io/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/