# Istio Playground project 
A playground project for istio

Steps 2,3,6,7 I dont know how many of them are relavent. Follow from steps 8 again. Istio does not seem to work with driver none and docker. Lets try now with driver virtualbox. ok! Istio does not work with minikube in baremetal or docker. Only works with virtual box. Have not tried kvm2. (see point 11)

Steps 12-16 are just verification and "play around" steps or rather info steps. No actions.

A good tutorial is https://www.youtube.com/watch?v=WFu8OLXUETY&list=PL34sAs7_26wPkw9g-5NQPP_rHVzApGpKP. Check that out for more info.

### Important Lessons:
1. Register docker as a service
2. Dont run minikube on baremetal or docker. Run it in virtualbox. kvm2 yet to be tried
3. Dont install anything as root. It stops normal user from doing anything. For example godav did not have access to ~/.kube. It sucks!
4. Please learn more about how these orchestration tools work. SHOULD NOT blindly follow steps given in the websites. Blindly following steps given in websites or tutorials wont cut it anymore in the world of Devops. If not learnt completly, forget about causing disaster on production, you wont even get far with application development.
5. Any command in /usr/bin requires sudo to run. Anything in /usr/local/bin does not require sudo and will not run with sudo.

### Installation Steps:
1. First read the complete overview. Did not understand a lot of what is in the security section. Thats ok. Will see that later.
2. Take ownership of the config directory (the user who is running things should have the ownership)
```
sudo chown $(id -u):$(id -g) $HOME/.kube
```
3. Do the below so that sudo can find it. Why should sudo find it? Because driver=none requires sudo
```
sudo mv /usr/local/bin/minikube /usr/bin/
```
4. (IMPORTANT) Require the following to run kubernates as root
```
sudo yum install conntrack
```
5. (IMPORTANT) Register docker as a service 
```
systemctl enable docker.service
```
6. DID NOT MAKE ANY CHANGE (IGNORE) (whatever that means)
```
swapoff -a 
```
7. DID NOT WORK (IGNORE)- Now start minikube properly. With driver none cpus and memory are not going to be used. But adding here anyway if we want to start with driver=kvm2 or virtualbox. Which I am not using.
```
sudo minikube start --memory=16384 --cpus=4 --kubernetes-version=v1.18.3 --driver=none
```
8. Install virtualbox on centos. linuxtechi.com/install-virtualbox-6-centos-8-rhel-8/
```
sudo dnf config-manager --add-repo=https://download.virtualbox.org/virtualbox/rpm/el/virtualbox.repo
sudo rpm --import https://www.virtualbox.org/download/oracle_vbox.asc
sudo dnf install binutils kernel-devel kernel-headers libgomp make patch gcc glibc-headers glibc-devel dkms -y
dnf search virtualbox
sudo dnf install VirtualBox-6.1.x86_64
sudo usermod -aG vboxusers godav
virtualbox
```
9. Download istio using https://istio.io/docs/setup/getting-started/#download
10. Start minikube with virtual box
```
export PATH=/opt/istio-1.6.1/bin:$PATH
sudo minikube start --memory=16384 --cpus=4 --kubernetes-version=v1.18.3 --driver=virtualbox
```
11. Install istio now! Wow! It works. Trick was starting it with VirtualBox.
```
istioctl install --set profile=demo
```
12. Just as a note. Istio does not work with minikube in baremetal or docker. Only works with virtual box. Have not tried kvm2
13. Display the list of available profiles to check installation.
```
istioctl profile list
```
14. Verify if all pods and deployments for istio are available in its namespace
```
kubectl get pods -n istio-system
kubectl get deployments -n istio-system
```
15. See the difference between normal and demo profiles.
```
istioctl profile diff default demo
```
16. Do some more fancy steps given in https://istio.io/docs/setup/install/istioctl/ to check more things out. Not doing it now. Its 1.15 in the morning. Zzzzzzz..... I THINK WE ARE DONE AT THIS POINT

### Deleting everything and reinstalling again to see if they work properly.
1. Uninstall istio
```
minikube delete
```
2. Uninstall minikube
```
rm -r ~/.kube ~/.minikube/
sudo rm /usr/bin/minikube
sudo rm -rf /etc/kubernetes/
sudo systemctl stop '*kubelet*.mount'
sudo docker system prune -af --volumes
sudo service docker stop
```
3. Remove Kubectl
```
sudo rm /usr/local/bin/kubectl
```
4. No idea why to remove docker its asking me to install perl. But i still did that with Yum. BTW even after installing perl you will get the error
```
Modular dependency problems:

 Problem 1: conflicting requests
  - nothing provides module(perl:5.26) needed by module perl-DBD-SQLite:1.58:8010020191114033549:073fa5fe-0.x86_64
 Problem 2: conflicting requests
  - nothing provides module(perl:5.26) needed by module perl-DBI:1.641:8010020191113222731:16b3ab4d-0.x86_64
```
Do the following
```
sudo yum module enable perl:5.26
```
Then the error goes away
5. Uninstall docker. The command in the website wont cut it. The below does not work
```
sudo yum remove docker
```
Do this
```
sudo yum list installed | grep 'docker'
sudo yum remove docker-ce-cli.x86_64
sudo yum remove containerd.io.x86_64
sudo yum remove docker-ce-cli.x86_64
```

# NOW WE ARE SET. TIME FOR FRESH INSTALL
