#1   On monitoring server setup following
- Hostname
- Timezone
- Disable SELINUX
- install docker
- Clone this repository

#2   Attach a new disk to store monitoring data, create 
- LVM
- use ext4 filesystem as it supports both increase and decrease
- mount at /monitoring-service
- update /etc/fstab


#3   Create necessary directories
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

#5   Recheck the following configuration under Dockerfile before building docker image
- ./graphite/carbon.conf   (MAX_UPDATES_PER_SECOND)
- ./graphite/storage-schema.conf  (Retention)
- ./grafana/grafana.db ( if you have existing grafana database and want to apply that)

#4   Create docker image using Dockerfile
```docker build  -t 'karansingh/rht_sat_monitoring_service:v3' .```

#5   Start docker container using Docker image created in last step. Make sure 
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

#6   Verify clients can connect to monitoring server on the following exposed ports
80, 81, 2003

#7   Setup grafana web, take examples from grafana dashboards in the repo

#8   If you are migrating your app , then
- Take grafana.db backup in advance
- Taek grafana dashboards just in case
- Make sure metric data is availab for you to use again.
