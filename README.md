# Node Js project deployment
## Creating by Mücahit ERDEM


## Features
1) 3 VM
2) Jenkins , Docker Registy , K8s multi node cluster

## Installation

We were asked to build this project using the infrastructure as a code structure. Because of this, I used [Vagrant](https://www.vagrantup.com) when creating a virtual machine. 

For create VM's...

```
Vagrant.configure("2") do |config|
  vm_name=['server1','server2','server3']
  vm_name.each do |i|
      config.vm.define "#{i}" do |node|
      node.vm.box = "generic/ubuntu2004"
    end
  end
end
```
for see all network conf.
``` vagrant ssh-config > vagrant-ssh ```

## Step 4 
for install jenkins i prefer to use jenkins inside docker container. Because we can use docker more effective.

``` 
$ sudo apt install curl
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh``` 

and running jenkins with docker : 

``` docker run -p 8080:8080 -p 50000:50000 -d -v jenkins_home:/var/jenkins_home  -v /var/run/docker.sock:/var/run/docker.sock jenkins/jenkins:lts  ```

i changed my vm network settings. Before that we have a local ip like : 127.0.0.*
but i changed to bridge network and we have a public ip for our network.
I select "normal suggested plugin install" for jenkins.
all username and password is admin in jenkins.
the url of jenkins  : http://192.168.1.18:8080/ (after changes http://192.168.1.102:8080)

## Step 5 

before installing docker registry, i create Self Signed Certificate.

```  docker run -d -p 5000:5000 --restart=always --name registry -v /etc/certs:/etc/certs -e REGISTRY_HTTP_TLS_CERTIFICATE=/etc/certs/ca.crt -e REGISTRY_HTTP_ADDR=0.0.0.0:5000 -e REGISTRY_HTTP_TLS_KEY=/etc/certs/ca.key registry ```

## Step 3 
before install the k8s to our server we should change swap settings. to do that : 
```swapoff -a
nano /etc/fstab ``` 

## for master : 
```ifconfig eth1 192.168.56.100 netmask 255.255.255.0 up
sudo hostnamectl set-hostname kubernetemaster
kubeadm init --apiserver-advertise-address=192.168.1.100 --pod-network-cidr=192.168.0.0/16```
- after than we have to following that showing screen 
``` mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config``` 

## for node : 
``` 
ifconfig eth1 192.168.56.101 netmask 255.255.255.0 up
sudo hostnamectl set-hostname kubernetenode``` 
we are running command that prompted on the screen and starting with kubeadm join 
``` sudo kubeadm join 192.168.1.100:6443 --token 2dldpa.qmajbssfwcchrunf \
    --discovery-token-ca-cert-hash sha256:a1a90f00441eeaef5532297196853747d39c30fdc8fd6934e1e0dc0bfaf7df38``` 
	
#for 3th server : 
I installed Jenkins and Docker Registry on this vm. When i close the vm ip that i changed is gone. Because of this you have to run this command after close the vm.
ifconfig eth0 192.168.1.102 netmask 255.255.255.0 up
Solution : kubeadm reset ... 
Jenkins create pipeline 
1) using node agent
2) manually install node to server.



## Problem / Solution :
1) firstly we see an error when we are trying to connect server. it said 
```[kubelet-check] It seems like the kubelet isn't running or healthy.
[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get http://localhost:10248/healthz: dial tcp 127.0.0.1:10248: connect: connection refused. ```
Solution : we see this message because of we setup vm with vagrant. Docker dont know vagrant as a user. 
https://stackoverflow.com/a/68722458
2) some ports and some files have been created 


