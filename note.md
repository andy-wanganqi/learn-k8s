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

