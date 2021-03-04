# proxmox_docker
Let’s Install Docker on Proxmox, with ZFS, and good performance!

We are installing docker-ce on the host, and Portainer to help manage containers.

Docker inside a Virtual Machine or LXC container can’t really take advantage of some of the ZFS features. 

# See the official documentation
https://docs.docker.com/storage/storagedriver/zfs-driver/

Using the default rpool that Proxmox gives you, and setting up/adding a ssdpool at /ssdpool.

Adding the docker storage on the SSDs for now.

    $zfs create -o mountpoint=/var/lib/docker ssdpool/docker-root
    $zfs create -o mountpoint=/var/lib/docker/volumes ssdpool/docker-volumes 

zfs list will show you the mounted ZFS stuff around your filesystem.

Disable auto-snapshotting on the root but enable it on the volumes:

    $zfs set com.sun:auto-snapshot=false ssdpool/docker-root 
    $zfs set com.sun:auto-snapshot=true ssdpool/docker-volumes

Finally, according to the ZFS storage driver doc above we need to add the storage driver as ZFS. You can also set a quota

# edit or create /etc/docker/daemon.json

    $nano /etc/docker/daemon.json
    {   
         "storage-driver": "zfs"
    } 

# Setting up Docker

Now we can install docker CE. Follow their docs, or cheatsheet:

    $apt install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
    $curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -

create etc/apt/sources.list.d/docker.list

    deb [arch=amd64] https://download.docker.com/linux/debian buster stable

Finally, install amd test

    apt update
    apt install docker-ce docker-ce-cli containerd.io

    $docker run hello-world 

# Portainer

    zfs create ssdpool/docker-volumes/portainer_data
    docker volume create portainer_data
    docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce

Go to http://yourip.example.com:9000 and set a password
