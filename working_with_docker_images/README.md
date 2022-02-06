# Working with Docker Images

Two ways to make a docker image
- Tweaking docker container and commiting them
- Build docker images using docker files

## Creating an image from the container

```bash
$ docker container run -it ubuntu bash
root@d2b396a5cc25:/#
root@d2b396a5cc25:/# apt-get update
Get:1 http://archive.ubuntu.com/ubuntu focal InRelease [265 kB]
...
root@d2b396a5cc25:/# apt-get install apache2
Reading package lists... Done
Building dependency tree      
...
```

In different terminal
```bash
$ docker container ls
CONTAINER ID   IMAGE     COMMAND   CREATED         STATUS         PORTS     NAMES
d2b396a5cc25   ubuntu    "bash"    7 minutes ago   Up 7 minutes             competent_feistel
$ 
$ docker container commit --author "Himanshu Patel" --message "Ubuntu with apache2" competent_feistel ubuntu_apache
sha256:7ec2fb44623cf7607ea08309ca11406e024319fccc84d61d4b591f4e97ae6865
$ 
$ docker images ls
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
$ docker image ls
REPOSITORY      TAG       IMAGE ID       CREATED          SIZE
ubuntu_apache   latest    7ec2fb44623c   10 seconds ago   220MB
ubuntu          latest    597ce1600cf4   4 months ago     72.8MB
```

We learned that Docker images are layered and that each layer is stacked on top of its parent image. When we start a container, a read/write ephemeral filesystem layer gets created. The changes to the filesystem (that is, the addition,
modification, or deletion of files) through the `apt-get` update and `install` commands are preserved in this read/write ephemeral filesystem layer. If we stop and delete the container, the ephemeral layer associated with that container is removed and, essentially, we lose all the changes we applied to the container.

In this recipe, we persisted the container's ephemeral layer by using the `docker container commit` command. In effect, the commit operation creates another image layer and saves it along with other images in the Docker host.

The `docker container diff` command lists all the changes to the container filesystem from its image, as shown in the following code:

```bash
Learn-Docker $ docker container diff competent_feistel
A /var/lib/dpkg/triggers/File
C /var/lib/dpkg/triggers/Lock
C /var/lib/dpkg/triggers/Unincorp
A /var/lib/dpkg/triggers/update-ca-certificates
C /var/lib/dpkg/statoverride
A /var/lib/dpkg/status-old
...
```

- `A`: This is for when a file/directory has been added
- `C`: This is for when a file/directory has been modified
- `D`: This is for when a file/directory has been deleted

By default, a container gets paused while doing the commit. You can change its behavior by passing `--pause=false` to commit.

## Logging in and out of the Docker image registry

The `docker login` command lets you log in to more than one Docker registry at the same time. Similarly, the `docker logout` command lets you log out from the specified server. Here is the syntax for the Docker login and logout commands:
```bash
$ docker login [OPTIONS] [SERVER]
$ docker logout [SERVER]
```
By default, both the docker login and docker logout commands assume `https:/​/​hub.docker.​com/​` as the default registry, but this can be changed.

```bash
$ docker login
Username: patelhimanshu
Password: 
Login Succeeded
```

We can see the registery in which we logged in in docker config. Here I logged in only one registery so we can see only one entry. You can change this interactive behavior to batch mode by supplying the username using the `-u` or `--username` option and the password using the `-p` or `--password` option.

```bash
$ cd /home/himanshu/.docker
$ cat config.json 
{
        "auths": {
                "https://index.docker.io/v1/": {
                        "auth": "cGF0ZWxoasdrbnNodTpGdnnnc3RhY2tKKKKKKK=="
                }
        }
}
```

```bash
$ docker logout
Removing login credentials for https://index.docker.io/v1/
 $ cat config.json 
{
        "auths": {}
}
```

## Publishing an image to the registry

