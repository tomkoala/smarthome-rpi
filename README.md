# smarthome-rpi
A simple setup to configure your own smart home server with a container based approach on a Raspberry Pi

## Introduction
Home Assistant as main cencentrator

The smart home market is really fragmented. Using different gateways and Apps from different ecosystems can be annoying, expansive and might not work very well together. Instead I suggest using a custom smart home server with some open source software that can replace all your hubs and give you access to one system to control it all.

## Hardware Requirements
The Raspberry Pi 3 is the right candidate to run home automation open source assets, however the guidelines provided in this repo can be used on pretty much any platform managed by Docker.

## Install OS

To install [Raspberry PI OS](https://www.raspberrypi.com/software/) on device plenty of tutorials are available online. For beginners a HowTo for a [Headless RPI setup on Mac](/docs/Headless-Rpi-setup-on-Mac.md) is available. Following a clean install it is recommended to switch to a [read-only file system](/docs/ReadOnly-Filesystem.md) to preserve SD card lifetime.

If you already have a Raspberry Pi set up with Raspbian, internet and enabled SSH, you can skip this first step.

Overall these are some pre-requisites for a robust and reliable home automation solution :
- SSH Keys for remote authentication
- Ethernet connection to be prefered for security purposes
- External drive as main storage for docker data

## Install Docker

Install Docker and Docker-compose with the following [guide](/docs/Install-docker.md)

## Install Home Automation containers

Create a folder to hold all your docker data on device (e.g. ``/mnt/usbdrive/docker/``). Then clone this repository and copy the content on target, finally update the ``.env`` file located in ``/compose-files``

```
docker-compose -f base.yml up -d
docker-compose -f homeautomation.yml up -d

// to see logs
docker-compose -f ...yml logs -f
// to see if containers are running
docker-compose -f ...yml ps

// to stop
docker-compose -f ...yml down
```

### Content

``base.yml`` Compose file :

| Service  | Port |  Setup |
| ------------- | ------------- | ------------- |
| Portainer  | 9443  | Create credentials with the first login at https://domopi.local:9443/|
| Fluentd  | 24224 (udp)  | - |

``homeautomation.yml`` Compose file :

| Service  | Port |  Setup |
| ------------- | ------------- | ------------- |
| HomeAssistant  | 8123  | Just go to the webpage and follow the setup wizard |
| Mosquitto  | 1883  | The server relies on the configuration located in `` /conf/mosquitto`` and already configured in Mosquitto container. A new authentication entry and password must created for all MQTT clients. For more details reach the installation [guide](/docs/Install-mosquitto.md)   |
| zwave-js-ui  | -  | Setup can be done according to the [guide](/docs/Install-zwave-js.md) especially for the binding of the USB Z-Wave controller 

