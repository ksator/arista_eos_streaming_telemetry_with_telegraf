# About this repository  

This repository shows the steps to demo streaming telemetry with Arista EOS devices and Telegraf.  

The building blocks are: 
  - EOS devices: There is a gNMI server in EOS devices.    
  - Telegraf: A gNMI client sends a `subscribe` RPC to request to targets (network devices with a gNMI server) to stream values for paths
  - Influxdb: Telegraf writes the data to Influxdb.   

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

# Telegraf configuration files 

To subscribe to both openconfig and native paths, the gNMI client must send 2 differents requests.  

We will use Telegraf.  

We will use two telegraf configuration files: 
  - [telegraf_openconfig.conf](telegraf_openconfig.conf)
  - [telegraf_eos.conf](telegraf_eos.conf) 

The configuration file [telegraf_openconfig.conf](telegraf_openconfig.conf) uses:
- a gNMI input plugin configured to subscribe to openconfig paths 
- the influxdb output plugin   
  
The configuration file [telegraf_eos.conf](telegraf_eos.conf) uses: 
- a gNMI input plugin configured to subscribe to EOS native paths 
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
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
grafana/grafana     7.0.3               22fccd4fab0a        6 days ago          158MB
telegraf            1.14.3              6fdd3e021713        2 weeks ago         251MB
influxdb            1.8.0               1bf862b66ac1        3 weeks ago         304MB
```

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
docker run -d --name telegraf_openconfig -v $PWD/telegraf_openconfig.conf:/etc/telegraf/telegraf.conf:ro --network=tig telegraf:1.14.3
docker run -d --name telegraf_eos -v $PWD/telegraf_eos.conf:/etc/telegraf/telegraf.conf:ro --network=tig telegraf:1.14.3
docker run -d --name grafana -p 3000:3000 --network=tig grafana/grafana:7.0.3
```
```
docker ps
CONTAINER ID        IMAGE                   COMMAND                  CREATED             STATUS              PORTS                                            NAMES
77311aa74c6a        telegraf:1.14.3         "/entrypoint.sh tele…"   6 seconds ago       Up 5 seconds        8092/udp, 8125/udp, 8094/tcp                     telegraf_eos
fd7e83343d23        telegraf:1.14.3         "/entrypoint.sh tele…"   17 seconds ago      Up 16 seconds       8092/udp, 8125/udp, 8094/tcp                     telegraf_openconfig
34df9c8620bd        influxdb:1.8.0          "/entrypoint.sh infl…"   22 seconds ago      Up 21 seconds       0.0.0.0:8083->8083/tcp, 0.0.0.0:8086->8086/tcp   influxdb
c818fb9ce85f        grafana/grafana:7.0.3   "/run.sh"                3 hours ago         Up 3 hours          0.0.0.0:3000->3000/tcp                           grafana
```
```
docker network inspect tig
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
            "34df9c8620bdbf3fd8245ff2d6d885a3d512dd630ae1ff1815c4fe2977a32058": {
                "Name": "influxdb",
                "EndpointID": "b80859b8e91a26f0530577ba37270699915b90948fe2316bbb4c73430d33c4a1",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            },
            "77311aa74c6a60eb36297c990beef9503aa2972e83d653ecb2dac719816e2d7c": {
                "Name": "telegraf_eos",
                "EndpointID": "4f9e474a15d8e02d94e4a849413990f366f08f988011bec3bd98d8e0316a8b49",
                "MacAddress": "02:42:ac:12:00:05",
                "IPv4Address": "172.18.0.5/16",
                "IPv6Address": ""
            },
            "c818fb9ce85fdb91fdbac91a2135b20d50bbba93ddc1fca453c85d81677f31f6": {
                "Name": "grafana",
                "EndpointID": "a471c77ee0afc1e953206f3eefe6996f1a23b82d930a3b36a99e1fe07b0d59b0",
                "MacAddress": "02:42:ac:12:00:04",
                "IPv4Address": "172.18.0.4/16",
                "IPv6Address": ""
            },
            "fd7e83343d23cbfda57b4fedaaff23a89c80618f21e89d08e72752c6c106c81c": {
                "Name": "telegraf_openconfig",
                "EndpointID": "38b8b1956d269a26596d3d4c7d017975c9857a4d2996d7572118613c13396d02",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```
# Telegraf logs
```
docker logs telegraf_eos
2020-06-09T23:11:41Z I! Starting Telegraf 1.14.3
2020-06-09T23:11:41Z I! Using config file: /etc/telegraf/telegraf.conf
2020-06-09T23:11:41Z I! Loaded inputs: cisco_telemetry_gnmi
2020-06-09T23:11:41Z I! Loaded aggregators: 
2020-06-09T23:11:41Z I! Loaded processors: 
2020-06-09T23:11:41Z I! Loaded outputs: influxdb
2020-06-09T23:11:41Z I! Tags enabled: host=77311aa74c6a
2020-06-09T23:11:41Z I! [agent] Config: Interval:10s, Quiet:false, Hostname:"77311aa74c6a", Flush Interval:10s
```
```
docker logs telegraf_openconfig
2020-06-09T23:11:30Z I! Starting Telegraf 1.14.3
2020-06-09T23:11:30Z I! Using config file: /etc/telegraf/telegraf.conf
2020-06-09T23:11:30Z I! Loaded inputs: cisco_telemetry_gnmi
2020-06-09T23:11:30Z I! Loaded aggregators: 
2020-06-09T23:11:30Z I! Loaded processors: 
2020-06-09T23:11:30Z I! Loaded outputs: influxdb
2020-06-09T23:11:30Z I! Tags enabled: host=fd7e83343d23
2020-06-09T23:11:30Z I! [agent] Config: Interval:10s, Quiet:false, Hostname:"fd7e83343d23", Flush Interval:10s
```




