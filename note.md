# Start
Here is some resources for this course
* DiveInto Community[https://communityinviter.com/apps/diveintocommunity/join-diveinto]
* GitHub DiveIntoKubernetes Discussions[https://github.com/spurin/diveintokubernetes/discussions]

# 7. Container Images

> Pull container image
```
docker image pull docker.io/wernight/funbox:latest
docker pull docker.io/wernight/funbox@sha256:538c146646c327b0e988f9a53aa225c6b4a80fdde802befe77501784b4b59104
docker pull wernight/funbox
```

> Show images
```
docker images --digests
docker images
```

> Show manifest of an image
```
docker manifest inspect wernight/funbox
```

> Extract an image
```
docker save wernight/funbox -o funbox.tar
tar xvf funbox.tar
cat 538c146646c327b0e988f9a53aa225c6b4a80fdde802befe77501784b4b59104.json | python3 -m json.tool
```

> About top level digest
If an image covers many different variants, it will have a top level digest. [Alpine Example](https://explore.ggcr.dev/?image=alpine)

> Image history
```
docker history alpine
```

> Remove image
```
docker image remove alpine
docker rmi alphine
```

# 10. Running Containers

> Run wernight/funbox
```
docker run -it wernight/funbox
```

> Check the help of docker run
```
docker run --help
```

> List docker containers
```
docker ps -a
```

> Remove a docker container
```
docker rm [ id / name ]
```

> Override the default command in a Docker container when running it
```
docker run -it --rm wernight/funbox nyancat
docker run -it --rm wernight/funbox bash
```
* Notice the layers with USER and CMD

# 13.Container Networking Services and Volumns

> Run nginx in detach mode
```
docker run -d --rm -p 80:80 nginx
```
* -P will expose all ports
* -p 80:80 will map local:80 to container:80 port
* Notice the layer with EXPOSE 80

> Execute another process on a container
```
docker exec -it [container id] bash
```

> Linux command to find index.html file, and output any error to /dev/null
```
find / -name "index.html" 2>/dev/null
```
* The output might be: /usr/share/nginx/html/index.html

> Echo command
```
echo hello nginx > /usr/share/nginx/html/index.html
echo " Something" >> /usr/share/nginx/html/index.html
```
* \> will override the file
* \>> will append to the file

> Mount a local folder as a volumn to container
```
docker run -d --rm -p 80:80 -v [local folder]:/usr/share/nginx/html nginx
```
* -v local_folder:/usr/share/nginx/html will mount local folder to container as a volumn

# 16. Building Container Images
In this section, check the cmatrix folder, it will dive into creating a cmatrix docker image from alpine.

> Build a docker image
```
docker build . -t andywanganqi/cmatrix
```

> Run a container and check the user with whoami
```
docker run --rm -it andywanganqi/cmatrix whoami
```

> Build a docker image with multi arch and push
```
docker buildx create --use --name buildx-multi-linux
docker buildx use buildx-multi-linux
docker buildx build --no-cache --platform linux/amd64,linux/arm64/v8,linux/arm/v7,linux/arm/v6 . -t andywanganqi/cmatrix --push
```
Other linux platform: linux/386, linux/ppc64le, linux/s390x

> Clean temp resources
```
docker system prune
```

# 24. Container Runtimes
* Docker, containerd and runc
* CRI-O, Kata Containers and gVisor
* cri-dockerd, Mirantis: https://github.com/Mirantis/cri-dockerd

# 26. Hands on with Container Runtimes
```
docker run --rm -it ubuntu bash
```
Docker command as above is based on the runtime

Trying to use ctr to manage the container lifecycle:
```
ctr images pull docker.io/library/ubuntu:latest
ctr run -t -d docker.iio/library/ubuntu:latest quirky-name
ctr container list
ctr task list
ps -ef --forest
ctr task attach quirky-name
ctr container delete quirky-name
```

## About nerdctl
Install nerdctl, and its dependency of cni (Lab resources)
```
ls -l /resources/*linux*
tar -xzvf /resources/nerdctl-1.1.0-linux.tar.gz --directory /usr/local/bin nerdctl
tar -xzvf /resources/cni-plugins-linux-v1.2.0.tgz -C /opt/cni/bin
```

nerdctl network is supported by cni
```
nerdctl run --rm -it ubuntu bash
apt update
apt install -y iproute2
ip addr
```

cni config is created, and we can make a backup of it
```
cat /etc/cni/net.d/nerdctl-bridge.conflist
mv /etc/cni/net.d/nerdctl-bridge.conflist /resources
```

Then we can run a container by nerdctl
```
nerdctl run --rm -it quirky-name ubuntu bash
```

# 28 Installing Kubernetes
Some tools can be used for kubernetes: K3D, K3S, MicroK8s, Kind.

Kubeadm is more suitable for learning k8s. Kubeadm built clusters are often referenced in the official kubernetes exams.

Kubernetes certificates
> Kubernetes and CLOUD NATIVE ASSOCIATE
> CERTIFIED kubernetes ADMINISTRATOR
> CERTIFIED kubernetes APPLICATION DEVELOPER

Official kubernetes documentation can be refered at https://kubernetes.io

Start kubeadm initialization with specified IP address range for Kubernetes pods
```
kubeadm init --pod-network-cidr=10.10.0.0/16 --kubernetes-version=1.26.0
```

Note the output below:
```
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
```

And note the prompt to start using the cluster
```
To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf
```

Then we can see the first node with name control-plane, and we can see it is not ready, because CNI is not ready.
```
kubectl get nodes
```

Set up CNI by using bridge plugins (Lab resources)
```
sleep 3 && cp /resources/cni/10-bridge.conf /etc/cni/net.d & watch kubectl get nodes -o wide
```

Note the NoSchedule taint in node description, that is to restrict workloads from being schedules on certain nodes
```
kubectl describe node control-plane | more
```

Here is okay to remove it as learning kubernetes
```
kubectl taint node control-plane node-role.kubernetes.io/control-plane:NoSchedule-
```

# 31. Kubernetes Pods
Pod is the smallest deployable compute unit you can create and manage in kubernetes.

Pod provides shared networking and storage for one or more containers.

You can think of a pod as an isolated host with the serving the needs of the application it serves.

Containers run within the pod and will share networking of that provided by the pod.

Run nginx in kubernetes and watch it:
```
kubectl run nginx --image=nginx; watch kubectl get pods -o wide
kubectl get pods -o wide
```

Check the containers and check the namespace of kubernetes that is k8s.io
```
nerdctl ps -a
nerdctl namespace ls
nerdctl -n k8s.io ps -a
```

Start a pod with --restart=Never and env, and then delete it
```
kubectl run alpine --image=alpine --env="MY_VARIABLE=Hello, Kubernetes!" --restart=Never -it -- /bin/sh
env
kubectl delete pod/alpine
```

# 35. The Pause Container
Every pod has a pause container. The pause container provides shared namespaces for containers. 

Your favourite containers run inside the pod alongside the pause container providing a host like environment for containers.

With an example of a pod providing a host for a pause container and other two that are a MySQL and Wordpress within the shared namespace.
```
docker run --rm -d --ipc=shareable --name pause -p 8080:80 registry.k8s.io/pause:3.9
docker run --rm -d --name mysql -e MYSQL_DATABASE=exampledb -e MYSQL_USER=exampleuser -e MYSQL_PASSWORD=examplepass -e MYSQL_RANDOM_ROOT_PASSWORD=1 --net=container:pause --ipc=container:pause --pid=container:pause mysql:8
docker run --rm -d --name wordpress -e WORDPRESS_DB_HOST="127.0.0.1" -e WORDPRESS_DB_USER=exampleuser -e WORDPRESS_DB_PASSWORD=examplepass -e WORDPRESS_DB_NAME=exampledb --net=container:pause --ipc=container:pause --pid=container:pause wordpress:6.1.1
```

We can test the shared memory between two containers inside the pod:
```
docker exec -it mysql sh
echo hello > /dev/shm/test
docker exec -it wordpress sh
cat /dev/shm/test
```

We can see all the containers running with a PID when we go into any container. And if we kill the paused one, then all of them inside the pod will exit.
```
ps -ef
kill 1
```

Think of the pause container as allowing container processes to be running and visible to each other. As if you had started different processes on your main system, they will share the same network and will be able to communicate using IPC.

Last one, use this to clean up the pod above.
```
docker stop pause mysql wordpress
```

# 37. Kubernetes Pod Networking (Lab Resources)
List all paused k8s containers
```
nerdctl -n k8s.io ps -a | grep -v pause
```

We can use this way to grab the container id of the un-paused nginx container into NGINXID
```
NGINXID=$(nerdctl -n k8s.io ps -a | grep nginx | grep -v pause | awk {'print $1'}); echo $NGINXID
```

If we stop a nginx container in a pod, then it will recreate the nginx container.
```
nerdctl -n k8s.io stop $NGINXID; watch kubectl get pods -o wide
```

Bug if we stop the pause container in the pod, then k8s will recreate the pause container and the nginx container, but with different IP.
```
nerdctl -n k8s.io ps -a | grep nginx | grep pause
nerdctl -n k8s.io stop $PAUSEID; watch kubectl get pods -o wide
```

Then we can clean up the nginx container
```
kubectl delete pod/nginx --grace-period=0
```

# 41. Kubernetes YAML
We can generate output of the equivalent yaml by using --dry-run with a kubernetes command:
```
kubectl run nginx --image=nginx --dry-run=client -o yaml > /etc/kubernetes/nginx_pod.yaml
```

Or we can choose a pod and output the yaml file
```
kubectl run alpine --image=alpine
kubectl get pod alpine -o yaml > /etc/kubernetes/alpine_pod.yaml
```

And the yaml file will be like:
```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

Then kubectl apply and kubectl create can be used to manage resources with yaml file
> kubectl apply
kubectl apply: is used to create or update resources based on the contents of a YAML or JSON file. It will first check if a resource with the same name and namespace already exists in the cluster. If it does, it will udpate the existing resource with the new configuration. If it does not , it will create a new resource with the specified configuration.
```
kubectl apply -f /etc/kubernetes/resources/nginx_pod.yaml
```

> kubectl create
kbuectl create: is used to create new resources based on the contents of a YAML or JSON file. It will not check if a resource with the same name and namespace already exists in the cluster. If it does, it will return an error and the resource will not be created. This command is useful if you want to ensure that a new resource is created, even if one with the same name and namespace already exists in the cluster.

The same yaml file can also be used to delete container
```
kubectl delete --grace-period=0 -f /etc/kubernetes/resources/nginx_pod.yaml
```

# 44. Kubernetes Deployments
Deployment is a declarative means for defining a desired state.

We can use deployments to specify how an application should run as well as the number of replicas that should be running.

Deployments also provide useful interaction points for other areas in k8s such as Services.

Here is an example to create a nginx deployment with port 80
```
kubectl create deployment nginx --image=nginx --port 80
kubectl get deployment
kubectl get replicaset
```

A ReplicaSet is automatically created with the deployment. ReplicaSet is a **controller** that ensures that a specified number of replica pods are running and available at all times.

A ReplicaSet creates and manages a set of identical Pod replicas, and is responsible for scaling the number of replicas up or down based on the desired state specified in the Kubernetes configuration.

In order to scale deployment, we can use:
```
kubectl scale deployment/nginx --replicas=2; watch kubectl get pods -o wide
```

Extra things, using kubectl edit to modify the deployment, and then use kubectl describe, use kubectl rollout status to check status.
```
kubectl edit deployment/nginx
kubectl describe deployment/nginx
kubectl rollout status deployment/nginx
```

# 47. Kubernetes Services
Kubernetes Services are a way to expose your application running on a set of pods as a network service.

Services provide a stable endpoint for external clients to access the application, as well as load balancing and service discovery.

## Cluster IP Service
This is the default type of service and creates a virtual IP (VIP) that is only accessible within the cluster.

This type of service is useful for communication between different parts of your application that are running in the cluster.

```
kubectl expose deployment/nginx --type=ClusterIP --name=nginx-service
kubectl get service
IP=$(kubectl get service | grep nginx | awk {'print $3'}); echo $IP
curl $IP
```
The service will have a name nginx-service, and a DNS name nginx-service.default.svc.cluster.local would be available inside cluster.

## NodePort
This type of service exposes the service on a specific port on each node in the cluster. This allows external traffic to access the service by connecting to any node IP and the specified port.

## LoadBalancer
This type of service creates a load balancer in the cloud provider's infrastructure (such as an AWS ELB or a Google Cloud Load Balancer) and forwards traffic to the service. This is useful for exposing your application to external clients.

## ExternalName
This type of service creates a CNAME record that maps a service name to an external DNS name. This is useful for connecting to services that are NOT in your cluster, such as a database running in a different cluster or a different cloud provider.

## Headless Service
This type of service is similar to a ClusterIP service but does not create a virtual IP. Instead, it allows you to directly access individual pods, useful for scenarios where you need direct communication between pods and/or fine-grained control over load balancing.

# 50. Kubernetes DNS
Kubernetes is provided with a built-in DNS service that automatically assigns DNS names to Services and Pods in the cluster.
```
<servicename>.<namespace>.svc.cluster.local
<podname>.<namespace>.pod.cluster.local
```

A example of a pod with IP 10.10.0.50 will have DNS name:
```
10-10-0-50.default.pod.cluster.local
```

Below is an example to show the pods can communicate with each other
```
kubectl run hello-world --image=gcr.io/google-samples/hello-app:1.0 --port 8080
HELLO_IP=$(kubectl get pods -o wide | grep hello-world | awk {'print $6'}); echo $HELLO_IP
curl $HELLO_IP:8080
kubectl run curl --image=curlimages/curl --restart=Never -- sh -c "sleep infinity"
```
Above is to run two pods, one is listening to 8080, and the other one will curl to the previous one.

```
kcurl() { kubectl exec -it curl -- sh -c "curl $1"; }
knslookup() { kubectl exec -it curl -- sh -c "nslookup $1"; }
kcurl example.com
knslookup example.com
```
Above is to create two functions, and then we can test them against example.com. These functions are indeed exeucting inside from the pod.

```
kcurl nginx.default.svc.cluster.local
knslookup nginx.default.svc.cluster.local
HELLO_IP_DASHES=$(echo $HELLO_IP | sed 's/\./-/g'); echo $HELLO_IP_DASHES
kcurl $HELLO_IP_DASHES.default.pod.cluster.local:8080
knslookup $HELLO_IP_DASHES.default.pod.cluster.local
```
Then we can check whether the kubernetes DNS is working against service and pod.

Eventually we can clean up these resources.
```
kubectl delete deployment/nginx
kubectl delete service/nginx
kubectl delete --grace-period=0 pod/hello-world pod/curl
```

# 53. Kubernetes Architecture Dissection
Use these commands to check the contains in containerd, and the pause pods in kubernetes. As well as kubelet is aside.
```
nerdctl -n k8s.io ps -a | grep -v pause | grep Up
kubectl get all -A

ps -ef | grep kubelet | grep -v kube-apiserver | grep -v grep
```

Then we can check those kube-system things into yaml.
```
kubectl -n kube-system get deamonset.apps/kube-proxy -o yaml > /etc/kubernetes/resources/kube-proxy.yaml
```
kube-proxy has DESIRED, CURRENT, READY numbers which will map the number of pods.

```
kubectl -n kube-system get deamonset.apps/kube-dns -o yaml > /etc/kubernetes/resources/kube-dns.yaml
kubectl -n kube-system get deamonset.apps/coredns -o yaml > /etc/kubernetes/resources/coredns.yaml
```
These two provide the dns service.

We also have the kubelet, running as an independent process under systemctl.
```
ps -ef | grep kubelet | grep -v kube-apiserver | grep -v grep
systemctl status kubelet
```

If we move the kubernetes manifests, then other kube-* static pods will have stopped
```
mv /etc/kubernetes/manifests/* /etc/kubernetes/resources
```

Below are two commands used to stop and remove anything that is running within containerd
```
alias nerdctl_stopall="nerdctl -n k8s.io ps -a | grep -v CONTAINER | cut -d ' ' -f 1 | xargs -i sh -c 'nerdctl -n k8s.io stop {} || true'"
alias nerdctl_rmall="nerdctl -n k8s.io ps -a | grep -v CONTAINER | cut -d ' ' -f 1 | xargs -i sh -c 'nerdctl -n k8s.io rm {} || true'"

nerdctl_stopall
nerdctl_rmall
```

So far, only containerd is running.
