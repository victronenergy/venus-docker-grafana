# venus-docker-grafana

This is a docker-compose file that ties together all thats needed to store data from a
[Victron GX Device](https://www.victronenergy.com/live/venus-os:start) at ~2 second interval and analyse it using
[Grafana](https://grafana.com/). Grafana is a super powerful webbased data analysis tool.
Which is quite easy to learn.

This solution can work with one or more GX Devices in your local network, as well connect
to devices via the VRM cloud.

Running this solution can be done on a Linux host or VM, and also a RaspberryPi will
suffice. The latter obviously has limitations in storage and performance.

Besides hosting this yourself, it can also be hosted on Amazon AWS and other cloud
providers. See AWS instructions below.

This readme starts with how to use it. See further below for the dev. details.

## 1. How to use

### 1.1 Host it locally

1. Enable the MQTT service on your GX Device in Settings -> Services.
1. `git clone` this repository: `git clone https://github.com/victronenergy/venus-docker-grafana.git`
2. Follow instructions for option A, B OR C below
3. Accessing Grafana on http://localhost:3000 and enter `admin` for user name and `admin` for password.

#### Option A) Automatic discovery - works for devices in local network

In the directory containing the docker-compose.yaml file, execute `docker-compose up --detach`.

UPnP discovery will automatically find and start datalogging for all GX devices running
v2.30~35 or later.

If not running v2.30~35 or later or your devices are not on the local network, you can setup
a custom config file.

#### Option B) Manual configuration for devices in a local network

Modify [config/config_local.json](config/config_local.json) and enter the ip address and portalId of your device(
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

#### Option C) Setup To Use VRM MQTT

With this setup, you can run the system on any machine that has access to the internet, you do not
need to have local access to your GX devices.

Edit the [loader.env](loader.env) file and uncomment and change the `VRM_USER_NAME` and `VRM_PASSWORD` variables:

```
CONFIG_FILE=./config/config.json

VRM_USER_NAME=myusername
VRM_PASSWORD=mypassword
```

In the diretory containing the docker-compose.yaml file, execute `docker-compose up --detach`

### 1.2 Host it on Amazon AWS (EC2/ECS)

#### Step 1. Install the amazon ecs-cli
[Install ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_installation.html)

#### Step 2. Configure your profile
`ecs-cli configure profile --profile-name profile_name --access-key $AWS_ACCESS_KEY_ID --secret-key $AWS_SECRET_ACCESS_KEY`

See (https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_Configuration.html) for more information

#### Step 3. Configure your cluster
`ecs-cli configure --cluster victron --default-launch-type EC2 --region us-east-1 --config-name victron`

#### Step 4.  Create your cluster 
`ecs-cli up --keypair ECS --capability-iam --size 1 --instance-type t2.medium --cluster-config victron`

#### Step 5. Deploy to the cluster

Edit the docker-compose-ecs.yaml file and enter your VRM username and password

`ecs-cli compose --ecs-params ecs-params.yml -f docker-compose-ecs.yaml up --create-log-groups --cluster-config victron`

#### Step 6. Access Grafana
Go to the new EC2 instance to get the public ip address
Then go to http://EC2_PUBLIC_IP

#### Step 7. To cleanup and bring down the cluster
`ecs-cli down --cluster-config victron`


## 2. Influxdb Measurement Storage

Measurements are stored in influxdb using a modified version of the MQTT topics.

The portal id and instance numbers are removed from the name and are "tags" on the data

Example measurement names: `battery/Dc/0/Voltage`, `solarcharger/Dc/0/Current`

If you have multiple GX devices, or multiple devices of the same type, you may need to add
portalId and or/instanceNumber to your Grafana queries

For example: 
```
SELECT mean("value") FROM "solarcharger/Dc/0/Current" WHERE ("portalId" = '985dadcb8af0' AND "instanceNumber" = '258') AND $timeFilter GROUP BY time($__interval) fill(null)
```

## 3. Influxdb Retention Policy

The default retention policy is 30 days. To change this you can edit [loader.env](loader.env). The value is an influxdb [Duration](https://docs.influxdata.com/influxdb/v1.7/query_language/spec/#durations)

```
CONFIG_FILE=./config/config.json

INFLUXDB_RETENTION=1d
```

## 4. How does this work?

Docker is a container technology, see Google.

The data is retrieved using the
[MQTT protocol](https://github.com/victronenergy/dbus-mqtt).

[The loader](https://github.com/victronenergy/venus-docker-grafana-images/tree/master/loader)
contains the Python code that takes care of the MQTT communication and storing the data in
the Influx database.

This repo only contains the docker-compose file. The rest of the sources, there is only a handful, is in
[venus-docker-grafana-images](https://github.com/victronenergy/venus-docker-grafana-images).
