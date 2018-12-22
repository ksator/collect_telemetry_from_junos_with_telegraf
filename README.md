# About this repo

collect openconfig telemetry from junos devices using telegraf.  
store collected data in influxdb  
query influxdb database with cli and python to extract data   

# About Telegraf

Telegraf is an open source collector written in GO.  
Telegraf collects data and writes them into a database.  
It is plugin-driven (it has input plugins, output plugins, ...)  
We will use jti_openconfig_telemetry input plugin (grpc client to collect telemetry on junos devices) and influxb output plugin (database to store the data collected)  


# About Influxdb

Influxdb is an open source time series database written in GO.

# Requirements 

## Docker

you need to install docker.  
This is not covered in this repository

## Junos 

### Junos packages 

In order to collect data from Junos using openconfig telemetry, the devices require the Junos packages ```openconfig``` and ```network agent```

Starting with Junos OS Release 18.3R1: 
- the Junos OS image includes the ```OpenConfig``` package; therefore, you do not need anymore to install ```OpenConfig``` separately on your device.  
- the Junos OS image includes the ```Network Agent```, therefore, you do not need anymore to install the ```network agent``` separately on your device.  

This setup is using an older Junos release, so it is required to install these two packages. 

Here's the details:
```
jcluser@vMX1> show version | match "Junos:|openconfig|na telemetry"
Junos: 18.2R1.9
JUNOS na telemetry [18.2R1-S3.2-C1]
JUNOS Openconfig [0.0.0.10-1]
```

### Junos configuration

```
jcluser@vMX-1> show configuration system services netconf | display set
set system services netconf ssh
```
```
jcluser@vMX-1> show configuration system services extension-service | display set
set system services extension-service request-response grpc clear-text port 32768
set system services extension-service request-response grpc skip-authentication
set system services extension-service notification allow-clients address 0.0.0.0/0
```

# YANG modules

## OpenConfig YANG modules on Github

Openconfig yang modules https://github.com/openconfig/public/tree/master/release/models  
Each yang module has a version (`reference`), this is indicated in the YANG module

## YANG modules on Junos

