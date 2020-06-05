# Istio Playground project
A playground project for istio

Here we will try all the steps for docker, kubectl, minikube and istio all over again to make sure we get it right. Not including virualbox installation here. Does not matter.

### Install docker
Follow steps as is on the website. https://docs.docker.com/engine/install/centos/

### Install kubectl
Follow steps as is on the website. https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-linux. Make sure you install it using package manager. Its below the normal installation steps. Word of caution. Dont install it as root.

### Install minikube
1. Before installing minikube make sure virtualbox is installed. linuxtechi.com/install-virtualbox-6-centos-8-rhel-8/
```
sudo dnf config-manager --add-repo=https://download.virtualbox.org/virtualbox/rpm/el/virtualbox.repo
sudo rpm --import https://www.virtualbox.org/download/oracle_vbox.asc
sudo dnf install binutils kernel-devel kernel-headers libgomp make patch gcc glibc-headers glibc-devel dkms -y
dnf search virtualbox
sudo dnf install VirtualBox-6.1.x86_64
sudo usermod -aG vboxusers godav
virtualbox
```
2. Again follow instructions on https://kubernetes.io/docs/tasks/tools/install-minikube/
3. Run the below command to start minikube
```
minikube start --memory=16384 --cpus=4 --kubernetes-version=v1.18.3 --driver=virtualbox
```
4. Verify the installation with 
```
minikube status
kubectl get pods
kubectl get namespace
kubectl get pods -n kube-system
```

### Install istio
Just follow https://istio.io/docs/setup/platform-setup/minikube/ and https://istio.io/docs/setup/getting-started/. Also refer to TRIAL/README.md for all the circus I tried.
1. Download istio
2. Install the demo profile

COMPLETED ISTIO SETUP.


