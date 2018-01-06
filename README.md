[docs]
https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/
download kubectl
download & install minikube

[commands]
#!/bin/bash
alias k=kubectl
alias mk=minikube

mk start

mk start --vm-drive=xhyve --kubernetes-version="v1.6.0"

mk start --vm-driver virtualbox --alsologtostderr --iso-url=https://storage.googleapis.com/mk-builds/1542/mk-testing.iso --cpus 4 --memory 8192 --disk-size 50g

mk start --v=7 --alsologtostderr --vm-driver virtualbox --disk-size 90g --memory 12288 --cpus 3 --hyperv-virtual-switch ExternalSwitch --mount --mount-string "C:\Users\xxx\workspace\go\src:/go/src"

#defaults to virtualbox

mk start --vm-drive=hyperv

mk delete
mk start --vm-drive=xhyve --kubernetes-version="v1.6.0" && mk start --cpus 4 --memory 8192

mk  version

mk stop

mk delete

mk  dashboard

mk ip

mk ssh

k.exe get nodes

k.exe config current-context

#kubeadm - for manual install on all nodes (master and node)
Install:
docker (or rkt) - container runtime
kubelet (server)
k (client)
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

apt-get install docker.io kubeadm k kubelet kubernetes-cni

# create a single node cluster
kubeadm init

then copy paste and run commands as reg user:
sudo cp /etc/kubernetes/admin.conf $HOME/
sudo chown $(id -u):$(id -g) $HOME/admin.conf
export KUBECONFIG=$HOME/admin.conf

#not use it

k get nodes

k describe node mk


#will tell you that there is no dns pod available
k get pods --all-namespaces

#now create a pod network - mandatory to get the cluster up and running
k apply --filename https://git.io/weave-kube-1.6

#now add more nodes - installed deps as above
kubeadm join --token [take it from the result of previous kubeadm init]

# not you should be able to see them all
k get nodes


#Running an image

k run echo --image=gcr.io/google_containers/echoserver:1.4 --port=8080
k describe deployment echo
k describe pod
k describe pod echo-3004094957-srh3b
k get pods
 
k expose deployment echo --type=NodePort --port=80 --target-port=8080
k describe service echo
#get the auto ip of NodePort (not ip of service(inside cluster only) or targetport (oncontainer)) then go in browser: http://192.168.99.100:32525/
from inside the cluster (mk ssh) you can access the service ip and port: 



wget 10.0.0.236 && cat index.html
or
curl 10.0.0.236

k delete service echo
k delete deployment echo

k delete service echo-service
k delete -f looper/echo-deployment.yml

#connecting to a pod
k get pods
k exec -it shell-demo -- /bin/bash
##works on windows for exception: Git/usr/bin/bash: no such file or directory\
k exec -it echo-deployment-3745578590-24klf -- bash -c 'bash'

#executing individual commands in a container
k exec shell-demo env

#list ports open in container - none present - but you could yum install them - 

curl localhost:8080

sudo lsof -i -P -n | grep LISTEN 
sudo netstat -tulpn | grep LISTEN
sudo nmap -sTU -O IP-address-Here

$ sudo lsof -i -P -n
$ sudo lsof -i -P -n | grep LISTEN
$ doas lsof -i -P -n | grep LISTEN ### [OpenBSD] ###

#
k exec -it my-pod --container main-app -- /bin/bash

k exec echo-deployment-1415127006-rnxrn -- printenv
k exec echo-deployment-1415127006-rnxrn -- nslookup echo-service


#Exposing a service externally
• Configure nodePort for direct access

add to service definition
type: NodePort
ports:
- port: 8080
targetPort: 80
protocol: TCP
name: http
- port: 443
protocol: TCP
name: https
selector:
app: blue-reminders

k describe service echo-service 

' get the nodeport'
now connect directly to that port on that protocol - external cluster ip (virtual machine)


• Configure a Cloud load balancer if you run it in a Cloud environment

Ingress
Ingress is a Kubernetes configuration object that lets you expose a service to the
outside world and take care of a lot of details. It can do the following:
• Provide an externally visible URL to your service
• Load-balance traffic
• Terminate SSL
• Provide name-based virtual hosting

#Ingress
Ingress is a Kubernetes configuration object that lets you expose a service to the
outside world and take care of a lot of details. It can do the following:
• Provide an externally visible URL to your service
• Load-balance traffic
• Terminate SSL
• Provide name-based virtual hosting

#namespace

k create -f looper/looper-namespace.yml
k delete -f looper/looper-namespace.yml


##create a context - will allow restricting access only to this namespace
kubectl config set-context looper --namespace=looper-namespace --cluster=minikube --user=minikube
kubectl config use-context looper
k config view
k get pods
k get pods --namespace=default
k get pods --all-namespaces


#jobs - single command execution

