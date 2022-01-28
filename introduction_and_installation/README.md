# Introduction and Installation

## Namespaces

Namespaces are the building blocks of a container. There are different types of namespace,
and each one of them isolates applications from the others.

#### The PID namespace

The PID namespace allows each container to have its own process numbering. Each PID forms its own process hierarchy. A parent namespace can see the children namespaces and affect them, but a child can neither see the parent namespace nor affect it.

If there are two levels of hierarchy, then at the top level, we would see the process running inside the child namespace with a different PID. So a process running in a child namespace would have two PIDs: one in the child namespace and the other in the parent namespace.

On container PID is 31

```bash
/ # ps aux | grep -w sh
    1 root      0:00 /bin/sh
   31 root      0:00 sh
   46 root      0:00 grep -w sh
```

On host PID is 11365

```bash
~ $ ps aux | grep -w sh
root       10280  0.0  0.0   1692  1012 pts/0    Ss+  22:49   0:00 /bin/sh
himanshu   11347  0.0  0.5 1350528 46292 pts/2   Sl+  23:06   0:00 docker exec -it pensive_hopper sh
root       11365  0.0  0.0   1692  1164 pts/1    Ss+  23:06   0:00 sh
himanshu   11541  0.0  0.0   9280  2348 pts/1    S+   23:08   0:00 grep --color=auto -w sh
```

#### The net namespace

With the PID namespace, we can run the same program multiple times in different isolated environments; for example, we can run different instances of Apache on different containers. But without the net namespace, we would not be able to listen on port 80 on each one of them. The net namespace allows us to have different network interfaces on each container, which solves the problem I mentioned earlier. Loopback interfaces would be different in each container as well.

To enable networking in containers, we create pairs of special interfaces in two different net namespaces and allow them to talk to each other. One end of the special interface resides inside the container and the other resides on the host system. Generally, the interface inside the container is called `eth0`, and on the host system, it is given a random name, such as `veth516cc56`. These special interfaces are then linked through a bridge (`docker0`) on the host to enable communication between the containers and the route packets.

#### The IPC namespace
The inter-process communication (IPC) namespace provides semaphores, message queues, and shared memory segments. It is not widely used these days, but some programs still depend on it.
If the IPC resource created by one container is consumed by another container, then the application running on the first container could fail. With the IPC namespace, processes running in one namespace cannot access resources from another namespace.

#### The mnt namespace
With the mnt namespace, a container can have its own set of mounted filesystems and root directories. Processes in one mnt namespace cannot see the mounted filesystems of another
mnt namespace.

#### The UTS namespace
With the UTS namespace, we can have different hostnames for each container.

#### The user namespace
With user namespace support, we can have users who have a nonzero ID on the host, but who can have a zero ID inside the container. This is because the user namespace allows mappings of users and groups IDs per namespace.
There are ways to share namespaces between the host and container, and other containers as well. We'll see how to do this in subsequent chapters.


## Cgroups
Control groups (cgroups) provide resource limitations and accounting for containers. Instead of setting the resource limit to a single process, cgroups allow you to limit resources to a group of processes.

```bash
sudo apt-get install cgroup-tools
```

## Verifying requirements for Docker installation

- Docker is not supported on 32-bit architecture.
	```bash
	~ $ uname -i
	x86_64
	```
- Docker is supported on kernel 3.8 or later.
	```bash
	~ $ uname -r
	5.13.0-27-generic
	```

## Installing Docker on Ubuntu

[Refer Documentation](https://docs.docker.com/engine/install/ubuntu/)

## Useful commands

- The default Docker daemon configuration file is located at `/etc/docker`, which is used
while starting the daemon.
- To start the service, enter the following: `$ sudo systemctl start docker`
- To verify the installation, enter the following: `$ docker info`
- To update the package, enter the following: `$ sudo apt-get update`
- To enable the start of the service at boot time, enter the following: `$ sudo systemctl enable docker`
- To stop the service, enter the following: `$ sudo systemctl stop docker`

## Adding a nonroot user to administer Docker

1. Create the Docker group, if it is not there already: `$ sudo groupadd docker`
2. Create the user to whom you want to give permissions to administer Docker: `$ sudo useradd dockertest`
3. Run the following command to add the newly created user to administer Docker: `$ sudo usermod -aG docker dockertest`
