# KCD Geneva

## Setup

```sh
git clone https://github.com/tim-koko/kubevirt-demo.git
oc new-project kcd-demo
export USER=kcd-demo
```

1. Create the artefacts before the presentation and only start and observe them

```sh
kubectl apply -f kubevirt-demo/kcd/fedora-vm/cloudinit-userdata-secret.yaml --namespace=kcd-demo
kubectl apply -f kubevirt-demo/kcd/fedora-vm/svc-ingress.yaml --namespace=kcd-demo
kubectl apply -f kubevirt-demo/kcd/fedora-vm/vm.yaml --namespace=kcd-demo
```

## Demo

* go through the vm artefact
* start the vm
* go through the secret artefact
* show the custom config map

```sh
virtctl start fedora-vm --namespace=kcd-demo
```

* show the starting vm

```sh
kubectl get vm --namespace=kcd-demo
```

```sh
kubectl get vmi --namespace=kcd-demo
```

* And the pod

```sh
kubectl get pod --namespace=kcd-demo
```

* connect to the console and log in

```sh
virtctl console fedora-vm --namespace=kcd-demo
```

* Login: fedora, kubevirt
* show the running nginx service
* execute curl to verify the running nginx
* explore the service and ingress

## live migration

Terminal 1

```sh
while true; do sleep 1; echo -n `date +"[%H:%M:%S,%3N] "`; echo -n " "; curl --max-time 1 --connect-timeout 0.8 https://kcd.apps.lab.clusters.t-k.ch/api; echo ""; done
```

Terminal 2

```sh
virtctl migrate fedora-vm --namespace=kcd-demo
kubectl get vmi -w --namespace=kcd-demo
```

## Gitops with argo CD

* Open Argo CD showing the vmirs
* Change the amount of replicas from 1 to 3
* commit and push the changes
* show how simple it is to manage VMs using gitops

## Backup

. Create backup ns where everything is running already

```sh
kubectl create ns kcd-demo-backup
kubectl apply -f kubevirt-demo/kcd/backup/cloudinit-userdata-secret.yaml --namespace=kcd-demo-backup
kubectl apply -f kubevirt-demo/kcd/backup/svc-ingress.yaml --namespace=kcd-demo-backup
kubectl apply -f kubevirt-demo/kcd/backup/vm.yaml --namespace=kcd-demo-backup
kubectl apply -f kubevirt-demo/kcd/backup/vm-replicaset.yaml --namespace=kcd-demo-backup
```

```sh Backup
virtctl migrate fedora-vm-2 --namespace=kubevirt-demo-backup
kubectl get vmi -w --namespace=kubevirt-demo-backup
```

```sh Backup
while true; do sleep 1; echo -n `date +"[%H:%M:%S,%3N] "`; echo -n " "; curl --max-time 1 --connect-timeout 0.8 https://kcd-rs.apps.lab.clusters.t-k.ch/api; echo ""; done
```

```sh Backup
while true; do sleep 1; echo -n `date +"[%H:%M:%S,%3N] "`; echo -n " "; curl --max-time 1 --connect-timeout 0.8 https://kcd-backup.apps.lab.clusters.t-k.ch/; echo ""; done
```

## reset

```sh
virtctl stop fedora-vm --namespace=kcd-demo
kubectl apply -f kcd/fedora-vm/cloudinit-userdata-secret.yaml --namespace=kcd-demo
kubectl apply -f kcd/fedora-vm/svc-ingress.yaml --namespace=kcd-demo
kubectl apply -f kcd/fedora-vm/vm.yaml --namespace=kcd-demo
```
