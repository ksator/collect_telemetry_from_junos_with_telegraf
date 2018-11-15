# install docker

This is not covered in this repository

# Junos
```
jcluser@vMX-1> show version | match "na telemetry"
JUNOS na telemetry [18.2R1.9-C1]
```
```
jcluser@vMX-1> show version | match openconfig
JUNOS Openconfig [0.0.0.10-1]
```
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

# influxdb

pull docker images 
```
docker pull influxdb
```
Verify
```
docker images influxdb
```
Instanciate an influxdb container
```
docker run -d --name influxdb -p 8083:8083 -p 8086:8086 influxdb
```
Verify
```
docker ps | grep influxdb
```
for troubleshooting purposes you can run this command
```
docker logs influxdb
```
start a shell session in the influxdb container
```
docker exec -it influxdb bash
```
influxdb configuration file
```
more /etc/influxdb/influxdb.conf
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
exit
```

# telegraf

get ip address used by containers
```
ifconfig docker0
```

pull docker images 
```
docker pull telegraf
```
Verify
```
docker images telegraf
```

create a telegraf configuration file
```
vi telegraf.conf
```
```
more telegraf
[[inputs.jti_openconfig_telemetry]]
  servers = ["172.30.52.156:50051"]
  username = "lab"
  password = "m0naco"
  client_id = "telegraf"
  sample_frequency = "2000ms"
  sensors = ["/interfaces/"]
  retry_delay = "1000ms"
  str_as_tags = false

[[outputs.influxdb]]
      urls = ["http://172.17.0.1:8086"]
      database = "juniper"
      precision = "s"
      write_consistency = "any"
      timeout = "5s"
      username = "juniper"
      password = "juniper"
```
instanciate a telegraf container
```
docker run --name telegraf -d -v $PWD/telegraf.conf:/etc/telegraf/telegraf.conf:ro telegraf
```
verify
```
docker ps | grep telegraf
```
for troubleshooting purposes you can run this command
```
docker logs telegraf
```
start a shell session in the influxdb container
```
docker exec -it telegraf bash
```
verify the telegraf configuration file
```
more /etc/telegraf/telegraf.conf
```
exit the telegraf container
```
exit
```
# Verify

query the influxdb database
```
```


