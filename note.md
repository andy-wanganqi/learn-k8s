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
Notice the layers with USER and CMD

# 13.Container Networking Services and Volumns
