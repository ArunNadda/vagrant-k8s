
# Vagrantfile and Scripts to Automate Kubernetes Setup using Kubeadm 

## Documentation

To learn k8s, please follow learn guide at [akn/](https://github.com/ArunNadda/vagrant-k8s/tree/main/akn)

## Prerequisites

1. Working Vagrant setup
2. 8 Gig + RAM workstation as the Vms use 3 vCPUS and 4+ GB RAM
 
## Usage/Examples

To provision the cluster, execute the following commands.

```shell
git clone https://github.com/ArunNadda/vagrant-k8s.git
cd vagrant-k8s
vagrant up
```

## Set Kubeconfig file varaible.

```shell
cd vagrant-kubeadm-kubernetes
cd configs
export KUBECONFIG=$(PWD)/config
```

or you can copy the config file to .kube directory.

```shell
cp config ~/.kube/
```


## Kubernetes login token

Vagrant up will create the admin user token and saves in the configs directory.

```shell
cd vagrant-kubeadm-kubernetes
cd configs
cat token
```

## To shutdown the cluster, 

```shell
vagrant halt
```

## To restart the cluster,

```shell
vagrant up
```

## To destroy the cluster, 

```shell
vagrant destroy -f
```