The following are two syntaxes for the command that is used to push a Docker image to a registry:
```bash
$ docker image push [OPTIONS] NAME[:TAG]
$ docker push [OPTIONS] NAME[:TAG]
```

Steps to publish an image

1. tagging the image using the `docker image tag`.
```bash
# see previous images
 $ docker image ls
REPOSITORY      TAG       IMAGE ID       CREATED        SIZE
ubuntu_apache   latest    7ec2fb44623c   10 hours ago   220MB
ubuntu          latest    597ce1600cf4   4 months ago   72.8MB

# tag the image with username/imagename
$ docker image tag ubuntu_apache:latest patelhimanshu/ubuntu_apache

# see all images again
 $ docker image ls
REPOSITORY                    TAG       IMAGE ID       CREATED        SIZE
ubuntu_apache                 latest    7ec2fb44623c   10 hours ago   220MB
patelhimanshu/ubuntu_apache   latest    7ec2fb44623c   10 hours ago   220MB
ubuntu                        latest    597ce1600cf4   4 months ago   72.8MB
```

The image is tagged as `patelhimanshu/ubuntu_apache` because, in the next step, we will be pushing this image to the Docker Hub user `patelhimanshu`.

2. Push the image to repo. (make sure you are logged in the repo)
```bash
 $ docker image push patelhimanshu/ubuntu_apache
Using default tag: latest
The push refers to repository [docker.io/patelhimanshu/ubuntu_apache]
c7273f33dc88: Pushed 
da55b45d310b: Mounted from library/ubuntu 
latest: digest: sha256:e17105e42540eb11ab701b2a78837f3a33a819d2e9294681c113b5fe57a14629 size: 741
```

The docker image push command identifies all the image layers that make up the image that is to be pushed, and check whether any of those images are already available in the registry. Then the push command uploads all the images that are not present in the registry.

Let's say you want to push the image to a registry that is hosted locally. To do this, you first need to tag the image with the registry host's name or IP address with the port number on which the registry is running, and then push the images.
For example, let's say our registry is configured on `shadowfax.example.com`. To tag the
image, we would use the following command:
```bash
$ docker tag ubuntu_apache shadowfax.example.com:5000/patelhimanshu/ubuntu_apache
```
Then, to push the image, we would use the following command:
```bash
$ docker push shadowfax.example.com:5000/patelhimanshu/ubuntu_apache
```

## Looking at the history of an image
The `docker image history` command helps us find all the layers in the image, its image ID, when it was created, how it was created, the size, and any additional comments associated with that layer. When we build a Docker image, the Docker engine preserves the build instruction in the image metadata.

```bash
# Template
$ docker image history [OPTIONS] IMAGE

# example
$ docker image history ubuntu_apache:latest 
IMAGE          CREATED        CREATED BY                                      SIZE      COMMENT
7ec2fb44623c   11 hours ago   bash                                            147MB     Ubuntu with apache2
597ce1600cf4   4 months ago   /bin/sh -c #(nop)  CMD ["bash"]                 0B        
<missing>      4 months ago   /bin/sh -c #(nop) ADD file:8d2f4a45a58b3f542…   72.8MB
```

## Removing an image
The image's name along with its tag. If the tag is not specified, then the `latest` tag is assumed by default.

If the image happens to have more than one tag associated with it, then those tags must be removed before removing the image. Alternatively, you can forcefully remove them using the `-f` or `--force` option of the docker image rm command. In such a case, all the tags will also be automatically removed.
```bash
docker image rm [OPTIONS] IMAGE [IMAGE...]
```
Remove multiple image in single command
```bash
$ docker image rm redis:latest alpine:latest patelhimanshu/ubuntu_apache:latest 
```

