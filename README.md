```markdown
# Setting Up a Media Server with Docker Compose on Ubuntu

This guide walks you through setting up a media server on Ubuntu (or other Docker-compatible OS) using Docker Compose.  This approach containerizes various applications for easier maintenance and management.

![Screenshot](Screenshots/portainer-screenshot.png)

## Prerequisites

*   A fresh installation of Ubuntu Server (or another compatible OS).
*   A text editor (nano, vim, etc.).
*   Basic familiarity with the command line.

## Step 1: Install Docker and Docker Compose

1.  **Install Docker:** Follow the official Docker installation guide for your OS: [https://docs.docker.com/install/#supported-platforms](https://docs.docker.com/install/#supported-platforms)

    For Ubuntu, a common method is:

    ```bash
    sudo apt update
    sudo apt install apt-transport-https ca-certificates curl software-properties-common
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt update
    sudo apt install docker-ce docker-ce-cli containerd.io
    ```

2.  **Install Docker Compose:** Follow the official Docker Compose installation guide: [https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/)

    For most Linux systems:

    ```bash
    sudo apt install docker-compose-plugin
    ```

    Verify installation:

    ```bash
    docker compose version
    ```

## Step 2: Disk Partitioning and Formatting (Optional but Recommended)

If you have additional drives for media storage, partition and format them before mounting.  This section assumes you are using a single EXT4 partition for simplicity. Adapt these commands based on your specific drive setup.

**WARNING: Incorrectly using these commands can lead to data loss. Double-check device names before proceeding!**

1.  **Identify your drive:** Use `lsblk` or `sudo lsblk` to list available block devices and identify the drive you want to format (e.g., `/dev/sdb`).  *Be absolutely sure you identify the correct drive.*

2.  **Create a partition (if needed):**  If the drive is unpartitioned, use `fdisk`, `gdisk`, or `parted` to create a new partition.  This is beyond the scope of this simplified guide, but there are many tutorials online.  For example, using `fdisk`:

    ```bash
    sudo fdisk /dev/sdb  # Replace /dev/sdb with your drive
    ```
    Follow the prompts. Typically, you'll create a primary partition (type `n`), accept the defaults for start and end sectors, and then write the changes (type `w`).

3.  **Format the partition:** Format the partition with EXT4:

    ```bash
    sudo mkfs.ext4 /dev/sdb1 # Replace /dev/sdb1 with your partition
    ```

## Step 3: Directory Setup

Creating the correct directory structure is crucial for hard links and efficient media server operation.

1.  **Create the base directory:**

    ```bash
    sudo mkdir -p /mnt/media/media-server/data
    ```

2.  **Create subdirectories:**

    ```bash
    sudo mkdir -p /mnt/media/media-server/data/downloads
    sudo mkdir -p /mnt/media/media-server/data/media
    sudo mkdir -p /mnt/media/media-server/data/media/tv
    sudo mkdir -p /mnt/media/media-server/data/media/movies
    ```

3. **Docker configuration directory:**

    ```bash
    sudo mkdir -p /docker/appdata
    ```
   *Important: Create the /docker folder at the server root*

   Example:

   ```bash
   cd ~
   sudo mkdir docker
   cd docker
   sudo mkdir appdata
   ```

## Step 4: Mount Drive(s)

Mount the formatted drive(s) to the appropriate directory.

1.  **Create mount point (if needed):**  The directories created in the last step serve as the mount points.

2.  **Mount the drive:**

    ```bash
    sudo mount -t ext4 /dev/sdb1 /mnt/media/media-server/data # Replace with your partition and mount point
    ```

3.  **Make the mount permanent (using `/etc/fstab`):**  This is crucial to ensure the drive mounts automatically on reboot.

    a.  Get the UUID of your partition:

        ```bash
        sudo blkid /dev/sdb1 # Replace with your partition
        ```

        Copy the UUID.

    b.  Edit `/etc/fstab`:

        ```bash
        sudo nano /etc/fstab
        ```

    c.  Add a line like this (replace with *your* UUID and mount point):

        ```
        UUID=YOUR_UUID /mnt/media/media-server/data ext4 defaults 0 0
        ```

    d.  Save the file and exit.

    e.  Test the mount:

        ```bash
        sudo mount -a
        ```

        If no errors occur, the drive is mounted correctly.

## Step 5: .env File Setup

The `.env` file stores environment variables used by Docker Compose.

1.  **Create the file:**

    ```bash
    cd /docker # Replace with your docker directory
    touch .env
    sudo chown root:root .env
    sudo chmod 600 .env
    ```

2.  **Edit the file:**

    ```bash
    sudo nano .env
    ```

3.  **Add environment variables:**  Populate the `.env` file with the following variables.  **Replace the placeholder values with your actual values.**

```
## Directories ##
DOCKER_CONFIG_DIR=/docker/appdata
BASE_DIR=/mnt/media/media-server/data
DOWNLOAD_DIR=/mnt/media/media-server/data/downloads
MEDIA_DIR=/mnt/media/media-server/data/media
TV_DIR=/mnt/media/media-server/data/media/tv
MOVIES_DIR=/mnt/media/media-server/data/media/movies

