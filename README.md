![GitHub](https://img.shields.io/github/license/ksator/arista_eos_streaming_telemetry_with_Telegraf)    

# Tables of content. 

[About this repository](#about-this-repository)   
[building blocks](#building-blocks)   
[Requirements on EOS devices](#requirements-on-eos-devices)   
[Telegraf configuration file](#telegraf-configuration-file)   
[Install docker](#install-docker)   
[Pull the docker images](#pull-the-docker-images)   
[Create a docker network](#create-a-docker-network)   
[Create containers](#create-containers)   
[Display detailed information about the network](#display-detailed-information-about-the-network) 
[Telegraf logs](#telegraf-logs)   
[Query influxdb using CLI](#query-influxdb-using-cli)   
[Query Influxdb using Python](#query-Influxdb-using-python)   

# About this repository  

This repository shows the steps to demo streaming telemetry with Arista EOS devices and Telegraf.  

# building blocks 

  - EOS devices: There is a gNMI server in EOS devices.    
  - Telegraf: It has a gNMI client. It subscribes to paths on targets (network devices with a gNMI server)
  - Influxdb: Telegraf writes on Influxdb the data streamed from network devices.    

# Requirements on EOS devices 

Enable and allow gNMI:

```
enable
configure
username arista secret 0 arista
ip access-list GNMI
  10 permit tcp any any eq gnmi
management api gnmi
  transport grpc def
    ip access-group GNMI
   provider eos-native
```

`provider eos-native` is required to serve gNMI subscription requests to EOS native paths.  
So, using the above configuration, a gNMI client can subscribes to both openconfig and native paths.  

Note: To subscribe to both openconfig and native paths, the gNMI client must send 2 differents requests.  

# Telegraf configuration file

We will use the Telegraf configuration file [telegraf.conf](telegraf.conf). 
It uses: 
 - a gNMI input plugin configured to subscribe to openconfig and native paths 
 - the influxdb output plugin   

So the devices will stream openconfig and EOS native data to Telegraf and Telegraf will store the data to Influxdb.  

# Install docker

```
docker -v
Docker version 19.03.8, build afacb8b
```

# Pull the docker images

```
docker pull telegraf:1.14.3
docker pull influxdb:1.8.0
docker pull grafana/grafana:7.0.3
```

```
docker images
```
<details><summary>click me to see the response</summary>
<p>
  
```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
grafana/grafana     7.0.3               22fccd4fab0a        6 days ago          158MB
telegraf            1.14.3              6fdd3e021713        2 weeks ago         251MB
influxdb            1.8.0               1bf862b66ac1        3 weeks ago         304MB
```
</p>
</details>


# Create a docker network 

```
docker network create tig
```
```
docker network ls
```

# Create containers 

```
docker run -d --name influxdb -p 8083:8083 -p 8086:8086 --network=tig influxdb:1.8.0
docker run -d --name telegraf -v $PWD/telegraf.conf:/etc/telegraf/telegraf.conf:ro --network=tig telegraf:1.14.3
docker run -d --name grafana -p 3000:3000 --network=tig grafana/grafana:7.0.3
```
```
docker ps
```
<details><summary>click me to see the response</summary>
<p>
  
```
CONTAINER ID        IMAGE                   COMMAND                  CREATED             STATUS              PORTS                                            NAMES
bfe273b6b299        telegraf:1.14.3         "/entrypoint.sh tele…"   12 seconds ago      Up 11 seconds       8092/udp, 8125/udp, 8094/tcp                     telegraf
c3ead2edcf5a        influxdb:1.8.0          "/entrypoint.sh infl…"   20 seconds ago      Up 18 seconds       0.0.0.0:8083->8083/tcp, 0.0.0.0:8086->8086/tcp   influxdb
c818fb9ce85f        grafana/grafana:7.0.3   "/run.sh"                4 hours ago         Up 4 hours          0.0.0.0:3000->3000/tcp                           grafana
```
</p>
</details>

# Display detailed information about the network 

```
docker network inspect tig
```
<details><summary>click me to see the output</summary>
<p>

```
[
    {
        "Name": "tig",
        "Id": "8c56cd4a208c8f64556963f725af786dd3f3c14ac458db503c82574307d51c0b",
        "Created": "2020-06-01T20:31:43.5383398Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "bfe273b6b2992021b911f442acca4980ce19b46e6bc0865dcba354cf8fcf0121": {
                "Name": "telegraf",
                "EndpointID": "f6dfac8660c257b437a2b5b218ee67142961c7fc087f11b664c982052b4cee98",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            },
            "c3ead2edcf5a52303f66cf38fc9a24dd80f8b50e6819357bde4541798a825308": {
                "Name": "influxdb",
                "EndpointID": "6e5bf29fba94b2db65af929c2682317b6538292250ca04097697f8659f52ab40",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            },
            "c818fb9ce85fdb91fdbac91a2135b20d50bbba93ddc1fca453c85d81677f31f6": {
                "Name": "grafana",
                "EndpointID": "a471c77ee0afc1e953206f3eefe6996f1a23b82d930a3b36a99e1fe07b0d59b0",
                "MacAddress": "02:42:ac:12:00:04",
                "IPv4Address": "172.18.0.4/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```
</p>
</details>

# Telegraf logs

```
docker logs telegraf
```
<details><summary>click me to see the output</summary>
<p>

```
2020-06-09T23:46:34Z I! Starting Telegraf 1.14.3
2020-06-09T23:46:34Z I! Using config file: /etc/telegraf/telegraf.conf
2020-06-09T23:46:34Z I! Loaded inputs: cisco_telemetry_gnmi cisco_telemetry_gnmi
2020-06-09T23:46:34Z I! Loaded aggregators: 
2020-06-09T23:46:34Z I! Loaded processors: 
2020-06-09T23:46:34Z I! Loaded outputs: influxdb
2020-06-09T23:46:34Z I! Tags enabled: host=bfe273b6b299
2020-06-09T23:46:34Z I! [agent] Config: Interval:10s, Quiet:false, Hostname:"bfe273b6b299", Flush Interval:10s
```
</p>
</details>

# Query influxdb using CLI

```
docker exec -it influxdb bash

root@c3ead2edcf5a:/# influx
Connected to http://localhost:8086 version 1.8.0
InfluxDB shell version: 1.8.0
```
```
> SHOW DATABASES
```
<details><summary>click me to see the response</summary>
<p>

```
name: databases
name
----
arista
_internal
```
</p>
</details>

```
> USE arista
Using database arista
```
```
> SHOW MEASUREMENTS
```
<details><summary>click me to see the response</summary>
<p>

```
name: measurements
name
----
eos_bgp
ifcounters
openconfig_bgp
> 
```
</p>
</details>

Query ifcounters measurement 

```
> SHOW TAG KEYS FROM "ifcounters"
```
<details><summary>click me to see the response</summary>
<p>

```

name: ifcounters
tagKey
------
host
name
source
```
</p>
</details>

```
> SHOW TAG VALUES FROM "ifcounters" with KEY = "name"
```
<details><summary>click me to see the response</summary>
<p>

```
name: ifcounters
key  value
---  -----
name Ethernet1
name Ethernet10
name Ethernet11
name Ethernet12
name Ethernet13
name Ethernet14
name Ethernet15
name Ethernet16
name Ethernet17
name Ethernet18
name Ethernet19
name Ethernet2
name Ethernet20
name Ethernet21
name Ethernet22
name Ethernet23
name Ethernet24
name Ethernet25
name Ethernet26
name Ethernet27
name Ethernet28
name Ethernet29
name Ethernet3
name Ethernet30
name Ethernet31
name Ethernet32
name Ethernet33
name Ethernet34
name Ethernet35
name Ethernet36
name Ethernet37
name Ethernet38
name Ethernet39
name Ethernet4
name Ethernet40
name Ethernet41
name Ethernet42
name Ethernet43
name Ethernet44
name Ethernet45
name Ethernet46
name Ethernet47
name Ethernet48
name Ethernet49/1
name Ethernet49/2
name Ethernet49/3
name Ethernet49/4
name Ethernet5
name Ethernet50/1
name Ethernet50/2
name Ethernet50/3
name Ethernet50/4
name Ethernet51/1
name Ethernet51/2
name Ethernet51/3
name Ethernet51/4
name Ethernet52/1
name Ethernet52/2
name Ethernet52/3
name Ethernet52/4
name Ethernet6
name Ethernet7
name Ethernet8
name Ethernet9
name Management1
```
</p>
</details>

```
> SHOW TAG VALUES FROM "ifcounters" with KEY = "source"
```
<details><summary>click me to see the response</summary>
<p>

```

name: ifcounters
key    value
---    -----
source 10.83.28.122
source 10.83.28.125
```
</p>
</details>

```
> SHOW SERIES FROM "ifcounters"
```
<details><summary>click me to see the output</summary>
<p>

```
key
---
ifcounters,host=bfe273b6b299,name=Ethernet1,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet1,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet10,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet10,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet11,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet11,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet12,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet12,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet13,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet13,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet14,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet14,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet15,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet15,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet16,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet16,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet17,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet17,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet18,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet18,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet19,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet19,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet2,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet2,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet20,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet20,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet21,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet21,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet22,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet22,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet23,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet23,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet24,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet24,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet25,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet25,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet26,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet26,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet27,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet27,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet28,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet28,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet29,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet29,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet3,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet3,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet30,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet30,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet31,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet31,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet32,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet32,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet33,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet33,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet34,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet34,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet35,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet35,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet36,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet36,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet37,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet37,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet38,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet38,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet39,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet39,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet4,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet4,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet40,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet40,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet41,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet41,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet42,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet42,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet43,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet43,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet44,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet44,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet45,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet45,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet46,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet46,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet47,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet47,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet48,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet48,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet49/1,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet49/1,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet49/2,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet49/2,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet49/3,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet49/3,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet49/4,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet49/4,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet5,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet5,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet50/1,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet50/1,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet50/2,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet50/2,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet50/3,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet50/3,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet50/4,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet50/4,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet51/1,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet51/1,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet51/2,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet51/2,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet51/3,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet51/3,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet51/4,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet51/4,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet52/1,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet52/1,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet52/2,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet52/2,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet52/3,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet52/3,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet52/4,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet52/4,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet6,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet6,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet7,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet7,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet8,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet8,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Ethernet9,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet9,source=10.83.28.125
ifcounters,host=bfe273b6b299,name=Management1,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Management1,source=10.83.28.125
```
</p>
</details>

```
> SHOW SERIES FROM "ifcounters" WHERE "source" = '10.83.28.122'
```
<details><summary>click me to see the response</summary>
<p>
  
```
key
---
ifcounters,host=bfe273b6b299,name=Ethernet1,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet10,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet11,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet12,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet13,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet14,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet15,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet16,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet17,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet18,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet19,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet2,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet20,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet21,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet22,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet23,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet24,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet25,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet26,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet27,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet28,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet29,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet3,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet30,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet31,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet32,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet33,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet34,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet35,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet36,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet37,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet38,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet39,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet4,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet40,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet41,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet42,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet43,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet44,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet45,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet46,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet47,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet48,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet49/1,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet49/2,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet49/3,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet49/4,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet5,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet50/1,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet50/2,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet50/3,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet50/4,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet51/1,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet51/2,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet51/3,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet51/4,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet52/1,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet52/2,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet52/3,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet52/4,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet6,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet7,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet8,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Ethernet9,source=10.83.28.122
ifcounters,host=bfe273b6b299,name=Management1,source=10.83.28.122
```
</p>
</details>

```
> SHOW SERIES EXACT CARDINALITY ON arista
```
<details><summary>click me to see the response</summary>
<p>
  
```
name: eos_bgp
count
-----
24

name: ifcounters
count
-----
130

name: openconfig_bgp
count
-----
22
```
</p>
</details>

```
> SHOW SERIES EXACT CARDINALITY ON arista FROM "ifcounters" WHERE "source" = '10.83.28.122'
```
<details><summary>click me to see the response</summary>
<p>

```
name: ifcounters
count
-----
65
> 
```
</p>
</details>

```
> SELECT * FROM "ifcounters" WHERE "source" = '10.83.28.122'  ORDER BY DESC LIMIT 3
```
<details><summary>click me to see the response</summary>
<p>

```
name: ifcounters
time                host         in_broadcast_pkts in_discards in_errors in_multicast_pkts in_octets in_unicast_pkts name        out_broadcast_pkts out_discards out_errors out_multicast_pkts out_octets out_unicast_pkts source
----                ----         ----------------- ----------- --------- ----------------- --------- --------------- ----        ------------------ ------------ ---------- ------------------ ---------- ---------------- ------
1591747175792506596 bfe273b6b299                                                           286938640                 Management1                                                                                           10.83.28.122
1591747175792493464 bfe273b6b299 243616                                                                              Management1                                                                                           10.83.28.122
1591747175792483320 bfe273b6b299                                                                                     Management1                                                                          5768915          10.83.28.122
> 
```
</p>
</details>

```
> SELECT "in_octets","out_octets", "name" FROM "ifcounters" WHERE "source" = '10.83.28.122' ORDER BY DESC LIMIT 3
```
<details><summary>click me to see the response</summary>
<p>

```
name: ifcounters
time                in_octets out_octets name
----                --------- ---------- ----
1591747265864892858           6146169517 Management1
1591747265864883043 289180556            Management1
1591747257622763374           32223976   Ethernet4
> 
```
</p>
</details>

```
> SELECT "in_octets","out_octets" FROM "ifcounters" WHERE ("source" = '10.83.28.122' AND "name"='Ethernet24') ORDER BY DESC LIMIT 3
```
<details><summary>click me to see the response</summary>
<p>

```
name: ifcounters
time                in_octets out_octets
----                --------- ----------
1591747285684621082 362551    
1591747285684577516           352034
1591747269675996035 362325    
> 
```
</p>
</details>

```
> SELECT "in_octets","out_octets" FROM "ifcounters" WHERE ("source" = '10.83.28.122' AND "name"='Ethernet24' AND time >= now() - 60s) 
```
<details><summary>click me to see the response</summary>
<p>

```
name: ifcounters
time                in_octets out_octets
----                --------- ----------
1591747315702875678           352260
1591747315702934394 362777    
1591747329711620129           352349
1591747329711708507 362936    
1591747345721372958 363162    
1591747345721389797           352575
> 
```
</p>
</details>

```
> SELECT "in_octets","out_octets","name" FROM "ifcounters" WHERE ("source" = '10.83.28.122' AND "name" =~/Ethernet.*/ AND time >= now() - 60s) 
```
<details><summary>click me to see the response</summary>
<p>

```
name: ifcounters
time                in_octets out_octets name
----                --------- ---------- ----
1591747315673205906           32224513   Ethernet4
1591747315702875678           352260     Ethernet24
1591747315702934394 362777               Ethernet24
1591747329711620129           352349     Ethernet24
1591747329711708507 362936               Ethernet24
1591747343692188058 32267765             Ethernet4
1591747345693238453           32224737   Ethernet4
1591747345721372958 363162               Ethernet24
1591747345721389797           352575     Ethernet24
> 
```
</p>
</details>

```
> SELECT "in_octets","out_octets","name" FROM "ifcounters" WHERE ("source" = '10.83.28.122' AND "name" =~/Ethernet(1|24)/ AND time >= now() - 60s) 
```
<details><summary>click me to see the response</summary>
<p>

```
name: ifcounters
time                in_octets out_octets name
----                --------- ---------- ----
1591747435787448088 363999               Ethernet24
1591747435787483071           353342     Ethernet24
1591747449794133096 364158               Ethernet24
1591747449794171514           353431     Ethernet24
1591747465803270196           353657     Ethernet24
1591747465803323139 364384               Ethernet24
> 
```
</p>
</details>

```
> SELECT mean("in_octets") FROM "ifcounters" WHERE ("source" = '10.83.28.122' AND "name" = 'Ethernet24' AND time >= now() - 60s) 
```
<details><summary>click me to see the response</summary>
<p>

```
name: ifcounters
time                mean
----                ----
1591747439566953700 364384
> 
```
</p>
</details>

```
> SELECT mean("in_octets")*8 FROM "ifcounters" WHERE ("source" = '10.83.28.122' AND "name" = 'Ethernet24' AND time >= now() - 60s) 
```
<details><summary>click me to see the response</summary>
<p>

```
name: ifcounters
time                mean
----                ----
1591747477741147700 2918330.6666666665
> 
```
</p>
</details>

```
> SELECT derivative(mean("in_octets"), 1s) *8 FROM "ifcounters" WHERE ("name" = 'Ethernet24') AND ("source" = '10.83.28.122') AND (time >= now() - 10m)  GROUP BY time(1m) 
```
<details><summary>click me to see the response</summary>
<p>

```
name: ifcounters
time                derivative
----                ----------
1591746960000000000 81.46666666666667
1591747020000000000 81.46666666666667
1591747080000000000 81.46666666666667
1591747140000000000 81.46666666666667
1591747200000000000 81.46666666666667
1591747260000000000 81.46666666666667
1591747320000000000 81.46666666666667
1591747380000000000 81.46666666666667
1591747440000000000 81.46666666666667
1591747500000000000 81.46666666666667
> 
```
</p>
</details>

```
> SELECT derivative(mean("in_unicast_pkts"), 1s) FROM "ifcounters" WHERE ("source" = '10.83.28.122' AND "name" = 'Ethernet4') AND (time >= now() - 10m)  GROUP BY time(1m) 
```
<details><summary>click me to see the response</summary>
<p>

```
name: ifcounters
time                derivative
----                ----------
1591746960000000000 0.03333333333333333
1591747020000000000 0.03333333333333333
1591747080000000000 0.03333333333333333
1591747140000000000 0.03333333333333333
1591747200000000000 0.03333333333333333
1591747260000000000 0.03333333333333333
1591747320000000000 0.03333333333333333
1591747380000000000 0.03333333333333333
1591747440000000000 0.03333333333333333
1591747500000000000 0.03333333333333333
> 
```
</p>
</details>


Query openconfig_bgp measurement 


# Query Influxdb using Python

```
python -V
Python 3.7.7
```
```
pip install influxdb
```
```
pip freeze | grep influxdb
influxdb==5.3.0
```
```
python
Python 3.7.7 (default, Mar 10 2020, 15:43:33) 
[Clang 11.0.0 (clang-1100.0.33.17)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> 
>>> from influxdb import InfluxDBClient
>>> 
>>> influx_client = InfluxDBClient('localhost',8086)
>>> 
>>> influx_client.query('show databases')
ResultSet({'('databases', None)': [{'name': 'arista'}, {'name': '_internal'}]})
>>> 
>>> influx_client.query('show measurements', database='arista')
ResultSet({'('measurements', None)': [{'name': 'eos_bgp'}, {'name': 'ifcounters'}, {'name': 'openconfig_bgp'}]})
>>> 
>>> points = influx_client.query("""SELECT "in_octets" FROM "ifcounters" WHERE ("source" = '10.83.28.122' AND "name"='Ethernet24') ORDER BY DESC LIMIT 3""", database='arista').get_points()
>>> for point in points:
...     print(point['in_octets'])
... 
397378
397152
396993
>>> 
>>> exit()
```