1. Choose an existing image and add multiple tags to it. Notice all the tags have the same image ID of 54c9d81cbb44.
```bash
 $ docker image ls
REPOSITORY      TAG       IMAGE ID       CREATED        SIZE
ubuntu_apache   latest    7ec2fb44623c   12 hours ago   220MB
ubuntu          latest    54c9d81cbb44   4 months ago   72.8MB

 $ docker image tag ubuntu:latest ubuntu:tag1
 $ docker image tag ubuntu:latest ubuntu:tag2

 $ docker image ls
REPOSITORY      TAG       IMAGE ID       CREATED        SIZE
ubuntu_apache   latest    7ec2fb44623c   12 hours ago   220MB
ubuntu          latest    54c9d81cbb44   3 days ago     72.8MB
ubuntu          tag1      54c9d81cbb44   3 days ago     72.8MB
ubuntu          tag2      54c9d81cbb44   3 days ago     72.8MB
```
2. Trying to delete any of the image having tags.
```bash
 $ docker image rm 54c9d81cbb44
Error response from daemon: conflict: 
unable to delete 54c9d81cbb44 (must be forced) - image is referenced in multiple repositories
```
3. Either remove all tagged images first then the base image or remove all images using iamge id and forcing delete.
```bash
 $ docker image rm ubuntu:tag2
Untagged: ubuntu:tag2

 $ docker image ls
REPOSITORY      TAG       IMAGE ID       CREATED        SIZE
ubuntu_apache   latest    7ec2fb44623c   13 hours ago   220MB
ubuntu          latest    54c9d81cbb44   3 days ago     72.8MB
ubuntu          tag1      54c9d81cbb44   3 days ago     72.8MB

 $ docker image rm 54c9d81cbb44 -f
Untagged: ubuntu:latest
Untagged: ubuntu:tag1
Untagged: ubuntu@sha256:669e010b58baf5beb2836b253c1fd5768333f0d1dbcb834f7c07a4dc93f474be
Deleted: sha256:54c9d81cbb440897908abdcaa98674db83444636c300170cfd211e40a66f704f
Deleted: sha256:36ffdceb4c77bf34325fb695e64ea447f688797f2f1e3af224c29593310578d2
```
If for any reason you want to remove all containers and images, then you can use the following commands:

```bash
# To stop all containers, use the following command:
$ docker container stop $(docker container ls -q)
# To delete all containers, use the following command:
$ docker container rm $(docker container ls -a -q)
# To delete all images, use the following command:
$ docker image rm $(docker image ls -q)
```

## Exporting an image
The docker image save command lets you save or export images as tarballs.
```bash
docker image save [-o|--output]=file.tar IMAGE [IMAGE...]
```
```bash
$ docker image save --output=ubuntu_tarfile.tar ubuntu
$ ls
ubuntu_tarfile.tar
```
You can also export the container's filesystem using the following command:
```bash
$ docker container export --output=ubuntu_tarfile.tar ubuntu
```

## Importing an image
To get a local copy of the image, we either need to pull it from the accessible registry or import it from the already exported image.
```bash
docker image import [OPTIONS] file|URL|- [REPOSITORY[:TAG]]
```
Before you begin, you need a local copy of the exported Docker image.
```bash
 $ docker image ls
REPOSITORY      TAG       IMAGE ID       CREATED        SIZE
ubuntu_apache   latest    7ec2fb44623c   14 hours ago   220MB
$
$ docker image import ubuntu_tarfile.tar ubuntu:imported
sha256:4f11de2bb6442eb7a7cffab9ecbb30cdf8b63f65bf1ef2fa2f9e42b424c3578f
 $
 $ docker image ls
REPOSITORY      TAG        IMAGE ID       CREATED         SIZE
ubuntu          imported   4f11de2bb644   8 seconds ago   75.2MB
ubuntu_apache   latest     7ec2fb44623c   14 hours ago    220MB
```
You can also import TAR files stored in a remote location by specifying their URL.

## Building an image using a Dockerfile
The Dockerfile is a text-based build instruction file that enables us to define the content of the Docker image and automate image creation. The Docker build engine reads the instruction in the Dockerfile line by line and constructs the image as prescribed. The images created using Dockerfiles are considered **immutable**.

