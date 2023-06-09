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
docker buildx build --no-cache --platform linux/386,linux/amd64,linux/arm64/v8,linux/arm/v6,linux/arm/v7,linux/ppc64le,linux/s390x . -t andywanganqi/cmatrix --push
```

> Clean temp resources
```
docker system prune
```