Run this command to show YANG packages installed on Junos: 
```
jcluser@vMX-1> show system yang package
Package ID            :junos-openconfig
YANG Module(s)        :iana-if-type.yang ietf-inet-types.yang ietf-interfaces.yang ietf-yang-types.yang jnx-aug-openconfig-bgp.yang jnx-aug-openconfig-if-ip.yang jnx-aug-openconfig-interfaces.yang jnx-aug-openconfig-isis.yang jnx-aug-openconfig-lacp.yang jnx-aug-openconfig-lldp.yang jnx-aug-openconfig-local-routing.yang jnx-aug-openconfig-mpls.yang jnx-aug-openconfig-ni.yang jnx-aug-openconfig-routing-policy.yang jnx-openconfig-dev.yang junos-extension.yang openconfig-bgp-common-multiprotocol.yang openconfig-bgp-common-structure.yang openconfig-bgp-common.yang openconfig-bgp-global.yang openconfig-bgp-neighbor.yang openconfig-bgp-peer-group.yang openconfig-bgp-policy.yang openconfig-bgp-types.yang openconfig-bgp.yang openconfig-extensions.yang openconfig-if-aggregate.yang openconfig-if-ethernet.yang openconfig-if-ip-ext.yang openconfig-if-ip.yang openconfig-inet-types.yang openconfig-interfaces.yang openconfig-isis-lsdb-types.yang openconfig-isis-lsp.yang openconfig-isis-policy.yang openconfig-isis-routing.yang openconfig-isis-types.yang openconfig-isis.yang openconfig-lacp.yang openconfig-lldp-types.yang openconfig-lldp.yang openconfig-local-routing.yang openconfig-mpls-igp.yang openconfig-mpls-ldp.yang openconfig-mpls-rsvp.yang openconfig-mpls-sr.yang openconfig-mpls-static.yang openconfig-mpls-te.yang openconfig-mpls-types.yang openconfig-mpls.yang openconfig-network-instance-l2.yang openconfig-network-instance-l3.yang openconfig-network-instance-types.yang openconfig-network-instance.yang openconfig-platform-transceiver.yang openconfig-platform-types.yang openconfig-platform.yang openconfig-policy-types.yang openconfig-rib-bgp-ext.yang openconfig-rib-bgp-types.yang openconfig-rib-bgp.yang openconfig-routing-policy.yang openconfig-segment-routing.yang openconfig-terminal-device.yang openconfig-transport-types.yang openconfig-types.yang openconfig-vlan-types.yang openconfig-vlan.yang openconfig-yang-types.yang
Translation Script(s) :openconfig-bgp.slax openconfig-interface.slax openconfig-lldp.slax openconfig-local-routing.slax openconfig-mpls.slax openconfig-network-instance.slax openconfig-ni-bgp.slax openconfig-ni-mpls.slax openconfig-policy.slax openconfig-vlan.slax
Translation script status is enabled
```
Run this command to list YANG modules available on Junos: 
```
jcluser@vMX-1> file list /opt/yang-pkg/junos-openconfig/yang/

/opt/yang-pkg/junos-openconfig/yang/:
deviation/
iana-if-type.yang
ietf-inet-types.yang
ietf-interfaces.yang
ietf-yang-types.yang
jnx-aug-openconfig-bgp.yang
jnx-aug-openconfig-if-ip.yang
jnx-aug-openconfig-interfaces.yang
jnx-aug-openconfig-isis.yang
jnx-aug-openconfig-lacp.yang
jnx-aug-openconfig-lldp.yang
jnx-aug-openconfig-local-routing.yang
jnx-aug-openconfig-mpls.yang
jnx-aug-openconfig-ni.yang
jnx-aug-openconfig-routing-policy.yang
jnx-openconfig-dev.yang@ -> /opt/yang-pkg/junos-openconfig/yang/deviation/jnx-openconfig-dev.yang
junos-extension.yang
openconfig-bgp-common-multiprotocol.yang
openconfig-bgp-common-structure.yang
openconfig-bgp-common.yang
openconfig-bgp-global.yang
openconfig-bgp-neighbor.yang
openconfig-bgp-peer-group.yang
openconfig-bgp-policy.yang
openconfig-bgp-types.yang
openconfig-bgp.yang
openconfig-extensions.yang
openconfig-if-aggregate.yang
openconfig-if-ethernet.yang
openconfig-if-ip-ext.yang
openconfig-if-ip.yang
openconfig-inet-types.yang
openconfig-interfaces.yang
openconfig-isis-lsdb-types.yang
openconfig-isis-lsp.yang
openconfig-isis-policy.yang
openconfig-isis-routing.yang
openconfig-isis-types.yang
openconfig-isis.yang
openconfig-lacp.yang
openconfig-lldp-types.yang
openconfig-lldp.yang
openconfig-local-routing.yang
openconfig-mpls-igp.yang
openconfig-mpls-ldp.yang
openconfig-mpls-rsvp.yang
openconfig-mpls-sr.yang
openconfig-mpls-static.yang
openconfig-mpls-te.yang
openconfig-mpls-types.yang
openconfig-mpls.yang
openconfig-network-instance-l2.yang
openconfig-network-instance-l3.yang
openconfig-network-instance-types.yang
openconfig-network-instance.yang
openconfig-platform-transceiver.yang
openconfig-platform-types.yang
openconfig-platform.yang
openconfig-policy-types.yang
openconfig-rib-bgp-ext.yang
openconfig-rib-bgp-types.yang
openconfig-rib-bgp.yang
openconfig-routing-policy.yang
openconfig-segment-routing.yang
openconfig-terminal-device.yang
openconfig-transport-types.yang
openconfig-types.yang
openconfig-vlan-types.yang
openconfig-vlan.yang
openconfig-yang-types.yang
```
Run this command to know which `reference` of a YANG module is used on a Junos device.   
Example with openconfig-interfaces.yang YANG module
```
jcluser@vMX-1> file more /opt/yang-pkg/junos-openconfig/yang/openconfig-interfaces.yang
```
Run this command to understand which YANG deviations are used on a Junos device:
```
jcluser@vMX-1> file more /opt/yang-pkg/junos-openconfig/yang/jnx-openconfig-dev.yang
```

## Pyang

You can use pyang to:
 - Validate YANG modules against YANG RFCs
 - Convert YANG modules into equivalent YIN module (XML).
 - Generate a tree representation of YANG models for quick visualization.

### Pyang installation

pyang installation from pypi: 
```
$ pip install pyang
```


### validate yang modules

Example: let's validate the modules openconfig-bgp.yang and openconfig-interfaces.yang from openconfig github repository