Dockerfile
```bash
FROM ubuntu
LABEL maintainer="Himanshu Patel"
CMD date
```
build the image from Dockerfile. `-t` in the command specifies the image name,
```bash
 $ docker image build -t demo_image .
Sending build context to Docker daemon  75.18MB
Step 1/3 : FROM ubuntu
 ---> 54c9d81cbb44
Step 2/3 : LABEL maintainer="Himanshu Patel"
 ---> Running in f40e7e5a862b
Removing intermediate container f40e7e5a862b
 ---> c296e9770cdb
Step 3/3 : CMD date
 ---> Running in abb5838347a1
Removing intermediate container abb5838347a1
 ---> fecae3dde252
Successfully built fecae3dde252
Successfully tagged demo_image:latest
 $ 
 $ docker image ls
REPOSITORY      TAG       IMAGE ID       CREATED         SIZE
demo_image      latest    fecae3dde252   3 seconds ago   72.8MB
ubuntu_apache   latest    7ec2fb44623c   17 hours ago    220MB
ubuntu          latest    54c9d81cbb44   3 days ago      72.8MB
```
If there is a file named `.dockerignore` in the current working directory with the list of files and directories (new-line separated), then those files and directories will be ignored in the build context.  

Now, the Docker build system will read each instruction in the Dockerfile, launch an intermediate container, execute the instruction inside the container or update the metadata, commit the intermediate container as an image layer, and remove the intermediate container. This process is continued until all the instructions in the Dockerfile are executed.

### Format for Dockerfile

#### FROM

`FROM`: This must be the first instruction of any Dockerfile, and sets the base image for subsequent instructions. By default, the latest tag is assumed to be the following:
```dockerfile
FROM <image>
```
OR
```dockerfile
FROM <images>:<tag>
```

There can be more than one FROM instruction in one Dockerfile to create multiple images. If only the image names, such as fedora and Ubuntu, are given, then the images will be downloaded from the default Docker registry (Docker Hub). If you want to use private or third-party images, then you have to include them as follows:  

`[registry_hostname[:port]/][user_name/](repository_name:version_tag)`  

The following is an example using the preceding syntax:
```Dockerfile
FROM registry-host:5000/cookbook/apache2
FROM <images>:<tag> AS <build stage>
```
You can also prescribe a build stage to be used for multistage image building.

#### RUN
`RUN`: We can execute the RUN instruction in two ways. First, we can run it in the `shell (sh -c)`:
```dockerfile
RUN <command> <param1> ... <pamamN>
```
Second, we can directly run an executable:
```dockerfile
RUN ["executable", "param1",...,"paramN" ]
```
As we know, with Docker we create an `overlay`(a layer on top of another layer) to make the resulting image. Through each `RUN` instruction, we create and commit a layer on top of the earlier committed layer. A container can be
started from any of the committed layers.

By default, Docker tries to cache the layers committed by different `RUN` instructions so that they can be used in subsequent builds. However, this behavior can be turned off using `--no-cache` flag while building the image.

#### LABEL
`LABEL`: To give a label to an image, we use the `LABEL` instruction in the Dockerfile, 
```dockerfile
LABEL distro=ubuntu.
```

#### CMD
`CMD`: The `CMD` instruction provides a default executable while starting a container. If the `CMD` instruction does not have any executable (parameter 2), then it will provide arguments to `ENTRYPOINT`:
```dockerfile
CMD ["executable", "param1",...,"paramN" ]
CMD ["param1", ... , "paramN"]
CMD <command> <param1> ... <pamamN>
```
Only one `CMD` instruction is allowed in Dockerfile. If more than one is specified, then only the last one will be honored.

