# container-demo
This repository is a demo that allows you to experience the technology bihind container.
You will need **Linux** to try this demo, however you can try it in your browser by using the [gitpod](https://gitpod.io/#https://github.com/utam0k/container-demo) we provide.  

Technology you can experience.
- [chroot](https://man7.org/linux/man-pages/man1/chroot.1.html) - Change the `/` directory
- [unshare](https://man7.org/linux/man-pages/man1/unshare.1.html) - Create a new [namepsace]((https://man7.org/linux/man-pages/man7/namespaces.7.html)) and run a command

Enjoy the container technology 🥰

[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/utam0k/container-demo)

## Target audience
The target audience is people who have been exposed to containers such as docker, podman, and K8s, but want to know more about what containers are.

# chroot
In this chart, we will try to change `/` using `chroot`.
We use `chroot` to simplify the story, but in reality, `pivot_root` is used.

Set up an experimental directory that will be `/` and a trial file in it.
```
$ ROOTFS=$(mktemp -d)
$ echo "Hello" > $ROOTFS/test.txt
```

Prepare to run the bash and cat commands. Since the `/` will be changed and the libraries cannot be linked, copy them with the `cp` command.
```
$ mkdir -p $ROOTFS/{bin,/lib/x86_64-linux-gnu,lib64}
$ cp -v /bin/{bash,cat} $ROOTFS/bin
$ ldd /bin/cat | grep -o '/lib.*\.[0-9]' | xargs -I{} cp -v {} $ROOTFS/{} 
$ ldd /bin/bash | grep -o '/lib.*\.[0-9]' | xargs -I{} cp -v {} $ROOTFS/{} 
```

Finally, it's time to `chroot`. Change `/` to the `$ROOTFS` directory you created first, and run bash. You can see that `/` has been changed because the first `test.txt` is in `/`.
```
$ sudo chroot $ROOTFS bash
bash-5.0# cat /test.txt
Hello
bash-5.0# exit
exit
```

## Reference
- [chroot](https://man7.org/linux/man-pages/man1/chroot.1.html)
- [pivot_root](https://man7.org/linux/man-pages/man2/pivot_root.2.html)

## Futher challenge
- Try `pivot_root`

# make a container-like
In this chapter, we will use `unshare` command to create a namespace and the technique of changing `/` introduced in the previous chapter to create something container-like.
Namespace is a technique for isolating manipulable resources. For example, it isolates the space of a process.

Create the `/` directory where the container will work as in the previous chapter.
In this article, we will use `busybox` to create a container.
```
$ cd $(mktemp -d)
$ mkdir rootfs
$ docker export $(docker create busybox) | tar -C rootfs -xvf -
```

Now, we will use the unshare command to create a container. The options are roughly as follows.
- `--mount` `--uts` `--ipc` `--pid`   
    Isolate each namespace. For example, pid separates the process space. This can be experienced by using the `ps` command, where the source and visible processes are different.
- `--fork`   
    When executing a command, `fork` is used. This is because the pid namespace will be applied from the new process. This is because the pid namespace will be applied from the new process. You don't need to understand this at first.
- `--mount-proc`  
    This is the magic that allows the `ps` command to run. More precisely, it is an option to mount procfs so that `ps` commands can be executed.
- `--root rootfs`  
    We will use the `rootfs` created in the first part as `/`. You saw this in the previous chapter where we explained the technique to use.

```
$ sudo unshare --mount --uts --ipc --pid --fork --mount-proc --root rootfs sh
# ps -ef
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 11:50 pts/3    00:00:00 sh
root           2       1  0 11:50 pts/3    00:00:00 ps -ef
```

## Reference
- [unshare](https://man7.org/linux/man-pages/man1/unshare.1.html)
- [namespace](https://man7.org/linux/man-pages/man7/namespaces.7.html)
- [setns](https://man7.org/linux/man-pages/man2/setns.2.html)

## Futher challenge
- Try to make sure that other namespaces are isolated
- Reproduce the same environment as the demo without using the `--fork`, `--mount-proc` and `--root rootfs` options.
 

# join an existing container

In this chapter, we will experience joining an existing container.
Joining means joining the namespace of the target container.
This is similar to the behavior of `docker exec`.

First, create a container and get the pid of the container.
```
$ docker run -d --rm --name sample-container alpine sleep 1d
$ PID=$(docker inspect --format {{.State.Pid}} sample-container)
```

Next, use the nsenter command to join the container you have just created.
```
$ sudo nsenter --target $PID --uts --mount --ipc --net --pid /bin/sh
```

If you run the `ps` command in the joined container, you will see that the process space has been separated. In addition, you can run it with `docker run` first to see the processes of the `sleep` command.

```
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

## Reference
- [nsenter](https://man7.org/linux/man-pages/man1/nsenter.1.html)
- [setns](https://man7.org/linux/man-pages/man2/setns.2.html)

## Futher challenge
- Try to make sure that other namespaces are isolated