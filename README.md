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

We will use two telegraf configuration files: 
  - [telegraf_openconfig.conf](telegraf_openconfig.conf)
  - [telegraf_eos.conf](telegraf_eos.conf) 

The Telegraf configuration file [telegraf_openconfig.conf](telegraf_openconfig.conf) uses: 
  - a gNMI input plugin: it uses the `subscribe` RPC to request to targets (network devices with gNMI server) to stream values for paths
  - the influxdb output plugin 

So the devices stream data to Telegraf and Telegraf store the data to Influxdb.  

# 