#### Get Openconfig yang modules 
clone the openconfig repository
```
$ git clone https://github.com/openconfig/public.git
$ ls public
```
#### move all the yang files from Openconfig to the same directory

The YANG modules openconfig-bgp.yang and openconfig-interfaces.yang import other YANG modules: so we need to make sure pyang can find all these yang modules 

move all the yang files to the directory yang_modules
```
$ mkdir yang_modules
$ cp public/release/models/*.yang yang_modules/.
$ cp -R public/release/models/*/*.yang yang_modules/.
$ ls yang_modules
$ cd yang_modules
```

#### validate yang modules

```
$ pyang openconfig-bgp.yang
$ pyang openconfig-interfaces.yang 
```

### Generate a tree representation of YANG modules for quick visualization

```
$ pyang -f tree openconfig-bgp.yang 
```
```
$ pyang -f tree openconfig-interfaces.yang
```
```
$ pyang openconfig-interfaces.yang -f tree --tree-path=/interfaces/interface/state
module: openconfig-interfaces
  +--rw interfaces
     +--rw interface* [name]
        +--ro state
           +--ro name?            string
           +--ro type             identityref
           +--ro mtu?             uint16
           +--ro loopback-mode?   boolean
           +--ro description?     string
           +--ro enabled?         boolean
           +--ro ifindex?         uint32
           +--ro admin-status     enumeration
           +--ro oper-status      enumeration
           +--ro last-change?     oc-types:timeticks64
           +--ro logical?         boolean
           +--ro counters
              +--ro in-octets?             oc-yang:counter64
              +--ro in-pkts?               oc-yang:counter64
              +--ro in-unicast-pkts?       oc-yang:counter64
              +--ro in-broadcast-pkts?     oc-yang:counter64
              +--ro in-multicast-pkts?     oc-yang:counter64
              +--ro in-discards?           oc-yang:counter64
              +--ro in-errors?             oc-yang:counter64
              +--ro in-unknown-protos?     oc-yang:counter64
              +--ro in-fcs-errors?         oc-yang:counter64
              +--ro out-octets?            oc-yang:counter64
              +--ro out-pkts?              oc-yang:counter64
              +--ro out-unicast-pkts?      oc-yang:counter64
              +--ro out-broadcast-pkts?    oc-yang:counter64
              +--ro out-multicast-pkts?    oc-yang:counter64
              +--ro out-discards?          oc-yang:counter64
              +--ro out-errors?            oc-yang:counter64
              +--ro carrier-transitions?   oc-yang:counter64
              +--ro last-clear?            oc-types:timeticks64
```
Old openconfig BGP path, used by Junos 16.1R3 to 17.2
```
$ pyang openconfig-bgp.yang -f tree --tree-path=/bgp/neighbors/neighbor/state --tree-depth=5
module: openconfig-bgp
  +--rw bgp
     +--rw neighbors
        +--rw neighbor* [neighbor-address]
           +--ro state
              +--ro peer-group?                -> ../../../../peer-groups/peer-group/peer-group-name
              +--ro neighbor-address?          oc-inet:ip-address
              +--ro enabled?                   boolean
              +--ro peer-as?                   oc-inet:as-number
              +--ro local-as?                  oc-inet:as-number
              +--ro peer-type?                 oc-bgp-types:peer-type
              +--ro auth-password?             oc-types:routing-password
              +--ro remove-private-as?         oc-bgp-types:remove-private-as-option
              +--ro route-flap-damping?        boolean
              +--ro send-community?            oc-bgp-types:community-type
              +--ro description?               string
              +--ro session-state?             enumeration
              +--ro last-established?          oc-types:timeticks64
              +--ro established-transitions?   oc-yang:counter64
              +--ro supported-capabilities*    identityref
              +--ro messages
              |     ...
              +--ro queues
              |     ...
              +--ro dynamically-configured?    boolean

```
New openconfig BGP path, used from Junos 17.3