k create -f looper/looper-job.yml
k describe jobs/factorial5

k get pods --show-all

k get pods --show-all --selector=job-name=factorial5 --output=jsonpath={.items..metadata.name}
k logs factorial5-0c51
k logs $(kubectl get pods --show-all --selector=job-name=factorial5 --output=jsonpath={.items..metadata.name})

k create -f looper/looper-parallel-jobs.yml

k delete jobs/factoial5
k delete jobs/sleep20

#Scheduling cron jobs
You must enable cron jobs by starting the API server
with the following:
--runtime-config=batch/v2alpha1

k create -f looper/looper-cronjob.yml
k delete cronjobs/stretch
k delete jobs -l name=stretch

#use json output selector
--output=jsonpath={.items..metadata.name}

#use label selector

#create docker image

k ssh

echo "FROM alpine:3.5" >> DockerFile
echo CMD sh -c "echo 'Started...'; while true ; do sleep 10 ; done" >> Dockerfile

docker build -t looper
docker tag looper docmseco/looper
docker push docmseco/looper

use in image as: docmseco/looper

k create -f looper/looper-pod.yml
k get pods
k describe pod looper-pod
k logs looper-pod
k delete pods looper-pod

k create -f looper/looper-deployment.yml
k describe deployment looper-deployment
k get pods
#update
k apply -f looper/looper-deployment.yml
k delete deployment looper-deployment

#Working with pods:

Feed the manifest file describing the pod to the apiserver{} yml or json

k create -f nigel-pod.yml
or
k create -f nigel-pod.json

k describes pods

k get pods/hello-pod

k get pods -all-namespaces

k delete pods hello-pod

#Service: types:
ClusterIP - stable internal cluster ip
NodePort: Exposes the app outside of the cluster by adding a cluster-wide port on top of ClusterIP
LoadBalancer: Integrates NodePort with cloud-based load balancers

k expose rc hello-rc --name=hello-service --target-port=8080 --type=NodePort
or
k expose deployment echo --type=NodePort

k describe svc hello-service

port=k get service echo --output='jsonpath="{.spec.ports[0].NodePort}"'
curl http://192.168.99.100:$(port)/hi

#also see the created endpoint object
k describe ep hello-service

# go in the browser and check

k delete svc hello-service

#deployment
are all about updates and rollback
Without deployments:
define new replication controller file - with a new version then run:
k rolling-update -f updated-rc.yml

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

k create -f deployment.yml
k describe deploy hello-deploy
k get rs

Now update your git versioned deployment.yml and deploy again
k apply -f deploy.yml --record
k rollout status deployment hello-world
k get deploy hello-deploy
k rollout history deployment hello-deploy
k get rs
k describe deploy hello-world
k rollout undo deployment hello-world --to-revision=1
k get deploy
k rollout status deployment hello-world

#Liveness probes

apiVersion: v1
kind: Pod
metadata:
    name: blue-music
    labels:
        app: blue-music
spec:
    containers:
        image: gcr.io/google_containers/echoserver:1.4
        livenessProbe:
            httpGet:
                path: /pulse
                port: 8080
                httpHeaders:
                - name: X-Custom-Header
                 value: Awesome
        initialDelaySeconds: 30
        timeoutSeconds: 1
        name: blue-music
        
There are two other types of probe:
• TcpSocket – Just check that a port is open
• Exec – Run a command that returns 0 for success        

#Using readiness probes to manage dependencies
wait for other services to start

readinessProbe:
    exec:
        command:
            - /usr/local/bin/checker
            - --full-check
            - --data-service=blue-multimedia-service
        initialDelaySeconds: 60
        timeoutSeconds: 5
        
#Employing init containers for orderly pod bring-up

apiVersion: v1
kind: Pod
metadata:
    name: blue-fitness
    annotations:
        pod.beta.kubernetes.io/init-containers: '[
            {
                "name": "install",
                "image": "busybox",
                "command": ["/support/safe_init"],
                "volumeMounts": [
                    {
                        "name": "workdir",
                     "mountPath": "/work-dir"
                    }
                ]
            }
        ]'
spec:        

#Sharing with DaemonSet pods
typically used for keeping an eye on nodes: Monitoring, Logging, and Troubleshooting

apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
    name: blue-collect-proxy
    labels:
        tier: stats
        app: blue-collect-proxy
spec:
    template:
        metadata:
            labels:
                blue-collect-proxy
        spec:
            hostPID: true
            hostIPC: true
            hostNetwork: true
            containers:
                image: the_g1g1/blue-collect-proxy
                name: blue-collect-proxy

#Persistent volumes walkthrough
##Using emptyDir for intra-pod communication

###emptyDir - some empty dir on the host

