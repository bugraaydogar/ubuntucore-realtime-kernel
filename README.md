# ubuntucore-realtime-kernel

A blog post is also available at 
https://bugraaydogar.medium.com/real-time-kernel-on-ubuntu-core-70f2e41c1080

## Purpose & Description

This repository is created to demonstrate how to build a customer kernel snap with PREEMPT_RT patch.
Also it can be used as a reference to create custom Linux kernel snap.


## Prerequisites
- It is assumed that you have an idea about the Ubuntu Core and snaps.
- It is assumed that you already know how to build a Linux kernel.
- It is assumed that you are using Ubuntu 18.04 or later ( preferably 20.04 )
- It is assumed that you already installed LXD in your system. If not, you can find the details here.

## Choosing the Kernel and RT Patch Versions

Ubuntu forks the mainline kernel, applies lots of security and performance patches and renames it and according to their own labelling. 
Therefore, it is crucial to understand the Ubuntu Kernel version and the corresponding mainline kernel version. 
Here is the web page that enlightens the mystery;
https://people.canonical.com/~kernel/info/kernel-version-map.html

I have selected Ubuntu-5.4.0–65.73 as reference since it is the latest version and there is a RT patch available for it here. 
To clarify it, Ubuntu-5.4.0–65.73 is based on Linux Kernel 5.4.78 so, you have to choose corresponding RT patch which is 5.4.78-rt44.

## Building

- install lxd and create a focal (20.04) lxd container

    ```
    $ sudo snap install lxd
    $ sudo lxd init --auto
    $ sudo lxc launch ubuntu:20.04 focal
    ```

- enter the container and install snapcraft

    ```
    $ sudo lxc shell focal
    # snap install snapcraft --classic
    ```

- clone this branch and build it

    ```
    # git clone https://github.com/bugraaydogar/ubuntucore-realtime-kernel.git
    # cd ubuntucore-realtime-kernel
    # snapcraft --destructive-mode
    # exit
    $
    ```

- pull the file from the container

    ```
    $ lxc file pull focal/root/ubuntucore-realtime-kernel/yourcustomkernel.snap .
    ```

## Creating Ubuntu Core 18 Image


There is a great tutorial about how to create Ubuntu Core 18 image. 
I would definitely suggest reading it here since I won’t describe the basic steps here.

https://snapcraft.io/tutorials/create-your-own-core-18-image#1-overview

Once you have created your model.json and sign it, it is time to build the image with the ubuntu-image utility.
Here is the trick, it is possible to replace snaps that are delivered by Canonical by using the “extra-snaps” parameters. 
Here it is;
```
$ ubuntu-image snap --extra-snaps my-custom-kernel.snap my-model.model
```

Booting on Qemu

```
qemu-system-x86_64 -enable-kvm -smp 2 -m 1500 -netdev user,id=mynet0,hostfwd=tcp::8022-:22,hostfwd=tcp::8090-:80 -device virtio-net-pci,netdev=mynet0 -drive file=pc.img,format=raw

```


