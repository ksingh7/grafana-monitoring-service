# Step-1   
On monitoring server setup following
- Hostname
- Timezone
- Disable SELINUX
- install docker
- Clone this repository

# Step-2   
Attach a new disk to store monitoring data, create 
- LVM
- use ext4 filesystem as it supports both increase and decrease
- mount at /monitoring-service
- update /etc/fstab


# Step-3
Create necessary directories
```
cd /monitoring-service;
mkdir -p data/whisper;
mkdir -p data/elasticsearch;
mkdir -p data/grafana;
mkdir -p log/graphite;
mkdir -p log/graphite/webapp;
mkdir -p log/elasticsearch;
chmod -R 777 data;
chmod -R 777 log;
```

# Step-4
Recheck the following configuration under Dockerfile before building docker image
- ./graphite/carbon.conf   (MAX_UPDATES_PER_SECOND)
- ./graphite/storage-schema.conf  (Retention)
- ./grafana/grafana.db ( if you have existing grafana database and want to apply that)

# Step-5
Create docker image using Dockerfile
```docker build  -t 'karansingh/rht_sat_monitoring_service:v3' .```

# Step-6
Start docker container using Docker image created in last step. Make sure 
- To change directory to /monitoring-service
- docke restart policy is set

```
cd /monitoring-service;
docker run \
   --detach \
   --restart always \
   --publish=80:80 \
   --publish=81:81 \
   --publish=8125:8125/udp \
   --publish=2003:2003 \
   --publish=8126:8126 \
   --name monitoring-service-v3 \
   --volume=$(pwd)/data/whisper:/opt/graphite/storage/whisper \
   --volume=$(pwd)/data/elasticsearch:/var/lib/elasticsearch \
   --volume=$(pwd)/data/grafana:/opt/grafana/data \
   --volume=$(pwd)/log/graphite:/opt/graphite/storage/log \
   --volume=$(pwd)/log/elasticsearch:/var/log/elasticsearch \
   karansingh/rht_sat_monitoring_service:v3
```

# Step-7
Verify clients can connect to monitoring server on the following exposed ports
80, 81, 2003

# Step-8
Setup grafana web, take examples from grafana dashboards in the repo

# Step-9
If you are migrating your app , then
- Take grafana.db backup in advance
- Taek grafana dashboards just in case
- Make sure metric data is availab for you to use again.

# Troubleshooting
If you see problems like , graphite not storing data past 7 days (https://github.com/kamon-io/docker-grafana-graphite/issues/79). This can be related to retention configured in storage-schema.conf. To fix this i have updated retention with correct format , refer that.
To verify if retention set is correct , run  /opt/graphite/bin/validate-storage-schemas.py

Restart carbon-cache service
```
/usr/bin/python /opt/graphite/bin/carbon-cache.py --pidfile /var/run/carbon-cache-a.pid --debug stop
/usr/bin/python /opt/graphite/bin/carbon-cache.py --pidfile /var/run/carbon-cache-a.pid --debug status
netstat -plunt ; ps -ef | grep -i carbon
/usr/bin/python /opt/graphite/bin/carbon-cache.py --pidfile /var/run/carbon-cache-a.pid start
/usr/bin/python /opt/graphite/bin/carbon-cache.py --pidfile /var/run/carbon-cache-a.pid --debug status
netstat -plunt ; ps -ef | grep -i carbon 
```
Verify whisper .wsp files are have format as mentioned in retention
```
pip install whisper
whisper-info.py <whisper.wsp>
```
