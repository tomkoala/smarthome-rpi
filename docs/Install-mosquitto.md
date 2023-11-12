# Mosquitto

The Mosquitto MQTT broker is suitable for a lightweight install especially in docker environment
https://github.com/vvatelot/mosquitto-docker-compose

## Install the docker container

Add the following content to the Docker Compose file:

```
services:
  mosquitto:
    image: eclipse-mosquitto
    container_name: mosquitto
    restart: unless-stopped
    volumes:
      - "${DATADIR}/conf/mosquitto:/mosquitto/config"
      - "${DATADIR}/volumes/mosquitto:/mosquitto/data"
      - "${DATADIR}/logs/mosquitto:/mosquitto/log"
    ports:
      - 1883:1883
```

## Prepare mosquitto configuration

Mosquitto is configurable through `mosquito.conf` located `/mosquitto/config`
> https://mosquitto.org/man/mosquitto-conf-5.html

By default the log and data persistance is activated (logs are in the log folder, and data are stored in a docker voume).

```
persistence true
persistence_location /mosquitto/data/

log_dest stdout
log_dest file /mosquitto/log/mosquitto.log
log_type warning
log_timestamp true
connection_messages true

listener 1883

## Authentication ##
allow_anonymous false
password_file /mosquitto/config/password.txt
```

By default, Mosquitto >=2.0 allows only authenticated connections. Disabling this setting could be at risk since any device will be able to publish or subscribe to MQTT topics without any restrictions.


## Setting up authentication for Mosquitto

From Portainer portal open a new Mosquitto `/bin/sh` shell into the running Mosquitto docker container

Reach the Console section on the Container details view and select the /bin/sh shell, finally click on connect.

Once connected on the shell, run the following command to create a new MQTT user and password for authentication : 

```
mosquitto_passwd -c /mosquitto/config/password.txt <username>
```
where `<username>` can be for instance `hass`

> :memo: **Note**
> Option `-c` must be used to create a new password file. It shall not be used to add new password(s) to the existing file

Restart the Mosquitto docker container to apply changes.

## Connecting Home Assistant to the MQTT Broker

In Home Assistant, navigate to the Configuration menu and to the Integrations page.  

Click the Add Integration button at the bottom right, and search for the MQTT integration.

Type the IP Address of your linux server in as the Broker address, leave port 1883 as default and then enter the username and password for the hass MQTT user you created in the previous step.

> Reference docs :
>
> https://www.homeautomationguy.io/docker-tips/configuring-the-mosquitto-mqtt-docker-container-for-use-with-home-assistant/