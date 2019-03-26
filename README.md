# victron-influxdb-grafana

## Local Network Setup Using Local MQTT Access 

* `git clone` this repository: `git clone https://github.com/victronenergy/venus-docker-grafana.git`
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

## Accessing Grafna

Go to http://localhost:3000 and enter `admin` for user name and `admin` for password. Please change the password when prompted.

## Influxdb Measurement Storage

Measurments are stored in influxdb using a modified version of the MQTT topics.

The portal id and instance numbers are removed from the name and are "tags" on the data

Example measurement names: `battery/Dc/0/Voltage`, `solarcharger/Dc/0/Current`

If you have multiple venus servers or mulitple devices of the same type, you may need to add portalId and or/instanceNumber to your Grafana queries

For example: 
```
SELECT mean("value") FROM "solarcharger/Dc/0/Current" WHERE ("portalId" = '985dadcb8af0' AND "instanceNumber" = '258') AND $timeFilter GROUP BY time($__interval) fill(null)
```
