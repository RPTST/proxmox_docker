# proxmox_docker
Let’s Install Docker on Proxmox, with ZFS, and good performance!

We are installing docker-ce on the host, and Portainer 1 to help manage containers.

This is for dev only. I think. See, you really have to have a deep understanding of all the systems involved to understand what sort of risks you’re opening up yourself to by going off script. LXC containers, by default (imho) have better host isolation than Docker, for example.

And Docker inside a Virtual Machine or LXC container can’t really get at some of the ZFS features it would be nice to be able to access from inside the container.

https://docs.docker.com/storage/storagedriver/zfs-driver/

So on my setup I have the default rpool that Proxmox gives you, and I have added ssdpool at /ssdpool.

I am adding my docker storage on the SSDs for now.

    $zfs create -o mountpoint=/var/lib/docker ssdpool/docker-root
    $zfs create -o mountpoint=/var/lib/docker/volumes ssdpool/docker-volumes 

zfs list will show you the mounted ZFS stuff around your filesystem. It’s nice.

We’ll want to disable auto-snapshotting on the root vut enable it on the volumes:

    $zfs set com.sun:auto-snapshot=false ssdpool/docker-root 
    $zfs set com.sun:auto-snapshot=true ssdpool/docker-volumes

And finally, according to the ZFS storage driver doc above we need to add the storage driver as ZFS. You can also set a quota

edit or create /etc/docker/daemon.json

    $nano /etc/docker/daemon.json
    {   
         "storage-driver": "zfs"
    } 

Actually Setup Docker

Now we can install docker CE. Follow their docs, or cheatsheet:

    $apt install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
    $curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -

create etc/apt/sources.list.d/docker.list

    deb [arch=amd64] https://download.docker.com/linux/debian buster stable

Finally, install amd test

    apt update
    apt install docker-ce docker-ce-cli containerd.io

    $docker run hello-world 
