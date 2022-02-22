# Docker Security

## Introduction
Docker containers, actually, are not Sandbox applications, which means they are not recommended to run random applications on the system as `root` with Docker. You should always treat a container running a service/process as a service/process running on the host system, and put all the security measures inside the container you put on the host system.

The six namespaces that Docker uses are **Process**, **Network**, **Mount**, **Hostname**,
**Shared Memory**, and **User**. Not everything in Linux is namespaced, for example, **SELinux**,
**Cgroups**, **Devices** (/dev/mem, /dev/sd*), and **Kernel Modules**. Filesystems under /sys,
/proc/sys, /proc/sysrq-trigger, /proc/irq, and /proc/bus are also not namespaced, but they are mounted as read-only by default with the container runtime.

- We can also consider turning off the default inter-container communication over the network with `--icc=false` on the Docker host; though containers can still communicate through links, which overrides the default DROP policy of iptables, they get set with the `--icc=false` option.
- We can also set `Cgroups` resource restrictions, through which we can prevent Denial of Service (DoS) attacks through system resource constraints.
- Docker takes advantage of the special device, `Cgroups`, that allows us to specify which device nodes can be used within the container. It blocks processes from creating and using device nodes that could be used to attack the host.

#### The following are some guidelines (they may not be complete) you can follow to achieve a secure Docker environment:

- Run services as non-root and treat the root in the container, as well as outside the container, as root.
- Use images from trusted parties to run the container; avoid using the `-insecure-registry=[]` option.
- Don't run the random container from the Docker registry or anywhere else.
- Have your host kernel up to date.
- Avoid using `--privileged` whenever possible, and drop container privileges as soon as possible.
- Configure Mandatory Access Control (MAC) through `SELinux` or `AppArmor`.
- Collect logs for auditing.
- Do regular auditing.
- Run containers on hosts, which are specially designed to run containers. Consider using Project Atomic, CoreOS, or similar solutions.
- Mount devices with the `--device` option rather than using the `--privileged` option to use devices inside the container.
- Prohibit SUID and SGID inside the container.

#### Let's use that default installation to try an experiment:

1. Disable **SELinux** using the following command:
```bash
 $ sudo setenforce 0
setenforce: SELinux is disabled
```
2. Create a user and add it to the default Docker group so that the user can run docker commands without sudo:
```bash
# adding user 
 $ sudo useradd docker_user
[sudo] password for himanshu: 
 $ less /etc/passwd | grep docker
docker_user:x:1001:1001::/home/docker_user:/bin/sh

# adding group
 $ groups
himanshu adm cdrom sudo dip plugdev lpadmin lxd sambashare vboxusers docker
 $ sudo groupadd docker
groupadd: group 'docker' already exists

# add user to group
 $ sudo gpasswd -a docker_user docker
Adding user docker_user to group docker

# show group
 $ cat /etc/group | grep docker
docker:x:998:himanshu,docker_user
docker_user:x:1001:
```
3. Log in using the user we created earlier, and start a container as follows:
```bash
 $ su - docker_user
Password: 
su: warning: cannot change directory to /home/docker_user: No such file or directory
$ whoami
docker_user
$ docker container run -it -v /:/host alpine ash
/ # chroot /host/
root@f13f14bb042c:/# 
/ #
```
4. From the container, `chroot` to `/host` and run the shutdown command:
```bash
/ # chroot /host/
groups: cannot find name for group ID 11
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.
root@bf440fe99409:/# shutdown
Failed to set wall message, ignoring: Access denied
Failed to call ScheduleShutdown in logind, no action will be taken: 
Transport endpoint is not connected
```

## Setting Mandatory Access Control (MAC) with SELinux
It is recommended that you set up some form of MAC on the Docker host, either through SELinux or AppArmor, depending on the Linux distribution.

- SELinux is a labeling system
- Every process has a label
- Every file, directory, and system object has a label
- Policy rules control access between labeled processes and labeled objects
- The kernel enforces the rules

- `Type enforcement`: This is used to protect the host system from container processes. Each container process is labeled `svirt_lxc_net_t`, and each container file is labeled `svirt_sandbox_file_t`. The `svirt_lxc_net_t` type is allowed to manage any content labeled with `svirt_sandbox_file_t`. Container processes can only access/write container files.

- `Multi Category Security enforcement`: By setting type enforcement, all container processes will run with the `svirt_lxc_net_t` label, and all content will be labeled with `svirt_sandbox_file_t`. However, with just these settings, we are not protecting one container from another because their labels are the same.

We use `Multi Category Security (MCS)` enforcement to protect one container from another, which is based on `Multi Level Security (MLS)`. When a container is launched, the Docker daemon picks a random MCS label, for example, `s0:c41,c717` and saves it with the container metadata. When any container process starts, the Docker daemon tells the kernel to apply the correct MCS label. As the MCS label is saved in the metadata, if the container restarts, it gets the same MCS label.

Refer Book for more.
