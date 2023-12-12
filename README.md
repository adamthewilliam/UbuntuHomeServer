Steps to setup your own ubuntu server using docker compose to containerise many different applications with seamless maintenance achieving something like below.

![Screenshot](Screenshots/portainer-screenshot.png)

1. Get a USB stick, install the latest LTS version of Ubuntu Server and then download balanaEtcher. Run balanaEtcher with what you just downloaded and turn your usb stick into a bootable drive.
2. Plug the USB stick into your server, turn on the server and follow the steps to setting up your server via the console. If you're failing to find drivers then shutdown the installation and re-run the computer to get into the BIOS settings. One common reason for a missing drive is RAID enabled so disable that if so.
3. You have now finished your installation of Ubuntu Server and you should be ready to start creating Docker containers and orchestrating them via docker compose.
5. I have provided an example docker compose file with a .env file. By looking at the .env file you can see that the directory structure is like so:

7. Run the following command to run all of your containers together: "Docker Compose up -d"