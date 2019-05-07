# venus-docker-grafana

This is a docker-compose file that ties together all thats needed to store data from a
[Victron GX Device](https://www.victronenergy.com/live/venus-os:start) at ~2 second interval and analyse it using
[Grafana](https://grafana.com/). Grafana is a super powerful webbased data analysis tool.
Which is quite easy to learn.

This solution can work with one or more GX Devices in your local network, as well connect
to devices via the VRM cloud.

Running this solution can be done on any platform that support Docker, and also a RaspberryPi will
suffice. The latter obviously has limitations in storage and performance.

Besides hosting this yourself, it can also be hosted on Amazon AWS and other cloud
providers. See AWS instructions below.

This readme starts with how to use it. See further below for the dev. details.

## 1. How to use

### 1.1 Host it locally

1. Enable the MQTT service on your GX Device in Settings -> Services.
2. Download the docker compose [file](https://raw.githubusercontent.com/victronenergy/venus-docker-grafana/master/docker-compose.yaml)
3. In the directory containing the downloaded file, execute `docker-compose up --detach`.
4. Go to the admin interface @ http://localhost:8088
4. Follow instructions for option A, B OR C below
5. Accessing Grafana on http://localhost:3000 and enter `admin` for user name and `admin` for password.

#### Option A) Automatic discovery - works for devices in local network

Go to Configuraton -> Local Discovery and turn `Enabled` on and click `Save`

UPnP discovery will automatically find and start datalogging for all GX devices running
v2.30~35 or later.

#### Option B) Manual configuration for devices in a local network

Go to Configuration -> Manual and turn on `Enabled`

Click `Add` and enter the IP address or hostname for each device you want to use.

Clisk `Save` to start collecting data

#### Option C) Setup To Use VRM MQTT

With this setup, you can run the system on any machine that has access to the internet, you do not
need to have local access to your GX devices.

If you have two factor authentication enabled on VRM, please turn it off.

Go to Configuration -> VRM and turn on `Enabled`

Enter your VRM username and password and click `Login`.

All your devices in VRM will show in the list.

Clisk `Save` to start collecting data


## 2. Influxdb Measurement Storage

Measurements are stored in influxdb using a modified version of the MQTT topics.

The portal id and instance numbers are removed from the name and are "tags" on the data

THe device name, if available, is also added as a "tag"

Example measurement names: `battery/Dc/0/Voltage`, `solarcharger/Dc/0/Current`

If you have multiple GX devices, or multiple devices of the same type, you may need to add
portalId and or/instanceNumber to your Grafana queries

Examples: 
```
SELECT mean("value") FROM "solarcharger/Dc/0/Current" WHERE ("portalId" = '985dadcb8af0' AND "instanceNumber" = '258') AND $timeFilter GROUP BY time($__interval) fill(null)
```
```
SELECT mean("value") FROM "solarcharger/Dc/0/Current" WHERE ("name" = 'Boat' AND "instanceNumber" = '258') AND $timeFilter GROUP BY time($__interval) fill(null)
```

## 3. Influxdb Retention Policy

The default retention policy is 30 days. To change this you can go to Configuration -> InfluxDB In the admin interface. The value is an influxdb [Duration](https://docs.influxdata.com/influxdb/v1.7/query_language/spec/#durations)

## 4. How does this work?

Docker is a container technology, see Google.

The data is retrieved using the
[MQTT protocol](https://github.com/victronenergy/dbus-mqtt).

[The server](https://github.com/victronenergy/venus-docker-grafana-images/tree/master/server)
contains the Node JS code that takes care of the MQTT communication and storing the data in
the Influx database.

This repo only contains the docker-compose file. The rest of the sources, there is only a handful, is in
[venus-docker-grafana-images](https://github.com/victronenergy/venus-docker-grafana-images).