```
$ pyang openconfig-network-instance.yang --tree-path=/network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/state -f tree
module: openconfig-network-instance
  +--rw network-instances
     +--rw network-instance* [name]
        +--rw protocols
           +--rw protocol* [identifier name]
              +--rw bgp
                 +--rw neighbors
                    +--rw neighbor* [neighbor-address]
                       +--ro state
                          +--ro peer-group?                -> ../../../../peer-groups/peer-group/peer-group-name
                          +--ro neighbor-address?          oc-inet:ip-address
                          +--ro enabled?                   boolean
                          +--ro peer-as?                   oc-inet:as-number
                          +--ro local-as?                  oc-inet:as-number
                          +--ro peer-type?                 oc-bgp-types:peer-type
                          +--ro auth-password?             oc-types:routing-password
                          +--ro remove-private-as?         oc-bgp-types:remove-private-as-option
                          +--ro route-flap-damping?        boolean
                          +--ro send-community?            oc-bgp-types:community-type
                          +--ro description?               string
                          +--ro session-state?             enumeration
                          +--ro last-established?          oc-types:timeticks64
                          +--ro established-transitions?   oc-yang:counter64
                          +--ro supported-capabilities*    identityref
                          +--ro messages
                          |  +--ro sent
                          |  |  +--ro UPDATE?                            uint64
                          |  |  +--ro NOTIFICATION?                      uint64
                          |  |  +--ro last-notification-time?            oc-types:timeticks64
                          |  |  +--ro last-notification-error-code?      identityref
                          |  |  +--ro last-notification-error-subcode?   identityref
                          |  +--ro received
                          |     +--ro UPDATE?                            uint64
                          |     +--ro NOTIFICATION?                      uint64
                          |     +--ro last-notification-time?            oc-types:timeticks64
                          |     +--ro last-notification-error-code?      identityref
                          |     +--ro last-notification-error-subcode?   identityref
                          +--ro queues
                          |  +--ro input?    uint32
                          |  +--ro output?   uint32
                          +--ro dynamically-configured?    boolean
```


# Influxdb

pull docker images 
```
$ docker pull influxdb
```
Verify
```
$ docker images influxdb
```
Instanciate an influxdb container
```
$ docker run -d --name influxdb -p 8083:8083 -p 8086:8086 influxdb
```
Verify
```
$ docker ps | grep influxdb
```
for troubleshooting purposes you can run this command
```
$ docker logs influxdb
```
start a shell session in the influxdb container
```
$ docker exec -it influxdb bash
```
run this command to read the influxdb configuration file
```
# more /etc/influxdb/influxdb.conf
```
create a user and a database
```
# influx
Connected to http://localhost:8086 version 1.7.0
InfluxDB shell version: 1.7.0
Enter an InfluxQL query
> CREATE DATABASE juniper
> show databases
name: databases
name
----
_internal
juniper
>  CREATE USER "juniper" WITH PASSWORD 'juniper'
> show users
user   admin
----   -----
influx false
> exit
# 
```
exit the influxdb container
```
# exit
```

# Telegraf

get ip address used by containers
```
$ ifconfig docker0
```

pull docker images 
```
$ docker pull telegraf
```
Verify
```
$ docker images telegraf
```
create a telegraf configuration file ([use this file](telegraf.conf))  
it will use jti_openconfig_telemetry input plugin (grpc client to collect telemetry on junos devices) and influxb output plugin (database to store the data collected)  

```
$ vi telegraf.conf
```
instanciate a telegraf container
```
$ docker run --name telegraf -d -v $PWD/telegraf.conf:/etc/telegraf/telegraf.conf:ro telegraf
```
verify
```
$ docker ps | grep telegraf
```
for troubleshooting purposes you can run this command
```
$ docker logs telegraf
```
start a shell session in the telegraf container
```
$ docker exec -it telegraf bash
```
verify the telegraf configuration file
```
# more /etc/telegraf/telegraf.conf
```
exit the telegraf container
```
# exit
```
# query the influxdb database with cli to verify
start a shell session in the influxdb container
```
$ docker exec -it influxdb bash
```
query the database
```
# influx
Connected to http://localhost:8086 version 1.7.0
InfluxDB shell version: 1.7.0
Enter an InfluxQL query
> show databases
name: databases
name
----
_internal
juniper
> use juniper
Using database juniper
> show measurements
name: measurements
name
----
```
Devices streaming data
```
> SHOW TAG VALUES FROM  "/network-instances/network-instance/protocols/protocol/bgp/" with KEY = "device"
name: /network-instances/network-instance/protocols/protocol/bgp/
key    value
---    -----
device 100.123.1.0
device 100.123.1.1
device 100.123.1.2
device 100.123.1.4
device 100.123.1.5
device 100.123.1.6
>
```
BGP neighbors address
```
> SHOW TAG VALUES FROM  "/network-instances/network-instance/protocols/protocol/bgp/" with KEY = "/network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/@neighbor-address"
name: /network-instances/network-instance/protocols/protocol/bgp/
key                                                                                             value
---                                                                                             -----
/network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/@neighbor-address 192.168.1.1
/network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/@neighbor-address 192.168.1.2
/network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/@neighbor-address 192.168.1.3
/network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/@neighbor-address 192.168.1.4
/network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/@neighbor-address 192.168.1.5
/network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/@neighbor-address 192.168.1.6
/network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/@neighbor-address 192.168.1.7
/network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/@neighbor-address 192.168.2.1
/network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/@neighbor-address 192.168.2.2
/network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/@neighbor-address 192.168.2.3
/network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/@neighbor-address 192.168.2.4
/network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/@neighbor-address 192.168.2.5
/network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/@neighbor-address 192.168.2.6
/network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/@neighbor-address 192.168.2.7
/network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/@neighbor-address 192.168.3.1
/network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/@neighbor-address 192.168.3.2
/network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/@neighbor-address 192.168.3.3
/network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/@neighbor-address 192.168.3.4
/network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/@neighbor-address 192.168.3.5
/network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/@neighbor-address 192.168.3.6
/network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/@neighbor-address 192.168.3.7
```

