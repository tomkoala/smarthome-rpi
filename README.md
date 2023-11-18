# smarthome-rpi
A simple setup to configure your own smart home server based on 🏡 [Home Assistant](https://github.com/home-assistant/) with a container based approach on a Raspberry Pi

## Introduction
Home Assistant is used as main cencentrator / Hub to rule them all!

Using different proprietary home automation gateways and systems can be annoying over time, open source solutions are definitely improving interoperability between eco-systems

Below is the chosen architecture and underlying components :

| Component  | Description | 
| ------------- | ------------- |
| Concentrator | a Home Assistant instance managed as central control system for smart home devices 
| Zigbee Controller | a dedicated Zigbee Hub to connect the main wireless sensors / devices managed by a customized OpenWrt linux distro and running a Zigbee2MQTT instance | 
| Z-Wave Controller | a dedicated Z-Wave Hub to connect some additionnal legacy devices | 
| Message Broker |  a MQTT broker to send/publish messages back and forth | 
| Control API | a RESTful architecture to manage some remote commands on some smart controllers 


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

Finally run the containers with docker-compose :

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

``base.yml`` Docker Compose file :

| Service  | Port |  Setup |
| ------------- | ------------- | ------------- |
| Portainer  | 9443  | Create credentials with the first login at https://domopi.local:9443/|
| Fluentd  | 24224 (udp)  | - |

``homeautomation.yml`` Docker Compose file :

| Service  | Port |  Setup |
| ------------- | ------------- | ------------- |
| HomeAssistant  | 8123  | Just go to the homepage and follow the setup wizard at http://domopi.local:8123|
| Mosquitto  | 1883  | The server relies on the configuration located in `` /conf/mosquitto`` and already configured in Mosquitto container. A new authentication entry and password must created for all MQTT clients. For more details reach the installation [guide](/docs/Install-mosquitto.md)   |
| zwave-js-ui  | -  | Setup can be done according to the [guide](/docs/Install-zwave-js.md) especially for the binding of the USB Z-Wave controller. Web interface is available at http://domopi.local:8091

## Troubleshooting

[Fluentd](https://www.fluentd.org/) is used in this smart home setup as a unified logging solution to store/manage App logs, it requires very little system resource so is a good match with the chosen Pi-based platform. 

More info about Fluentd flow and configuration :
https://docs.fluentd.org/quickstart/life-of-a-fluentd-event

## FAQ

Feel free to ask any questions using GitHub [Issues](http://github.com/tomkoala/smarthome-rpi/issues).