## User ##
PUID=1000  # Your user ID (run `id -u`)
PGID=1000  # Your group ID (run `id -g`)
TZ="Europe/London"  # Your timezone (e.g., "America/Los_Angeles")

## Portainer ##
PORTAINER_PORT=9000

## Plex ##
PLEX_CLAIM= # Your Plex Claim Token (Optional, if claiming a new server)

## Tailscale
TAILSCALE_PLEX_AUTH_KEY=tskey- # Generate a reusable key from the Tailscale dashboard
TAILSCALE_OVERSEERR_AUTH_KEY=tskey- # Generate a reusable key from the Tailscale dashboard

## Overseerr ##
OVERSEERR_WEB_PORT=5055

## Transmission ##
TRANSMISSION_WEBUI_PORT=9091
TRANSMISSION_CONNECTION_PORT=51413

TRANSMISSION_WEBUI_USER=user # Change to your desired username
TRANSMISSION_WEBUI_PASS=strong-password # Change to a strong password

TRANSMISSION_MAX_RETENTION=2592000
TRANSMISSION_CONNECTION_PORT=10

## Qbittorrent ##
QBITTORRENT_WEB_UI_PORT=8080
QBITTORRENT_CONNECTION_PORT=6881

## Prowlarr ##
PROWLARR_PORT=9696
PROWLARR_API_KEY= # Get this from Prowlarr's settings after setup

## Radarr ##
RADARR_PORT=7878
RADARR_API_KEY= # Get this from Radarr's settings after setup

## Sonarr ##
SONARR_PORT=8989
SONARR_API_KEY= # Get this from Sonarr's settings after setup

## Bazarr ##
BAZARR_PORT=6767

## Heimdall ##
HEIMDALL_PORT=8888
HEIMDALL_SSL_PORT=4444

## PiHole ##
PIHOLE_WEB_PASSWORD=strong-password # Set a strong password
PIHOLE_WEB_PORT=53
PIHOLE_CONNECTION_PORT=80
SERVER_LAN_IP=192.168.1.125 # The static IP address of your server

## Prometheus  ##
PROMETHEUS_PORT=9090

## Node-exporter ##
NODE_EXPORTER_PORT=9100

## Grafana ##
GRAFANA_PORT=3000
GF_SECURITY_ADMIN_USER="admin"
GF_SECURITY_ADMIN_PASSWORD="grafana"

## Sabnzbd ##
SABNZBD=8080

## Scrutiny ##
SCRUTINY_WEB_INFLUX_DB_TOKEN=
SCRUTINY_WEB_INFLUXDB_INIT_USERNAME=
SCRUTINY_WEB_INFLUXDB_INIT_PASSWORD=
USENET_BASE_DIR=/mnt/media/media-server/data/usenet
TORRENTS_BASE_DIR=/mnt/media/media-server/data/torrents
```

*   **Explanation of Important Variables:**

    *   `PUID` and `PGID`:  These variables define the user and group IDs that the containers will run under. This is critical for file permissions.  Use `id -u` and `id -g` to find your user and group IDs, respectively.

    *   `TZ`:  Your timezone, used for accurate scheduling.

    *   `API_KEY` variables (Radarr, Sonarr, Prowlarr): You'll get these *after* you initially set up those containers through their web UI. They are used for inter-application communication.

    *   `SERVER_LAN_IP`:  The static IP address of your server. Important if you're using Pi-hole for DNS.

    *   `MEDIA_DIR`, `DOWNLOAD_DIR`: Defines where all of the data will be stored on the server

4.  **Save the file and exit.**

## Step 6: Run Docker Compose

1.  **Navigate to the directory containing your `docker-compose.yml` file:**

    ```bash
    cd /docker # Replace with your docker directory
    ```

2.  **Start the containers:**

    ```bash
    docker compose up -d
    ```

    This will download the necessary images and start all the containers in detached mode (running in the background).

3.  **Check container status:**

    ```bash
    docker compose ps
    ```

    Verify that all containers are running and healthy.

## Step 7: Container Setup (Web UI Configuration)

Access the web UIs of each service using the ports defined in your `docker-compose.yml` file (e.g., `http://your_server_ip:9000` for Portainer, `http://your_server_ip:7878` for Radarr).  Configure each application according to your preferences.