BGP neighbors address for device 100.123.1.0
```
> SHOW TAG VALUES FROM  "/network-instances/network-instance/protocols/protocol/bgp/" with KEY = "/network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/@neighbor-address" WHERE device='100.123.1.0'
name: /network-instances/network-instance/protocols/protocol/bgp/
key                                                                                             value
---                                                                                             -----
/network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/@neighbor-address 192.168.1.1
/network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/@neighbor-address 192.168.1.3
/network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/@neighbor-address 192.168.1.5
/network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/@neighbor-address 192.168.1.7
>
```
BGP groups for device 100.123.1.0
```
> SHOW TAG VALUES FROM  "/network-instances/network-instance/protocols/protocol/bgp/" with KEY ="/network-instances/network-instance/protocols/protocol/bgp/peer-groups/peer-group/@peer-group-name" WHERE device='100.123.1.0'
name: /network-instances/network-instance/protocols/protocol/bgp/
key                                                                                                value
---                                                                                                -----
/network-instances/network-instance/protocols/protocol/bgp/peer-groups/peer-group/@peer-group-name underlay
>
```
BGP groups 
```
> SHOW TAG VALUES FROM  "/network-instances/network-instance/protocols/protocol/bgp/" with KEY ="/network-instances/network-instance/protocols/protocol/bgp/peer-groups/peer-group/@peer-group-name"
name: /network-instances/network-instance/protocols/protocol/bgp/
key                                                                                                value
---                                                                                                -----
/network-instances/network-instance/protocols/protocol/bgp/peer-groups/peer-group/@peer-group-name underlay
>
```
Sessions state on device 100.123.1.0
```
> SELECT "/network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/state/session-state" from "/network-instances/network-instance/protocols/protocol/bgp/" WHERE device='100.123.1.0' ORDER BY DESC LIMIT 4
name: /network-instances/network-instance/protocols/protocol/bgp/
time                /network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/state/session-state
----                -------------------------------------------------------------------------------------------------
1545167962424407403 ESTABLISHED
1545167962424407403 ESTABLISHED
1545167962424407403 ESTABLISHED
1545167962424407403 ESTABLISHED
>

```
Sessions state
```
> SELECT "/network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/state/session-state" FROM "/network-instances/network-instance/protocols/protocol/bgp/" ORDER BY DESC LIMIT 10
name: /network-instances/network-instance/protocols/protocol/bgp/
time                /network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/state/session-state
----                -------------------------------------------------------------------------------------------------
1545168049575284624 ESTABLISHED
1545168049575284624 ESTABLISHED
1545168049575284624 ESTABLISHED
1545168049575284624 ESTABLISHED
1545168049092114919 ESTABLISHED
1545168049092114919 ESTABLISHED
1545168049092114919 ESTABLISHED
1545168049092114919 ESTABLISHED
1545168048906507987 ESTABLISHED
1545168048906507987 ESTABLISHED
>
```
Operational state of interface ge-0/0/0 of device 100.123.1.0
```
> SELECT "/interfaces/interface/state/oper-status" FROM /interfaces/ WHERE ("device" = '100.123.1.0' AND "/interfaces/interface/@name" = 'ge-0/0/0' AND "/interfaces/interface/state/oper-status" = 'UP') ORDER BY DESC LIMIT 5
name: /interfaces/
time                /interfaces/interface/state/oper-status
----                ---------------------------------------
1545505895528360658 UP
1545505893529000291 UP
1545505891515823104 UP
1545505889443348550 UP
1545505887437980580 UP
```
Operational state of interface ge-0/0/0 of device 100.123.1.0 since 10 seconds
```
> SELECT "/interfaces/interface/state/oper-status" FROM /interfaces/ WHERE ("device" = '100.123.1.0' AND "/interfaces/interface/@name" = 'ge-0/0/0' AND "/interfaces/interface/state/oper-status" = 'UP' AND time >= now() - 10s)
name: /interfaces/
time                /interfaces/interface/state/oper-status
----                ---------------------------------------
1545506127609492045 UP
1545506129602754473 UP
1545506131607695707 UP
1545506133615007174 UP
1545506135612608747 UP
>
```
Total prefixes on device 100.123.1.4
```
> SELECT MEAN("/network-instances/network-instance/protocols/protocol/bgp/global/state/total-prefixes") FROM "/network-instances/network-instance/protocols/protocol/bgp/" WHERE "device" = '100.123.1.4'
name: /network-instances/network-instance/protocols/protocol/bgp/
time mean
---- ----
0    36
>
```

