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

# Create a docker nertwork 

```
docker network create tig
```
```
docker network ls
docker network inspect tig
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
docker network inspect tig
```