Container 1 and container 2 simply mount the same volume and can
communicate by reading and writing to this shared space.
An emptyDir volume is an empty directory on the host. Note that
it is not persistent because when the pod is removed from the node, the contents
are erased. If a container just crashes, the pod will stick around and you can access
it later.

apiVersion: v1
kind: Pod
metadata:
  name: looper-volume
spec:
  containers:
  - name: blue-global-listener
    image: the_g1g1/blue-global-listener
    volumeMounts:
    - mountPath: /notifications
      name: sharedvolume
  - image: the_g1g1/blue-job-scheduler
    name: blue-job-scheduler
    volumeMounts:
    - mountPath: /incoming
      name: sharedvolume
  volumes:
  - name: sharedvolume
    emptyDir: {}

or in volatile RAM option:
  volumes:
    - name: sharedvolume
      emptyDir: {}        
       medium: Memory

##Using HostPath for intra-node communication
you want pods on the same node to communicate with each other. 
There are two cases where a pod can rely on other
pods being scheduled with it on the same node:
• In a single-node cluster all pods obviously share the same node
• DaemonSet pods always share a node with any other pod that matches
their selector. DeamonSet pod that serves as an aggregating proxy to other pods.
simply write their data to a mounted
volume that is bound to a host directory and the DaemonSet pod can directly read
it and act on it.
limitations:
• The behavior of pods with the same configuration might be different if they
are data-driven and the files on their host are different.
• It can violate resource-based scheduling (coming soon to Kubernetes)
because Kubernetes can't monitor HostPath resources.
• The containers that access host directories must have a security context
with privileged set to true or, on the host side, you need to change the
permissions to allow writing.

apiVersion: v1
kind: Pod
metadata:
    name: blue-coupon-hunter
spec:
    containers:
    - image: the_g1g1/blue-coupon-hunter
      name: blue-coupon-hunter
      volumeMounts:
      - mountPath: /coupons
        name: coupons-volume
      securityContext:
        privileged: true
    volumes:
    - name: coupons-volume
      host-path:
        path: /etc/blue/data/coupons

#Provisioning persistent volumes
##Provisioning persistent volumes statically
##Provisioning persistent volumes dynamically

##Creating persistent volumes
apiVersion: v1
kind: PersistentVolume
metadata:
    name: pv-1
    annotations:
        volume.beta.kubernetes.io/storage-class: "normal"
    labels:
        release: stable
        capacity: 100Gi
spec:
    capacity:
        storage: 100Gi
    accessModes:
    - ReadWriteOnce
    - ReadOnlyMany
    persistentVolumeReclaimPolicy: Recycle
    nfs:
        path: /tmp
        server: 172.17.0.8
    
• ReadOnlyMany: Can be mounted read-only by many nodes
• ReadWriteOnce: Can be mounted as read-write by a single node
• ReadWriteMany: Can be mounted as read-write by many nodes
NFS supports all modes