Total prefixes on device 100.123.1.0 since one minute
```
> SELECT mean("/network-instances/network-instance/protocols/protocol/bgp/global/state/total-prefixes") FROM "/network-instances/network-instance/protocols/protocol/bgp/" WHERE ("device" = '100.123.1.0') AND time >= now() - 1m
name: /network-instances/network-instance/protocols/protocol/bgp/
time                mean
----                ----
1545168168469112825 44
>
```
Sessions state and total prefixes
```
> SELECT "/network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/state/session-state",  "/network-instances/network-instance/protocols/protocol/bgp/global/state/total-prefixes" FROM "/network-instances/network-instance/protocols/protocol/bgp/" ORDER BY DESC LIMIT 4
name: /network-instances/network-instance/protocols/protocol/bgp/
time                /network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/state/session-state /network-instances/network-instance/protocols/protocol/bgp/global/state/total-prefixes
----                ------------------------------------------------------------------------------------------------- --------------------------------------------------------------------------------------
1545168279586592682 ESTABLISHED
1545168279586592682 ESTABLISHED
1545168279586592682                                                                                                   44
1545168279586592682 ESTABLISHED
>
```
Sessions state and total prefixes since 10 secondes for device 100.123.1.0
```
> SELECT "/network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/state/session-state", "/network-instances/network-instance/protocols/protocol/bgp/global/state/total-prefixes" FROM "/network-instances/network-instance/protocols/protocol/bgp/" WHERE "device" = '100.123.1.0' AND time >= now() - 10s
name: /network-instances/network-instance/protocols/protocol/bgp/
time                /network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/state/session-state /network-instances/network-instance/protocols/protocol/bgp/global/state/total-prefixes
----                ------------------------------------------------------------------------------------------------- --------------------------------------------------------------------------------------
1545168296433372717 ESTABLISHED
1545168296433372717                                                                                                   44
1545168296433372717 ESTABLISHED
1545168296433372717 ESTABLISHED
1545168296433372717 ESTABLISHED
1545168298425250176 ESTABLISHED
1545168298425250176 ESTABLISHED
1545168298425250176                                                                                                   44
1545168298425250176 ESTABLISHED
1545168298425250176 ESTABLISHED
1545168300432730351 ESTABLISHED
1545168300432730351 ESTABLISHED
1545168300432730351 ESTABLISHED
1545168300432730351 ESTABLISHED
1545168300432730351                                                                                                   44
1545168302438985906 ESTABLISHED
1545168302438985906                                                                                                   44
1545168302438985906 ESTABLISHED
1545168302438985906 ESTABLISHED
1545168302438985906 ESTABLISHED
>
```
Others influxdb queries examples
```
> SELECT * FROM "/interfaces/" ORDER BY DESC LIMIT 1
...
> SELECT * FROM "/network-instances/network-instance/protocols/protocol/bgp/" ORDER BY DESC LIMIT 1
...
> SELECT * FROM "/network-instances/network-instance/protocols/protocol/bgp/" WHERE device='100.123.1.0' AND time >= now() - 10s
...
> SELECT * FROM "/network-instances/network-instance/protocols/protocol/bgp/" WHERE "/network-instances/network-instance/protocols/protocol/bgp/neighbors/neighbor/@neighbor-address" ='192.168.1.7' ORDER BY DESC LIMIT 1
...
> SELECT COUNT("/interfaces/interface/state/oper-status") FROM /interfaces/ WHERE "device" = '100.123.1.0' AND "/interfaces/interface/@name" = 'ge-0/0/0' AND "/interfaces/interface/state/oper-status" = 'UP' AND time >= now() - 1m GROUP BY time(10s)
name: /interfaces/
time                count
----                -----
1545168180000000000 1
1545168190000000000 5
1545168200000000000 5
1545168210000000000 5
1545168220000000000 5
1545168230000000000 5
1545168240000000000 4
>
```
exit 
```
> exit
#
```
exit the influxdb container
```
# exit 
```
# query the influxdb database with python to verify
install the influxdb python library.
This python library is a client for interacting with InfluxDB.
```
$ pip install influxdb
```
Verify
```
$ pip list | grep influx
```
you can now interact with InfluxDB using Python