*   **Key Configuration Points:**

    [Follow this in-depth guide to configure the arr services, transmission and plex](https://trash-guides.info)

    Read [Sabznbd Wiki](https://sabnzbd.org/wiki/introduction/usage) and [Usenet subreddit wiki](https://www.reddit.com/r/usenet/wiki/index/) to learn more about Usenet and to find the best indexers and providers. 
    *   **Radarr/Sonarr:**
        *   Add your indexers/download clients (e.g., Transmission, SABnzbd).
        *   Configure root folders (where media files will be stored).
        *   Set up quality profiles.
        *   Obtain the API keys for Radarr/Sonarr and populate them in the `.env` file for use with other applications.
    *   **Prowlarr:**  Add indexers and link them to Radarr and Sonarr.  Get the API key.
    *   **Transmission/SABnzbd:** Configure download directories and web UI credentials.
    *   **Plex:**  Add your media libraries.
    *   **Overseerr:**  Configure connection to Plex/Jellyfin, Radarr and Sonarr.
    *   **Bazarr:** Configure Sonarr and Radarr connections.


## Step 8: Initial Tailscale Configuration

1.  **Access the Tailscale containers via CLI**

    ```bash
    docker exec -it tailscale-plex tailscale status
    docker exec -it tailscale-overseerr tailscale status
    ```

2.  **Authenticate via the Tailscale dashboard**

    You should see that the service asks you to authenticate, navigate to the website and connect to your tailnet.

3.  **Disable key expiry**

    In the tailscale dashboard under "machines" disable key expiry to ensure the container stays connected to your tailnet.

4. Go to your Tailscale dashboard and add any devices that you would like to access plex or overseerr on outside of the LAN hosting your Ubuntu server.


## Troubleshooting

*   **Permissions Issues:**  Ensure that the `PUID` and `PGID` in your `.env` file match the user and group that own the media directories.  Docker volumes can sometimes have permission issues.
*   **Network Issues:**  Double-check that ports are not blocked by your firewall (UFW, etc.).  If using Pi-hole, verify that DNS is resolving correctly from inside the containers.
*   **Configuration Errors:** Carefully review the configuration of each application in their respective web UIs.  Check logs for error messages.
*   **Docker Compose Errors:** If `docker compose up` fails, check the logs for errors related to image downloads or configuration issues.

## Useful Links

*   [https://github.com/ghostserverd/mediaserver-docker/tree/master#installation](https://github.com/ghostserverd/mediaserver-docker/tree/master#installation)
*   [https://trash-guides.info](https://trash-guides.info)
*   [https://wiki.servarr.com/radarr/quick-start-guide](https://wiki.servarr.com/radarr/quick-start-guide)
*   [https://wiki.servarr.com/sonarr/quick-start-guide](https://wiki.servarr.com/sonarr/quick-start-guide)
*   [https://wiki.servarr.com/prowlarr/quick-start-guide](https://wiki.servarr.com/prowlarr/quick-start-guide)
*   [https://github.com/awesome-jellyfin/awesome-jellyfin](https://github.com/awesome-jellyfin/awesome-jellyfin)
*   [https://www.phind.com/search?home=true](https://www.phind.com/search?home=true)

## Updating Containers

To update your containers to the latest versions:

1.  Navigate to the directory containing your `docker-compose.yml` file.

2.  Pull the latest images:

    ```bash
    docker compose pull
    ```

3.  Recreate the containers:

    ```bash
    docker compose up -d
    ```

## Maintaining the Server

*   **Regularly Update Ubuntu:** Keep your Ubuntu server up-to-date with security patches and updates:

    ```bash
    sudo apt update
    sudo apt upgrade
    ```

*   **Monitor Disk Space:** Regularly check disk space usage to prevent issues:

    ```bash
    df -h
    ```

*   **Backups:** Implement a backup strategy for your configuration files and media library to prevent data loss.

This comprehensive guide provides a solid foundation for setting up a media server with Docker Compose. Remember to adapt the instructions to your specific needs and preferences.
```