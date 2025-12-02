# KCD Geneva

## Setup

```sh
git clone https://github.com/tim-koko/kubevirt-demo.git
oc new-project kcd-demo
export USER=kcd-demo
```

1. Create the artefacts before the presentation and only start and observe them

```sh
kubectl create secret generic fedora-vm --from-file=userdata=kubevirt-demo/kcd/cloudinit-userdata.yaml --namespace=$USER
kubectl apply -f kubevirt-demo/kcd/svc-ingress.yaml --namespace=$USER
kubectl apply -f kubevirt-demo/kcd/vm.yaml --namespace=$USER
kubectl apply -f kubevirt-demo/kcd/vm-replicaset.yaml --namespace=$USER
```

1. Create backup ns where everything is running already

```sh
kubectl create ns kubevirt-demo-backup
kubectl create secret generic fedora-vm-2 --from-file=userdata=kubevirt-demo/cloud-native-meetup/backup/cloudinit-userdata.yaml --namespace=kubevirt-demo-backup
kubectl apply -f kubevirt-demo/kcd/backup/svc-ingress.yaml --namespace=kubevirt-demo-backup
kubectl apply -f kubevirt-demo/kcd/backup/vm.yaml --namespace=kubevirt-demo-backup
```

## Demo

* go through the secret artefact
* go through the vm artefact
* start the vm

```sh
virtctl start fedora-vm --namespace=$USER
```

* show the starting vm

```sh
kubectl get vm --namespace=$USER
```

```sh
kubectl get vmi --namespace=$USER
```

* And the pod

```sh
kubectl get pod --namespace=$USER
```

* connect to the console and log in

```sh
virtctl console fedora-vm --namespace=$USER
```

* Login: fedora, kubevirt
* show the running nginx service
* execute curl to verify the running nginx
* explore the service and ingress

## live migration

Terminal 1

```sh
while true; do sleep 1; echo -n `date +"[%H:%M:%S,%3N] "`; echo -n " "; curl --max-time 1 --connect-timeout 0.8 https://kcd.apps.lab.clusters.t-k.ch/; echo ""; done
```

```sh Backup
while true; do sleep 1; echo -n `date +"[%H:%M:%S,%3N] "`; echo -n " "; curl --max-time 1 --connect-timeout 0.8 https://kcd-rs.apps.lab.clusters.t-k.ch/; echo ""; done
```

```sh Backup
while true; do sleep 1; echo -n `date +"[%H:%M:%S,%3N] "`; echo -n " "; curl --max-time 1 --connect-timeout 0.8 https://kcd-backup.apps.lab.clusters.t-k.ch/; echo ""; done
```

Terminal 2

```sh
virtctl migrate vm --namespace=$USER
kubectl get vmi -w --namespace=$USER
```

```sh Backup
virtctl migrate vm --namespace=kubevirt-demo-backup
kubectl get vmi -w --namespace=kubevirt-demo-backup
```

## reset

```sh

kubectl delete secret fedora-vm  --namespace=$USER
kubectl delete -f kubevirt-demo/cloud-native-meetup/vm.yaml --namespace=$USER
kubectl delete -f kubevirt-demo/cloud-native-meetup/svc-ingress.yaml --namespace=$USER
```

```sh Backup

kubectl delete secret fedora-vm-2  --namespace=kubevirt-demo-backup
kubectl delete -f kubevirt-demo/cloud-native-meetup/backup/vm.yaml --namespace=kubevirt-demo-backup
kubectl delete -f kubevirt-demo/cloud-native-meetup/backup/svc-ingress.yaml --namespace=kubevirt-demo-backup
```
