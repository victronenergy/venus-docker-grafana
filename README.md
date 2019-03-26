# victron-influxdb-grafana

## Local Network Setup Using Local MQTT Access 

* `git clone` this repository or download the [docker-compose.yaml](docker-compose.yaml) file
* By default, if running venus v2.30~35 or later, UPNP will be used to autoimatically discover your local devices
* If not running v2.30~35 or later or your devices are not on the local network, you can setup a custom config file.

### Start the system using UNPNP for discovery

In the diretory containing the docker-compose.yaml file, execute `docker-compose up --detach`

### Start the system using custom configuration file

Modify or copy [config/config_local.json](config/config_local.json) and enter the ip address and portalId of your device(s).

For example:

```json
{
  "mqtt_servers": [
    {
      "portalIDs": ["985dadcbffff"],
      "address": "192.168.1.3",
      "port": 1883
    },
    {
      "portalIDs": ["985dadcbfff0"],
      "address": "192.168.1.4",
      "port": 1883
    }
  ]
}
```


Edit the [loader.env](loader.env) file and change the `CONFIG_FILE` variable to point to your config json file:

In the diretory containing the docker-compose.yaml file, execute `docker-compose up --detach`

## Setup To Use VRM MQTT

With this setup, you can run the system on any machine that has access to the internet, you do not need to have access to your Venus devices.

Edit the [loader.env](loader.env) file and uncomment and change the `VRM_USER_NAME` and `VRM_PASSWORD` variables:

```
CONFIG_FILE=./config/config.json

VRM_USER_NAME=myusername
VRM_PASSWORD=mypassword
```

In the diretory containing the docker-compose.yaml file, execute `docker-compose up --detach`