Reclaim policy
• Retain – the volume will need to be reclaimed manually
• Delete – the associated storage asset such as AWS EBS, GCE PD, Azure disk,
or OpenStack Cinder volume is deleted
• Recycle – delete content only (rm -rf /volume/*)

The Retain and Delete policies mean the persistent volume is not available anymore
for future claims. The recycle policy allows the volume to be claimed again.

Currently, only NFS and HostPath support recycling. AWS EBS, GCE PD, Azure
disk, and Cinder volumes support deletion. Dynamically provisioned volumes are
always deleted.

#Making persistent volume claims

kind: PersistentVolumeClaim
apiVersion: v1
metadata:
    name: storage-claim
    annotations:
        volume.beta.kubernetes.io/storage-class: "normal"
spec:
    accessModes:
    - ReadWriteOnce
    resources:
        requests:
            storage: 80Gi
    selector:
        matchLabels:
            release: "stable"
        matchExpressions:
        - {key: capacity, operator: In, values: [80Gi, 100Gi]}

persistent volume claims belong to a namespace. Binding a persistent
volume to a claim is exclusive.

#Mounting claims as volumes
kind: Pod
apiVersion: v1
metadata:
    name: the-pod
spec:
    containers:
    - name: the-container
      image: some-image
      volumeMounts:
      - mountPath: "/mnt/data"
        name: persistent-volume
    volumes:
    - name: persistent-volume
      persistentVolumeClaim:
          claimName: storage-claim

#Storage classes
Storage classes let an administrator configure your cluster with custom persistent
storage (as long as there is a proper plugin to support it).

kind: StorageClass
apiVersion: storage.k8s.io/v1beta1
metadata:
    name: standard
provisioner: kubernetes.io/aws-ebs
parameters:
    type: gp2

#The currently supported volume types are as follows:
• emptyDir
• hostPath
• gcePersistentDisk
• awsElasticBlockStore
• nfs
• iscsi
• flocker
• glusterfs
• rbd
• cephfs
• gitRepo
• secret
• persistentVolumeClaim
• downwardAPI
• azureFileVolume
• azureDisk
• vsphereVolume
• Quobyte

#Default storage class

The cluster administrator can also assign a default storage class. When a default
storage class is assigned and the DefaultStorageClass admission plugin is turned
on, then claims with no storage class will be dynamically provisioned using the
default storage class. If the default storage class is not defined or the admission
plugin is not turned on, then claims with no storage class can only match volumes
with no storage class.

kubectl create -f persistent-volume.yaml
kubectl get pv

#Inside the node, we can communicate with the containers using Docker commands. Let's look at the last two running 
containers
docker ps -n=2

#AWS Elastic Block Store (EBS)
AWS provides the elastic block store as persistent storage for EC2 instances.
An AWS Kubernetes cluster can use AWS EBS as persistent storage with the
following limitations:
• The pods must run on AWS EC2 instances as nodes
• Pods can only access EBS volumes provisioned in their availability zone
• An EBS volume can be mounted on a single EC2 instance.
Those are severe limitations.

apiVersion: v1
kind: Pod
metadata:
    name: some-pod
spec:
    containers:
    - image: some-container
      name: some-container
      volumeMounts:
      - mountPath: /ebs
        name: some-volume
      volumes:
      - name: some-volume
        awsElasticBlockStore:
            volumeID: <volume-id>
            fsType: ext4
            
You must create the EBS volume in AWS and then you just mount it into the pod.
There is no need for a claim or storage class because you mount the volume directly
by ID. The awsElasticBlockStore volume type is known to Kubernetes.

#AWS Elastic File System (EFS) - a managed NFS service.
• Multiple EC2 instances can access the same files across multiple availability
zones (but within the same region)
• Capacity is automatically scaled up and down based on actual usage
• You pay only for what you use
• You can connect on-premise servers to EFS over VPN
• EFS runs off SSD drives that are automatically replicated across availability
zones

From a
Kubernetes point of view, AWS EFS is just an NFS volume. You provision it as such:

apiVersion: v1
kind: PersistentVolume
metadata:
    name: efs-share
spec:
    capacity:
        storage: 200Gi
    accessModes:
    - ReadWriteMany
    nfs:
        server: eu-west-1b.fs-64HJku4i.efs.eu-west-1.amazonaws.com
        path: "/"
        
Once the persist volume exists, you can create a claim for it, attach the claim as a
volume to multiple pods (ReadWriteMany access mode), and mount it into containers.

#GCE persistent disk
very similar to awsElasticBlockStore.
You must provision the disk ahead of time. It can only be used by GCE instances
in the same project and zone. But the same volume can be used as read-only on
multiple instances. This means it supports ReadWriteOnce and ReadOnlyMany. You
can use a GCE persistent

apiVersion: v1
kind: Pod
metadata:
    name: some-pod
spec:
    containers:
    - image: some-container
      name: some-container
      volumeMounts:
      - mountPath: /pd
        name: some-volume
    volumes:
    - name: some-volume
      gcePersistentDisk:
        pdName: <persistent disk name>
        fsType: ext4

#Azure data disk - imilar in capabilities to AWS EBS.

apiVersion: v1
kind: Pod
metadata:
name: some-pod
spec:
    containers:
    - image: some-container
      name: some-container
      volumeMounts:
      - name: some-volume
        mountPath: /azure
    volumes:
    - name: some-volume
      azureDisk:
        diskName: test.vhd
        diskURI: https://someaccount.blob.microsoft.net/vhds/test.vhd

In addition to the mandatory diskName and diskURI parameters, it also has a few
optional parameters:
• cachingMode: The disk caching mode. This must be one of None, ReadOnly,
or ReadWrite. The default is None.
• fsType: The filesystem type set to mount. The default is ext4.
• readOnly: Whether the filesystem is used as readOnly. The default is false.
Azure data disks are limited to 1,023 GB. Each Azure VM can have up to 16 data
disks. You can attach an Azure data disk to a single Azure VM.

#Azure file storage shared filesystem similar to AWS EFS.
  In order to use Azure file storage, you need to install on each client VM the
  cifs-utils package.
  
apiVersion: v1
kind: Secret
metadata:
    name: azure-file-secret
type: Opaque
data:
    azurestorageaccountname: <base64 encoded account name>
    azurestorageaccountkey: <base64 encoded account key>

Here is a configuration file for Azure file storage:

apiVersion: v1
kind: Pod
metadata:
    name: some-pod
spec:
    containers:
    - image: some-container
      name: some-container
      volumeMounts:
      - name: some-volume
        mountPath: /azure
    volumes:
    - name: some-volume
      azureFile:                    
        secretName: azure-file-secret
       shareName: azure-share
        readOnly: false

                   
                   
            