#### ENTRYPOINT
`ENTRYPOINT`: This helps us configure the container as an executable. There can be a maximum of one instruction for `ENTRYPOINT`; if more then one is specified, then only the last one will be honored: 
```dockerfile
ENTRYPOINT ["executable", "param1",...,"paramN" ]
ENTRYPOINT <command> <param1> ... <pamamN>
```
Once the parameters are defined with the `ENTRYPOINT` instruction, they cannot be overwritten at runtime. However, `ENTRYPOINT` can be used as `CMD`, if we want to use different parameters for `ENTRYPOINT`.

#### EXPOSE
`EXPOSE`: This exposes network ports on the container, on which it will listen at runtime:
```dockerfile
EXPOSE <port> [<port> ... ]
```
We can also expose a port while starting the container.

#### ENV
`ENV`: This will set the environment variable `<key>` to `<value>`. It will be passed to all the subsequent instructions and will persist when a container is run from the resulting image:
```dockerfile
ENV <key> <value>
```

### ADD
`ADD`: This copies files from the source to the destination:
```dockerfile
ADD <src> <dest>
```
The following ADD instruction is for a path containing white spaces:
```dockerfile
ADD ["<src>"... "<dest>"]
```
`<src>`: This must be the file or directory inside the build directory from which we are building an image, which is also called the context of the build. A source can be a remote URL as well.
`<dest>`: This must be the absolute path inside the container in which files/directories from the source will be copied.

#### COPY
COPY: This is similar to ADD. 
```dockerfile
COPY <src> <dest>
COPY ["<src>"... "<dest>"]
```
The `COPY` instruction optionally supports a `--from` option for multistage building.

#### VOLUME
`VOLUME`: This instruction will create a mount point with the given name and flag it as mounting the external volume using the following syntax:
```dockerfile
VOLUME ["/data"]
```
Alternatively, you can use the following code:
```dockerfile
VOLUME /data
```

#### USER
`USER`: This sets the username for any of the following run instructions using the following syntax:
```dockerfile
USER <username>/<UID>
```

#### WORKDIR
`WORKDIR`: This sets the working directory for any `RUN`, `CMD`, and `ENTRYPOINT` instructions that follow it. It can have multiple entries in the same Dockerfile. A relative path can be given that will be relative to the earlier `WORKDIR` instruction using the following syntax:
```dockerfile
WORKDIR <PATH>
```

#### ONBUILD
`ONBUILD`: This adds trigger instructions to the image that will be executed later, when this image will be used as the base image for another image. This trigger will run as part of the FROM instruction in a downstream Dockerfile using the
following syntax:
```dockerfile
ONBUILD [INSTRUCTION]
```

## Building an Apache image – a Dockerfile example
We will build a very simple Docker image that bundles the apache2 web server and also adds metadata to launch the apache2 application inside the container whenever a new container is created from this image.

Dockerfile is as below
```Dockerfile
FROM alpine:3.6
LABEL maintainer="Himanshu Patel <py.himanshu.patel@gmail.com>"
RUN apk add --no-cache apache2 \
	&& mkdir -p /run/apache2 && \
	echo "<html><h1>Hello Docker</h1></html>" > /var/www/localhost/htdocs/index.html
EXPOSE 80
ENTRYPOINT [ "/usr/bin/httpd", "-D", "FOREGROUND" ]
```

```bash
$ docker image build -t apache2 .
```

## Setting up a private index/registry
Earlier, we used a Docker-hosted registry (https:/​/​hub.​docker.​com) to push and pull images. Nonetheless, quite often you will come across use cases where you will have to host a private registry in your infrastructure. In this recipe, we will host our own private registry using Docker's `registry:2` image.

