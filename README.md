Steps to setup your own ubuntu server using docker compose to containerise many different applications with seamless maintenance achieving something like below.

![Screenshot](Screenshots/portainer-screenshot.png)


## Ubuntu Server Install ##

1. Get a USB stick, install the latest LTS version of Ubuntu Server and then download balanaEtcher. Run balanaEtcher with what you just downloaded and turn your usb stick into a bootable drive.
2. Plug the USB stick into your server, turn on the server and follow the steps to setting up your server via the console. If you're failing to find drivers then shutdown the installation and re-run the computer to get into the BIOS settings. One common reason for a missing drive is RAID enabled so disable that if so.
3. You have now finished your installation of Ubuntu Server and you should be ready to start creating Docker containers and orchestrating them via docker compose.
4. Install docker https://docs.docker.com/install/#supported-platforms, docker-compose https://docs.docker.com/compose/install/#install-compose, 


## Directory Setup ##

Getting your directory setup correct from the beginning is super important to save time and to have an efficient media-server when it comes to your containers moving files.

### Docker configuration ###

All of your container's docker configuration files will live here. Please note that the config will be stored on your SSD if you follow below. You can change this to whatever you'd like.

`/docker/appdata`

For example, portainer's config will live here `/docker/appdata/portainer`

### Base Directory ### 

The base directory serves a purpose of allowing hard links between media and downloads.

`/mnt/media/media-server/data`

### Downloads Directory ###

Sonarr and Radarr will download the torrents here and use hard links to copy the files over to the `media directory`

`/mnt/media/media-server/data/downloads`

### Media Directory ###

This is where all of your media (tv and movies) will be stored. 

`/mnt/media/media-server/data/media`

Inside media you then have

`/mnt/media/media-server/data/media/tv`
`/mnt/media/media-server/data/media/movies`


### Directory Overview ###

Don't get confused by the complete and incomplete directories. These are added by Radarr and Sonarr.

![Screenshot 2023-12-12 at 21 58 17](https://github.com/adamthewilliam/UbuntuHomeServer/assets/24702294/4912ada4-ad0c-41d2-b3f0-3a1e4c716b94)

You are free to change your directory setup but beware you will need to update the docker-compose.yml and .env to follow your changes.

## Mount Drive ##

This guide does not include any information about partitioning. You will need to figure this out on your own if you go down this path.

Run the following comand to find out information about your drives before mounting them `lsblk` or `sudo lsblk`. Identity your drive's location and type, for example my drive was located at `/dev/sda2` and the type is `exfat`. **You will need these details if you have to replace anything in the below command to mount your drive**.

This command simply mounts your drive to a given directory. The parameters uid and gid are providing the user and group with permissions to read/write on the mount. This is crucial for Radarr, Sonarr, bazarr and prowlarr to work!

`sudo mount -t exfat -o defaults,uid=1000,gid=1000 /dev/sda2 /mnt/media/media-server`

## Useful Links  ##

https://github.com/ghostserverd/mediaserver-docker/tree/master#installation
https://trash-guides.info
