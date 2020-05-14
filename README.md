# venus-docker-grafana - Advanced Victron Dashboarding

Venus-docker-grafana is a dashboarding solution for Victron Energy systems.
Its a niche alternative to our main solution, the [VRM Portal](https://vrm.victronenergy.com).

Compared to VRM, the docker grafana solution is:

- more work to install & configure
- not officially supported.

But also offers:

- more granular data, its recorded at a (approx) two second interval
- an offline solution: venus-docker-grafana can run on a computer local to a Victron system. No internet required.
- far more options in graphing, visualisation and customisation.

This solution can work with one or more GX Devices in your local network, as well connect
to devices via the VRM cloud.

Note that "Grafana" is not a Victron product. Its an existing widely used dashboarding solution.
Make sure to go online and learn more about it.

An example of a dashboard:

![hydro power docker example](https://raw.githubusercontent.com/victronenergy/venus-docker-grafana/master/hydro%20grafana.jpg)

## 1. Requirements

- A Victron Energy system including a [Victron GX Device](https://www.victronenergy.com/live/venus-os:start)
- A computer to host and run all this. It can be a Windows or Apple laptop or
computer. But also a raspberrypi: Running this solution can be done on any platform that support Docker.
- Or, rather than hosting it locally, it can also be hosted on Amazon AWS and other cloud
providers. See the [AWS instructions](AWS.md).
- Patience and willingness to study and figure all this out. Beware that thos is not an officially supported Victron solution. Our partners, dealers and neither ourselves will help you in case of problems. For support, [go to Community and search for Grafana](https://community.victronenergy.com/search.html?c=&includeChildren=&f=&type=question+OR+idea+OR+kbentry+OR+answer+OR+topic+OR+user&redirect=search%2Fsearch&sort=relevance&q=grafana).
The main item in this repository is a docker-compose file that ties together all thats needed to store & visualise the data. from a at ~2 second interval and analyse it using
[Grafana](https://grafana.com/). Grafana is a super powerful webbased data analysis tool.
Which is quite easy to learn.

This repository contains the docker compose configurations and is all that is needed to get up and running. The source code and setup for the docker images are located here: https://github.com/victronenergy/venus-docker-grafana-images 

For the latest info on changes we
are making to the solution, see: https://github.com/victronenergy/venus-docker-grafana-images/releases

## 2. How to install

### 2.1 Host it locally

This video walks through below steps: https://youtu.be/j4kdKqDGuys

1. Download and install [Docker desktop](https://www.docker.com/products/docker-desktop). Its available for Windows, macOS, Linux and more operating systems.
1. Enable the MQTT service on your GX Device in Settings -> Services.
2. Download the docker compose [file](https://raw.githubusercontent.com/victronenergy/venus-docker-grafana/master/docker-compose.yaml) (right-click and use Save as)
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

### 2.2 Host in on Amazon AWS or another cloud provider.

This section needs explaining. Pull requests welcome.

## 3. Influxdb Measurement Storage

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

## 4. Influxdb Retention Policy

The default retention policy is 30 days. To change this you can go to Configuration -> InfluxDB In the admin interface. The value is an influxdb [Duration](https://docs.influxdata.com/influxdb/v1.7/query_language/spec/#durations)

## 5. How does this work?

Docker is a container technology, see Google.

The data is retrieved using the
[MQTT protocol](https://github.com/victronenergy/dbus-mqtt).

[The server](https://github.com/victronenergy/venus-docker-grafana-images/tree/master/server)
contains the Node JS code that takes care of the MQTT communication and storing the data in
the Influx database.

This repo only contains the docker-compose file. The rest of the sources, there is only a handful, is in
[venus-docker-grafana-images](https://github.com/victronenergy/venus-docker-grafana-images).
