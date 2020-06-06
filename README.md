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
2. Move istioctl to /usr/local/bin
```
sudo mv /opt/istio-1.6.1/bin/istioctl /usr/local/bin/
```
3. Install the demo profile

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
kubectl apply -f /opt/istio-1.6.1/samples/bookinfo/networking/bookinfo-gateway.yaml
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
kubectl apply -f /opt/istio-1.6.1/samples/bookinfo/networking/destination-rule-all.yaml
```
13. Verify if the destination rules are set correctly
```
kubectl get destinationrules -o yaml
```

APPLICATION IS DEPLOYED.

### Writing microservices tutorial - https://istio.io/docs/examples/microservices-istio/
1. Create a namespace called tutorial
```
kubectl create namespace tutorial
```
2. Enable request access logging. Steps 
3. Deploy an app which will make requests
```
kubectl apply -f /opt/istio-1.6.1/samples/sleep/sleep.yaml
```
4. Export the name of the sleep pod in to an environment variable
```
export SOURCE_POD=$(kubectl get pod -l app=sleep -o jsonpath={.items..metadata.name})
```
5. Add and start a httpbin sample. Dont know why.
```
kubectl apply -f /opt/istio-1.6.1/samples/httpbin/httpbin.yaml
```
6. Send logs to stdout
```
istioctl install --set profile=demo --set meshConfig.accessLogFile="/dev/stdout"
```
7. Check the logs. This works because of the step 6 above and step 2 under the "application setup given under istio" section where auto injection was enabled - ```kubectl label namespace default istio-injection=enabled```
```
kubectl exec -it $(kubectl get pod -l app=sleep -o jsonpath='{.items[0].metadata.name}') -c sleep -- curl -v httpbin:8000/status/418
kubectl logs -l app=sleep -c istio-proxy
kubectl logs -l app=httpbin -c istio-proxy
```
8. Follow https://istio.io/docs/examples/microservices-istio/single/. Basically simulating to run an already developed microservice locally. 

9. Package a service - https://istio.io/docs/examples/microservices-istio/package-service/. In this we build the docker container and test it locally. Use --network=host with docker build. Otherwise does not work. Will have to check later. Bridge network is not working. Is this a problem with 19.03.X or something else? Lot of forum issues on the same. Will check later.

10. DOES NOT WORK COMPLETELY - Run the application with k8s - https://istio.io/docs/examples/microservices-istio/bookinfo-kubernetes/. You have to enable ingress add on for minikube using the below command otherwise IP will not be assigned. See for more information - https://stackoverflow.com/questions/51511547/empty-address-kubernetes-ingress. One more thing. You dont get IP immediately. Getting IP will take time. Probably a few seconds.
```
minikube addons enable ingress
```
11. Testing microservices - https://istio.io/docs/examples/microservices-istio/production-testing/
12. Add new versions of existing microservices - https://istio.io/docs/examples/microservices-istio/add-new-microservice-version/
13. DOES NOT WORK COMPLETELY - Steps 8 to 12 were pure docker and kubernetes. Now we move to istios. Enable Istio on productpage - https://istio.io/docs/examples/microservices-istio/add-istio/
14. Issues: static resources dont show up. grafana dashboard does not come up.
15. Adding istio to all services - https://istio.io/docs/examples/microservices-istio/istio-ingress-gateway/ - static works after virtualservice is added. Need to understand all this in detail. It should have worked with the k8s ingress itself. See https://github.com/istio/istio/issues/13244
16. MOnitoring - https://istio.io/docs/examples/microservices-istio/logs-istio/

On the whole the documentation does not seem to reflect what should have happened. I am going to rather follow this tutorial. Makes more sense I guess. https://www.youtube.com/watch?v=WFu8OLXUETY&list=PL34sAs7_26wPkw9g-5NQPP_rHVzApGpKP&index=2&t=301s

