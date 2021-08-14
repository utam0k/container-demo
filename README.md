# container-demo
This repository is a demo that allows you to experience the technology bihind container.
You will need **Linux** to try this demo, however you can try it in your browser by using the [gitpod](https://gitpod.io/#https://github.com/utam0k/container-demo) we provide.

[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/utam0k/container-demo)

# chroot

```
$ ROOTFS=$(mktemp -d)
$ echo "Hello" > $ROOTFS/test.txt
$ mkdir -p $ROOTFS/{bin,/lib/x86_64-linux-gnu,lib64}
$ cp -v /bin/{bash,cat} $ROOTFS/bin
$ ldd /bin/cat | grep -o '/lib.*\.[0-9]' | xargs -I{} cp -v {} $ROOTFS/{} 
$ ldd /bin/bash | grep -o '/lib.*\.[0-9]' | xargs -I{} cp -v {} $ROOTFS/{} 
$ sudo chroot $ROOTFS bash
bash-5.0# cat /test.txt
Hello
bash-5.0# exit
exit
```

# make a container-like

```
$ cd $(mktemp -d)
$ mkdir rootfs
$ docker export $(docker create busybox) | tar -C rootfs -xvf -
$ sudo unshare -m -u -n -i -p -f --mount-proc sh
# ps -ef
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 11:50 pts/3    00:00:00 sh
root           2       1  0 11:50 pts/3    00:00:00 ps -ef
```

# join an existing container

```
$ docker run -d --rm --name sample-container alpine sleep 1d
$ PID=$(docker inspect --format {{.State.Pid}} sample-container)
$ sudo nsenter --target $PID --uts --mount --ipc --net --pid -- /bin/sh
/ # ps -ef
PID   USER     TIME  COMMAND
    1 root      0:00 sh
   26 root      0:00 /bin/sh
   27 root      0:00 ps
/ # exit
```

⚠️ If you encounter the following error in gitpod, try to recreate the gitpod workspace.
```
docker: Error response from daemon: OCI runtime create failed
```