1. Let's begin by launching a local registry on the container using the following command:
```bash
docker container run -d -p 5000:5000 --name registry registry:2
```
The preceding command to pull the image will download the official registry image from Docker Hub and run it on port `5000`. The -p option publishes the container port to the host system's port.
2. To push the image to the local registry, you need to prefix the repository name with the `localhost` hostname or the IP address `127.0.0.1` and the registry port `5000` using the Docker image tag command, as shown in the following code:
```bash
$ docker tag apache2 localhost:5000/apache2

 $ docker image ls
REPOSITORY               TAG       IMAGE ID       CREATED        SIZE
apache2                  latest    0927a33d5db5   4 hours ago    7.25MB
localhost:5000/apache2   latest    0927a33d5db5   4 hours ago    7.25MB
```
3. Now, let's push the image to the local registry using the `docker image push` command.
```bash
 $ docker image push localhost:5000/apache2
Using default tag: latest
The push refers to repository [localhost:5000/apache2]
642a413b8427: Pushed 
721384ec99e5: Pushed 
latest: digest: sha256:fc514c5e2f6a6d8798606ad6c9b8f0fd3d51fc161616137af8097e17b68de07e size: 739
```

## Creating a custom base image
Install debootstrap on any Debian-based system using the following command:
```bash
$ apt-get install debootstrap
```

1. Create the directory on which you want to populate all the distribution files:
```bash
$ mkdir xenial
```
2. Now, using debootstrap, install Xenial Xerus inside the directory we created earlier:
```bash
$ sudo debootstrap xenial ./xenial
```
You will see the directory tree, similar to any Linux root filesystem, inside the directory in which Xenial Xerus is installed:
```bash
$ ls ./xenial 
bin boot dev etc home lib lib64 root run sbin srv sys tmp usr var
```
3. Now, we can export the directory as a Docker image with following command:
```bash
$ sudo tar -C xenial/ -c . | docker image import - xenial
```
4. Look at the `docker image ls` output. You should have a new image with xenial as the name.
```bash
 $ docker image ls
REPOSITORY               TAG       IMAGE ID       CREATED          SIZE
xenial                   latest    9e3ec58474bc   24 minutes ago   230MB
localhost:5000/apache2   latest    0927a33d5db5   11 hours ago     7.25MB
apache2                  latest    0927a33d5db5   11 hours ago     7.25MB
```

The debootstrap command pulls all the Ubuntu 18.04 (Xenial Xerus) packages to the directory from the package repository. Then they are bundled as a TAR file and pushed to the Docker image import command to create the Docker image.

## Creating a minimal image using a scratch base image

In the previous recipe, we custom-created a base image without any parent image. However, that image is bloated with all the binaries and libraries that are shipped with the Ubuntu 18.04 distribution. Typically, to run an application, we don't need the majority of the binaries and libraries we have bundled in the image. Besides, it leaves a large image footprint, and thus becomes a portability problem. To overcome this issue, you can diligently hand-pick the binaries and libraries that will constitute your image and then
bundle the Docker image. Alternatively, you can build using Docker's reserved image, called a scratch image. This scratch image is explicitly an empty image, and it does not add any additional layer to your image. Furthermore, unlike the previous recipe, you can automate image creation using a Dockerfile.

1. Ensure that the Docker daemon is running and has access to the `gcc` and `scratch` images.
2. The content of `demo.c` should be as follows:
```c
#include <stdio.h>
void main(){
	printf("Statically Built");
}
```
3. The content of the Dockerfile should be as follows:
```dockerfile
FROM scratch
ADD demo /
CMD ["/demo"]
```
4. Now, build a static executable demo from the `demo.c` file using the `gcc:7.2` runtime container, as shown in the following code:
```bash
$ docker container run --rm -v ${PWD}:/src -w /src gcc:7.2 gcc -static -o demo demo.c
```
5. Continue to build an image from the scratch base image with the executable demo created in the previous step.
```bash
 $ docker image build -t scratch-demo .
Sending build context to Docker daemon  953.3kB
Step 1/3 : FROM scratch
 ---> 
Step 2/3 : ADD demo /
 ---> 5210ae36bbc7
Step 3/3 : CMD ["/demo"]
 ---> Running in a14b78394eab
Removing intermediate container a14b78394eab
 ---> dff92f87457b
Successfully built dff92f87457b
Successfully tagged scratch-demo:latest
```
6. Finally, let us verify the image by spinning a container from the preceding image and checking the image size.
```bash
 $ docker image ls
REPOSITORY               TAG       IMAGE ID       CREATED         SIZE
scratch-demo             latest    dff92f87457b   6 seconds ago   949kB
xenial                   latest    9e3ec58474bc   2 hours ago     230MB
gcc                      7.2       81ffb25b1dec   4 years ago     1.64GB
```
```bash
$ docker container run --rm scratch-demo
Statically Built
```

