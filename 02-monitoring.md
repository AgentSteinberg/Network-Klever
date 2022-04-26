# Klever monitoring

## About

This guideline describes a toolset on how to monitor the operation of a node on the for the [Klever Blockchain](https://klever.finance/kleverchain/).

Please note, that there are multipe solutions on how to monitor your node. Whereas this guide is using the following toolchain:

* [Node-RED](https://nodered.org/) to read metrics from the Klever node and to convert and write the data into an influxDB.
* [influxDB timeseries database](https://www.influxdata.com/products/influxdb/) to hold a history of the node metrics.
* [Grafana Dashboard](https://grafana.com/) to visualize the node status.

For running the applications a [Docker](https://www.docker.com/) environment is used.

## Install Docker

There are many many guidelines and tutorials on Docker out there. So I don't want to go into detail here and it is not the focus of this guideline.

My recommendation is to setup a docker swarm on a server (e.g. VPS) as described here: <https://dockerswarm.rocks/>

## Setup influxDB

Use the following docker-compose to install an influxDB OSS edition.

```
version: "3.5"

services:
  influxdb2:
    image: influxdb:latest
    network_mode: "bridge"
    container_name: influxdb2
    volumes:
      - influx-database:/var/lib/influxdb2
      - influx-config:/etc/influxdb2/
    environment:
      - DOCKER_INFLUXDB_INIT_MODE=setup
      - DOCKER_INFLUXDB_INIT_USERNAME=admin
      - DOCKER_INFLUXDB_INIT_PASSWORD=<your initial influx password>
      - DOCKER_INFLUXDB_INIT_ORG=organization
      - DOCKER_INFLUXDB_INIT_BUCKET=Default
      - DOCKER_INFLUXDB_INIT_RETENTION=1w
      - DOCKER_INFLUXDB_INIT_ADMIN_TOKEN=randomTokenValue123546978
volumes:
  influx-config:
    driver: local
  influx-database:
    driver: local
```

Unce installed, you have to enter the infuxDB user interface and create a new Bucker called `Klever`. Create also an API token for the bucket, that allows you to read and write to the bucket.

Find more information on the usage of influxDB in the official documentation: <https://docs.influxdata.com/influxdb/v2.1/get-started/>

## Setup Node-RED

Use the following docker-compose to install Node-RED or installed as described in the [official manual](https://nodered.org/docs/getting-started/)

```
version: '3.3'
services:
  node-red:
    image: nodered/node-red:latest
    environment:
     - TZ=Europe/Amsterdam
     - NODE_RED_CREDENTIAL_SECRET=SECRETKEY123456789
    volumes:
     - /home/<USER>/apps/nodered:/data
```

Use the following flow to extract the data from the Klever node and to convert it into the data format that needs to be send to the influxDB

![nodered_flow.png](/assets/02/nodered_flow.png)

You may need to install the `node-red-contrib-stackhero-influxdb-v2` via Manage-Palette (ALT+SHIFT+p or Menu->Manage Palette) first.

Here is also the flow for import:

```
[{"id":"932dceeaadd487ad","type":"http request","z":"a4e69f52d6c0c095","name":"","method":"GET","ret":"txt","paytoqs":"ignore","url":"","tls":"","persist":false,"proxy":"","authType":"","senderr":false,"x":1230,"y":1220,"wires":[["a7daf773897f9abb"]]},{"id":"862487eb2a927e2a","type":"function","z":"a4e69f52d6c0c095","name":"set node URL","func":"\n\nmsg.url = \"http://<your node ip>/node/status\"\n\nreturn msg;","outputs":1,"noerr":0,"initialize":"","finalize":"","libs":[],"x":1240,"y":1160,"wires":[["932dceeaadd487ad"]]},{"id":"a7daf773897f9abb","type":"json","z":"a4e69f52d6c0c095","name":"","property":"payload","action":"","pretty":false,"x":1210,"y":1280,"wires":[["cc4665e623a83686"]]},{"id":"57d248058dca0c5c","type":"inject","z":"a4e69f52d6c0c095","name":"Trigger","props":[],"repeat":"10","crontab":"","once":true,"onceDelay":0.1,"topic":"","x":1060,"y":1160,"wires":[["862487eb2a927e2a"]]},{"id":"cc4665e623a83686","type":"rbe","z":"a4e69f52d6c0c095","name":"","func":"rbe","gap":"","start":"","inout":"out","septopics":true,"property":"payload","topi":"topic","x":1350,"y":1280,"wires":[["39c67b6c76f69524"]]},{"id":"557439e6beddb5b9","type":"Stackhero-InfluxDB-v2-write","z":"a4e69f52d6c0c095","server":"c4cc3be7ae181489","name":"","x":1290,"y":1340,"wires":[[]]},{"id":"39c67b6c76f69524","type":"function","z":"a4e69f52d6c0c095","name":"Convert to influxDB Measurement","func":"msg.payload = {\n  // You bucket\n  // Optional (it can be defined in the node credentials settings)\n  bucket: 'Klever',\n\n  // Precision of timestamp\n  // Optional\n  // Can be `ns` (nanoseconds),\n  //        `us` (microseconds),\n  //        `ms` (milliseconds),\n  //        `s` (seconds).\n  // The default is `ns`\n  // Note: if you set the `timestamp` field to `Date.now()`, you have to set the `precision` to `ms`\n  precision: 'ms',\n\n  // Data to send to InfluxDB\n  // Can be an array of objects or only one object\n  data: [\n    {\n      measurement: 'klever-node-status',\n\n      tags: {\n        \"chain_id\": String(msg.payload.data.metrics.klv_chain_id),\n      },\n\n      fields: msg.payload.data.metrics,\n\n      timestamp: Date.now()\n    },\n\n    // More data can be send here, simply re add an object\n    // { ... },\n  ]\n};\n\nreturn msg;","outputs":1,"noerr":0,"initialize":"","finalize":"","libs":[],"x":1590,"y":1280,"wires":[["557439e6beddb5b9"]]},{"id":"215d2dfa46547903","type":"comment","z":"a4e69f52d6c0c095","name":"Feed influxDB with Klever Node Metrics","info":"","x":1100,"y":1120,"wires":[]},{"id":"c4cc3be7ae181489","type":"Stackhero-InfluxDB-v2-Server","name":"Influx (Klever)","host":"","port":"443","tls":true}]
```

Don't forget to adjust the IP of your node as well as of your influxDB.

## Setup Grafana

Once the data gets pumped from the node into the database via Node-RED, we can visualize the data via a Grafana Dashboard.

![grafana_dashboard.png](/assets/02/grafana_dashboard.png)

Add the influxDB as new data source in the sidebar. Then add a new dashboard via the import of the contents of [assets/02/dashboard.json](assets/02/dashboard.json).

