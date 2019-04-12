# venus-docker-grafana

This is a docker-compose file that ties together all thats needed to store data from a
[Victron GX Device]() at ~2 second interval and analyse it using
[Grafana](). Grafana is a super powerful webbased data analysis tool.

## How to use.

### Local Network Setup Using Local MQTT Access 

* `git clone` this repository: `git clone https://github.com/victronenergy/venus-docker-grafana.git`
* By default, if running venus v2.30~35 or later, UPNP will be used to autoimatically discover your local devices
* If not running v2.30~35 or later or your devices are not on the local network, you can setup a custom config file.

### Start the system using UPNP for discovery

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

## Setup To Run On Amazon AWS (EC2/ECS)

#### Installthe amazon ecs-cli
[Install ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_installation.html)

#### Configure your profile
`ecs-cli configure profile --profile-name profile_name --access-key $AWS_ACCESS_KEY_ID --secret-key $AWS_SECRET_ACCESS_KEY`

See (https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_Configuration.html) for more information

#### Configure your cluster
`ecs-cli configure --cluster victron --default-launch-type EC2 --region us-east-1 --config-name victron`

#### Create your cluster 
`ecs-cli up --keypair ECS --capability-iam --size 1 --instance-type t2.medium --cluster-config victron`

#### Deploy to the cluster

Edit the docker-compose-ecs.yaml file and enter your VRM username and password

`ecs-cli compose --ecs-params ecs-params.yml -f docker-compose-ecs.yaml up --create-log-groups --cluster-config victron`

#### Access Grafana
Go to the new EC2 instance to get the public ip address
Then go to http://EC2_PUBLIC_UP

#### To cleanup and bring down the cluster
`ecs-cli down --cluster-config victron`


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

## Influxdb Retention Policy

The default retention policy is 30 days. To change this you can edit [loader.env](loader.env). The value is an influxdb [Duration](https://docs.influxdata.com/influxdb/v1.7/query_language/spec/#durations)

```
CONFIG_FILE=./config/config.json

INFLUXDB_RETENTION=1d
```