run this command to start a python interactive session

```
$ python
```
connect to InfluxDB using Python
```
>>> from influxdb import InfluxDBClient
>>> influx_client = InfluxDBClient('localhost',8086)
```
list the databases
```
>>> influx_client.query('show databases')
ResultSet({'(u'databases', None)': [{u'name': u'_internal'}, {u'name': u'juniper'}]})
```
list measurements for a database
```
>>> influx_client.query('show measurements', database='juniper')
ResultSet({'(u'measurements', None)': [{u'name': u'/interfaces/'}, {u'name': u'/network-instances/network-instance/protocols/protocol/bgp/'}]})
```
query data from a particular measurement and database

```
>>> gp = influx_client.query("""select "/interfaces/interface/subinterfaces/subinterface/state/counters/in-octets" from "/interfaces/" where "/interfaces/interface/@name" = 'ge-0/0/0' and "device" = '100.123.1.0' order by desc limit 10""", database='juniper').get_points()
>>> for item in gp:
...   print item['/interfaces/interface/subinterfaces/subinterface/state/counters/in-octets']                                                  ...
515548
515548
515496
515496
515496
515496
515496
515496
515496
515496
```
```
>>> gp = influx_client.query("""select "/interfaces/interface/subinterfaces/subinterface/state/counters/in-octets" from "/interfaces/" where "/interfaces/interface/@name" = 'ge-0/0/0' and "device" = '100.123.1.0' order by desc limit 10""", database='juniper').get_points()
>>> for item in gp:
...   print item
...
{u'/interfaces/interface/subinterfaces/subinterface/state/counters/in-octets': 515671, u'time': u'2018-12-18T22:24:11.94933689Z'}
{u'/interfaces/interface/subinterfaces/subinterface/state/counters/in-octets': 515671, u'time': u'2018-12-18T22:24:09.748368485Z'}
{u'/interfaces/interface/subinterfaces/subinterface/state/counters/in-octets': 515619, u'time': u'2018-12-18T22:24:07.742719284Z'}
{u'/interfaces/interface/subinterfaces/subinterface/state/counters/in-octets': 515619, u'time': u'2018-12-18T22:24:05.742853495Z'}
{u'/interfaces/interface/subinterfaces/subinterface/state/counters/in-octets': 515619, u'time': u'2018-12-18T22:24:03.542083513Z'}
{u'/interfaces/interface/subinterfaces/subinterface/state/counters/in-octets': 515619, u'time': u'2018-12-18T22:24:01.537968182Z'}
{u'/interfaces/interface/subinterfaces/subinterface/state/counters/in-octets': 515619, u'time': u'2018-12-18T22:23:59.338121834Z'}
{u'/interfaces/interface/subinterfaces/subinterface/state/counters/in-octets': 515619, u'time': u'2018-12-18T22:23:57.338440891Z'}
{u'/interfaces/interface/subinterfaces/subinterface/state/counters/in-octets': 515619, u'time': u'2018-12-18T22:23:55.328623407Z'}
{u'/interfaces/interface/subinterfaces/subinterface/state/counters/in-octets': 515619, u'time': u'2018-12-18T22:23:53.128843287Z'}
>>>
```
