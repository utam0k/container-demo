# container-demo

[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/utam0k/container-demo)

# chroot

```sh
$ ROOTFS=$(mktemp -d)
$ echo "Hello" > $ROOTFS/test.txt
$ mkdir -p $ROOTFS/{bin,/lib/x86_64-linux-gnu,lib64}
$ cp -v /bin/{bash,cat} $ROOTFS/bin
$ ldd /bin/cat | grep -o '/lib.*\.[0-9]' | xargs -I{} cp -v {} $ROOTFS/{} 
$ ldd /bin/bash | grep -o '/lib.*\.[0-9]' | xargs -I{} cp -v {} $ROOTFS/{} 
$ sudo chroot $ROOTFS bash
bash-5.0# cat test.txt
Hello
bash-5.0# exit
exit
```

# make a container-like

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