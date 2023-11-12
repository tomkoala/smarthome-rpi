# NodeRED

Node-RED is a flow-based programming tool built upon NodeJS.

## Install the docker container

Below an example of a Docker Compose file which can be run by docker stack or docker-compose. Please refer to the official Docker pages for more info about Docker stack and Docker compose.

```
services:
  node-red:
    image: nodered/node-red:latest
    environment:
      - TZ=Europe/Paris
    ports:
      - "1880:1880"
    networks:
      - node-red-net
    volumes:
      - node-red-data:/data

volumes:
  node-red-data:

networks:
  node-red-net:
```

The above compose file:
- creates a node-red service
- pulls the latest node-red image
- sets the timezone to Europe/Paris
- Maps the container port 1880 to the host port 1880
- creates a node-red-net network and attaches the container to this network
- persists the /data dir inside the container to the node-red-data volume in Docker

```
# docker-compose -f docker-compose-node-red.yml -p myNoderedProject up
```

## Set folder permissions

Attention ! Si vous utilisez un répertoire (et non un volume Docker), il faut que celui-ci appartienne à un utilisateur ayant comme uid 1000 et guid 1000. (voir la documentation)

Si l'utilisateur qui lance la stack n'a pas les bons uid et guid (comme root), vous pouvez simplement utiliser la commande suivante :

```
chown 1000:1000 /votre/repertoire/nodered/data
```

https://github.com/node-red/node-red-docker/wiki/Permissions-and-Persistence