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


### Setup the application that comes with Istio
1. Read up on https://istio.io/docs/examples/bookinfo/
2. Label the default namespace so that istio can automatically inject sidecar.
```
kubectl label namespace default istio-injection=enabled
```
3. Deploy the application
```
kubectl apply -f /opt/istio-1.6.1/samples/bookinfo/platform/kube/bookinfo.yaml
```
4. Check if everything is deployed.
```
kubectl get deployment
kubectl get services
```
5. Test if the application works. This is curl from ratings to product page
```
kubectl exec -it "$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -- curl productpage:9080/productpage | grep -o "<title>.*</title>"
```
6. Make it accessible from outside (steps 7 to 11)

7. Create the ingress gateway
```
kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
```
8. Verify it
```
kubectl get gateway
```
9. Run the following command to see if the external load balancer is available for the kubernates cluster. If yes, an external IP would be shown in the result of the below command. I have not set it up yet. So I have to use the node port.
```
kubectl get svc istio-ingressgateway -n istio-system
```
10. For building the URL for node port follow the below commands
```
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')
export TCP_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="tcp")].nodePort}')
export INGRESS_HOST=$(minikube ip)
export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
```
11. Hit the following url on browser to see if things work - http://192.168.99.102:32143/productpage. Hitting this url without /productpage will not take us anywhere since it is not allowed in the virtual service we have created. Also note that unlike docker for desktop in which one could access node port also from localhost, here it has to be the IP of the minikube.
12. Enable the destination rules where subsets are defined. These version map to the label defined for each pod in the application yaml file - /opt/istio-1.6.1/samples/bookinfo/platform/kube/bookinfo.yaml
```
kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml
```
13. Verify if the destination rules are set correctly
```
kubectl get destinationrules -o yaml
```

APPLICATION IS DEPLOYED.
