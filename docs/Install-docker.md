# Install Docker

Docker is an OS-level virtualization tool that allows us to run non-native software packages via containers.

Docker Compose is a tool for running multi-container applications on Docker defined using the Compose file format. A Compose file is used to define how one or more containers that make up your application are configured. Once you have a Compose file, you can create and start your application with a single command: docker compose up.

> Reference docs :
> - https://docs.docker.com/engine/install/linux-postinstall/
> - https://docs.docker.com/compose/gettingstarted/

## Install Docker runtime

Update the Rpi

```
sudo apt update
sudo apt upgrade -y
```

Reboot the pi to apply changes

Call the following script to install Docker CE

```
curl -sSL https://get.docker.com | sh
```

The docker installed release can be checked :

```
docker version
```

By default, only root can run containers. Add your non-root user to the Docker group which will allow it to execute docker commands.

```
sudo usermod -aG docker <username>
```

## Install Docker compose

Docker Compose V2 is a major version bump release of Docker Compose
> Reference docs : https://github.com/docker/compose#docker-compose-v2

Docker Compose binaries can be downloaded from the release page on this repository.

```
sudo curl -SL https://github.com/docker/compose/releases/download/v2.14.1/docker-compose-linux-armv7 -o /usr/local/bin/docker-compose
```

```
sudo chmod +x docker-compose
docker-compose version
```

## Enable docker at startup

Enable the Docker system service to start your containers on boot. With the following command you can configure your Raspberry Pi to automatically run the Docker system service, whenever it boots up.

```
sudo systemctl enable docker
```

## Managing Docker storage

> Reference docs : https://docs.docker.com/storage/

Volumes are stored in a part of the host filesystem which is managed by Docker (`/var/lib/docker/volumes/` on Linux). Non-Docker processes should not modify this part of the filesystem. Volumes are the best way to persist data in Docker.

A given volume can be mounted into multiple containers simultaneously. When no running container is using a volume, the volume is still available to Docker and is not removed automatically. You can remove unused volumes using docker `volume prune`.

## Prepare external drive (USB) to store Docker data

The first step is to check if the USB device was detected. A simple `lsusb` in a shell will show the connected USB devices.

```
Bus 001 Device 004: ID 0781:5583 SanDisk Corp. Ultra Fit
Bus 001 Device 003: ID 0424:ec00 Microchip Technology, Inc. (formerly SMSC) SMSC9512/9514 Fast Ethernet Adapter
Bus 001 Device 002: ID 0424:9514 Microchip Technology, Inc. (formerly SMSC) SMC9514 Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

Check existing partition with the `lsblk -f` command :

```
NAME        FSTYPE FSVER LABEL  UUID                                 FSAVAIL FSUSE% MOUNTPOINT
sda                                                                                 
└─sda1      ext4   1.0          b144f8aa-ba14-4a3f-8895-e72999dc697b                
mmcblk0                                                                             
├─mmcblk0p1 vfat   FAT32 boot   C839-E506                             201.4M    20% /boot
└─mmcblk0p2 ext4   1.0   rootfs 568caafd-bab1-46cb-921b-cd257b61f505   26.3G     5% /
```

`fdisk` shall be used to format the drive if needed and then `mkfs.ext4` will create a new EXT4 filesystem.

Create the mount point folder

```
sudo mkdir /mnt/usbdrive/
```

As the `/dev/sda` part might potentially change we will retrieve the disk UUID with :

```
ls -l /dev/disk/by-uuid/ 
```

To permanenty mount the usb drive add the following entry in the `/etc/fstab` file.

```
UUID=b144f8aa-ba14-4a3f-8895-e72999dc697b /mnt/usbdrive ext4 defaults 0 0 
```

Finally mount all
```
sudo mount -a
```

## Install Portainer web portal

Create a portainer.yml file in /mnt/usbdrive

``` yml
version: "3"
name: "base"
services:
    portainer:
        container_name: portainer
        image: 'portainer/portainer-ce:latest'
        restart: always
        ports:
            #- "9000:9000"
            # Only to https
            - "9443:9443"
        environment:
            - TZ=Europe/Paris
        volumes:
            - /var/run/docker.sock:/var/run/docker.sock
            - /mnt/usbdrive/docker/volumes/portainer:/data
        logging:
            driver: fluentd
                options:
                    fluentd-address: localhost:24224
                    tag: portainer

    fluentd:
        container_name: fluentd
        image: 'fluent/fluentd'
        restart: always
        ports:
            - "24224:24224"
            - "24224:24224/udp"
        volumes:
            - ./fluentd/conf:/fluentd/etc


```

Start the container services (portainer in this example) in the background

```
docker-compose -f portainer.yml up -d
docker-compose ls
```

To list downloaded images required to create the containers 
```
docker image ls
```

You can bring everything down, removing the containers entirely, with the down command

```
docker-compose -f portainer.yml down
```

## Open a shell in a docker image

Retrieve the container ID with :

```
docker ps
```

Then open a new bash shell

```
docker exec -ti [CONTAINER-ID] /bin/bash
```

## Stop all containers

You may face a situation where you are required to stop all running containers. For example if you want to remove all containers in Docker, you should stop them beforehand.

```
docker ps -q | xargs docker stop
```

## Delete docker data

To delete leftover images, containers, volumes and other related data, run the following command:

```
sudo rm -rf /var/lib/docker
```

The docker system prune command is a shortcut that prunes images, containers, and networks. Volumes are not pruned by default, and you must specify the --volumes flag for docker system prune to prune volumes.

```
docker system prune
```