The Docker build system intuitively understands the reserved image name scratch in the `FROM` instruction and starts bundling the image without any additional layer for the base image. So, in this recipe, the Docker build system just bundled the statically linked executable demo and the metadata for the image.
As mentioned earlier, the scratch image doesn't add any additional layer to the image. Check using `docker image history` command.
```bash
 $ docker image history scratch-demo:latest 
IMAGE          CREATED         CREATED BY                                      SIZE      COMMENT
dff92f87457b   4 minutes ago   /bin/sh -c #(nop)  CMD ["/demo"]                0B        
5210ae36bbc7   4 minutes ago   /bin/sh -c #(nop) ADD file:cf3f86051df51f238…   949kB 
```

## Building images in multiple stages
In the previous recipe, we created a statically linked executable using the gcc builder container and then bundled the executable using the scratch image. Build pipelines with the builder pattern are very common because, during the build time, you will need heavyweight building and supporting tools. However, the resulting artifacts usually don't need those tools during execution time. Therefore, the artifacts are usually built using the
appropriate runtime with additional capability and then the resulting artifacts are packaged with the runtime just enough to run the artifacts. Though this solution works very well, the complexity of this build pipeline is managed outside the Docker ecosystem through scripts.

Docker's multistage build enables us to orchestrate complex building stages in a single Dockerfile. In the Dockerfile, we can define one or more intermediate stages with the appropriate parent image and the build ecosystem to build the artifact.

1. The content of `src/app.c` should be as follows:
```c
#include <stdio.h>
void main(){
	printf("This is a Docker multistage build \n");
}
```
2. The content of `Dockerfile` should be as follows:
```bash
FROM gcc:7.2 AS builder
COPY src /src
RUN gcc -static -o /src/app /src/app.c && strip -R .comment -s /src/app
FROM scratch
COPY --from=builder /src/app .
CMD ["./app"]
```
3. Build docker image 
```bash
 $ docker build -t multistage .
Sending build context to Docker daemon  3.584kB
Step 1/6 : FROM gcc:7.2 AS builder
 ---> 81ffb25b1dec
Step 2/6 : COPY src /src
 ---> 2ad191e45fa7
Step 3/6 : RUN gcc -static -o /src/app /src/app.c && strip -R .comment -s /src/app
 ---> Running in cd75cdf33c6f
Removing intermediate container cd75cdf33c6f
 ---> 2e2ca3933e98
Step 4/6 : FROM scratch
 ---> 
Step 5/6 : COPY --from=builder /src/app .
 ---> f725f984abad
Step 6/6 : CMD ["./app"]
 ---> Running in 363481b10244
Removing intermediate container 363481b10244
 ---> 0740d53c92ae
Successfully built 0740d53c92ae
Successfully tagged multistage:latest
```
```bash
 $ docker image ls
REPOSITORY               TAG       IMAGE ID       CREATED          SIZE
multistage               latest    0740d53c92ae   2 minutes ago    738kB
scratch-demo             latest    dff92f87457b   49 minutes ago   949kB
xenial                   latest    9e3ec58474bc   3 hours ago      230MB
```
4. Make a container
```bash
 $ docker run --rm multistage
This is a Docker multistage build
```
