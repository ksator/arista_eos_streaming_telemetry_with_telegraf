# Requirements on EOS devices (enable and allow gNMI):

```
enable
configure
username arista secret 0 arista
ip access-list GNMI
  10 permit tcp any any eq gnmi
management api gnmi
  transport grpc def
    ip access-group GNMI
```

# Telegraf configuration file 

The Telegraf configuration file [telegraf.conf](telegraf.conf) uses: 
  - a gNMI input plugin: it uses the `subscribe` RPC to request to targets (network devices with gNMI server) to stream values for paths
  - the influxdb output plugin 

So the devices stream data to Telegraf and Telegraf store the data to Influxdb.  

# 




