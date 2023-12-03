Steps to setup your own ubuntu server using docker compose to containerise many different applications with seamless maintenance.

![Screenshot](Screenshots/portainer-screenshot.png)

1. Get a USB stick, install the latest LTS version of Ubuntu Server and then download balanaEtcher run it with the file you just downloaded and turn your usb stick into a bootable drive.
2. Follow the steps to setting up your server via the console. If you're failing to find drivers then shutdown the installation and re-run the computer to get into the BIOS settings to turn off RAID in the storage settings.
3. You have now finished your installation of Ubuntu Server and you should be ready to start creating Docker containers and orchestrating them via docker compose.
4. Follow this guide:
5. Here is the docker compose file that I have created my private variables hidden away in an environment file. You can find an example environment file here.
6. Change the credentials to anything you want.
7. 