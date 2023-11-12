
# Install Home Assistant

> Reference docs :
> - https://www.home-assistant.io/installation/raspberrypi#install-home-assistant-container
> - https://www.homeautomationguy.io/home-assistant-tips/installing-docker-home-assistant-and-portainer-on-ubuntu-linux/

Create a folder to hold all your docker data, e.g. a USB drive with the following path `/mnt/usbdrive/docker/volumes/`.

Clone the repository and update the .env file in the docker directory if needed.
 
If you see any errors errors in the container logs about Permission denied or similar then you have to check if that subfolder in your datafolder belongs to the same user the container is using.

To verify the user’s PUID and PGID, open a SSH session and enter the command : 

``` 
id <username>
``` 

The value we want is the uid and gid, so the next time you create a Docker container that requires the PUID and PGID, we would be using 1000 for the PUID and 1000 for the PGID.

## Docker Container

Create a compose.yml file:

``` yml
version: '3'
services:
  homeassistant:
    container_name: homeassistant
    image: "ghcr.io/home-assistant/home-assistant:stable"
    volumes:
      - /PATH_TO_YOUR_CONFIG:/config
      - /etc/localtime:/etc/localtime:ro
    restart: unless-stopped
    privileged: true
    network_mode: host

```

Finally run the HA container 

```
docker-compose -f compose-files/home-assistant.yml up -d
```

## Check configurations 

To check Home assistant configuration when executed within Docker container

```
docker exec homeassistant python -m homeassistant --script check_config --config /config
```

## Integrations

### MQTT integration

https://www.home-assistant.io/integrations/mqtt/#configuration

### Z-wave integration

https://www.home-assistant.io/integrations/zwave_js

### HACS

Open a terminal from HASS contrainer and run the following command :

```
wget -O - https://get.hacs.xyz | bash -
```
To Configure HACS in registering Github account : https://hacs.xyz/docs/configuration/basic

## Misc

https://ui-lovelace-minimalist.github.io/UI/setup/installation/
