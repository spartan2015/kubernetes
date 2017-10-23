[docs]
https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/

[commands]
minikube start

minikube start --vm-drive=xhyve --kubernetes-version="v1.6.0"
#defaults to virtualbox
minikube start --vm-drive=hyperv

minikube delete
minikube start --vm-drive=xhyve --kubernetes-version="v1.6.0" && minikube start --cpus 4 --memory 8192

minikube  version

minikube stop

minikube delete

minikube  dashboard

minikube ip

minikube ssh

kubectl.exe get nodes

kubectl.exe config current-context

#kubeadm - for manual install on all nodes (master and node)
Install:
docker (or rkt) - container runtime
kubelet (server)
kubectl (client)
CNI (Container Network Interface)
kubeadm

On Node 1 execute (ubuntu):

apt-get update
apt-get install -y apt-transport-https
curls -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update

apt-get install docker.io kubeadm kubectl kubelet kubernetes-cni

# create a single node cluster
kubeadm init

then copy paste and run commands as reg user:
sudo cp /etc/kubernetes/admin.conf $HOME/
sudo chown $(id -u):$(id -g) $HOME/admin.conf
export KUBECONFIG=$HOME/admin.conf

#not use it

kubectl get nodes

#will tell you that there is no dns pod available
kubectl get pods --all-namespaces

#now create a pod network - mandatory to get the cluster up and running
kubectl apply --filename https://git.io/weave-kube-1.6

#now add more nodes - installed deps as above
kubeadm join --token [take it from the result of previous kubeadm init]

# not you should be able to see them all
kubectl get nodes


#Working with pods:

Feed the manifest file describing the pod to the apiserver{} yml or json

kubectl create -f nigel-pod.yml
or
kubectl create -f nigel-pod.json

kubectl describes pods

kubectl get pods/hello-pod

kubectl get pods -all-namespaces

kubectl delete pods hello-pod

#Running an image
kubectl run echo --image=gcr.io/google_containers/echoserver:1.4 --port=8080
deployment "echo" created

#Service: types:
ClusterIP - stable internal cluster ip
NodePort: Exposes the app outside of the cluster by adding a cluster-wide port on top of ClusterIP
LoadBalancer: Integrates NodePort with cloud-based load balancers

kubectl expose rc hello-rc --name=hello-service --target-port=8080 --type=NodePort
or
kubectl expose deployment echo --type=NodePort

kubectl describe svc hello-service

port=kubectl get service echo --output='jsonpath="{.spec.ports[0].NodePort}"'
curl http://192.168.99.100:$(port)/hi

#also see the created endpoint object
kubectl describe ep hello-service

# go in the browser and check

kubectl delete svc hello-service

#deployment
are all about updates and rollback
Without deployments:
define new replication controller file - with a new version then run:
kubectl rolling-update -f updated-rc.yml

With a deployment descriptor:

apiVersion: 
extensions/v1beta1
kind: Deployment
metadata:
 name: hello-deploy
spec:
 replicas: 10
 
Now you can make changes to the same file to update the cluster
Underlying it creates a second replica set to do the rolling-update

kubectl create -f deployment.yml
kubectl describe deploy hello-deploy
kubectl get rs

Now update your git versioned deployment.yml and deploy again
kubectl apply -f deploy.yml --record
kubectl rollout status deployment hello-world
kubectl get deploy hello-deploy
kubectl rollout history deployment hello-deploy
kubectl get rs
kubectl describe deploy hello-world
kubectl rollout undo deployment hello-world --to-revision=1
kubectl get deploy
kubectl rollout status deployment hello-world
