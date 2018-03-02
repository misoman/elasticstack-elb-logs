# AWS ELB/ALB logs to ElasticStack
This is example of Logstash configuration + Kibana dashboard which graphicaly display traffic from AWS ELB/ALB logs.
It includes installation of ElasticStack on Ubuntu 16.04 or as Docker container.
Thanks to [vigeek](https://github.com/vigeek/aws-elb-logs-to-logstash) example and [elastic](https://github.com/elastic/stack-docker) for their work.

## Installation on Ubuntu 16.04
- install packages
```
sudo apt install apt-transport-https openjdk-8-jre
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-6.x.list
sudo apt update
sudo apt install elasticsearch kibana logstash
```
- install x-pack plugins
```
sudo /usr/share/elasticsearch/bin/elasticsearch-plugin install x-pack
sudo /usr/share/kibana/bin/kibana-plugin install x-pack
sudo /usr/share/logstash/bin/logstash-plugin install x-pack
```
- install logstash S3 input plugin
```
sudo /usr/share/logstash/bin/logstash-plugin install logstash-input-s3
```
- setup passwords for elastic/kibana/logstash_system
```
sudo /usr/share/elasticsearch/bin/x-pack/setup-passwords interactive
```
- open `/etc/kibana/kibana.yml` and edit
```
elasticsearch.password: kibanapassword
```
- open `/etc/logstash/logstash.yml` and add
```
xpack.monitoring.enabled: true
xpack.monitoring.elasticsearch.username: logstash_system
xpack.monitoring.elasticsearch.password: logstash_systempassword
xpack.monitoring.elasticsearch.url: http://localhost:9200
```
- edit `config/logstash/logstash.conf` with your AWS S3 parameters.
  - access_key_id
  - secret_access_key
  - bucket,region
  - prefix: check it in S3 bucket for exact path name
  - hosts: change it to localhost or ip/name of elasticsearch in output part
- copy the config file to `/etc/logstash/conf.d/` before Logstash start.


## Installation as Docker container
###### Prerequisites
- Docker and Compose.

before `docker-compose up` you have to install logstash-input-s3 plugin to logstash image.
- pull the image
```
sudo docker pull docker.elastic.co/logstash/logstash:6.2.1

```
- go to `build` folder (don't forget the dot at the end of command)
```
sudo docker build -t logstash-s3:6.2.1 .

```
- Edit `.env` with your own password.
- Edit `config/logstash/logstash.conf` with your AWS S3 parameters.
  - access_key_id
  - secret_access_key
  - bucket,region
  - prefix: check it in S3 bucket for exact path name
- Change user permission for data folder
```
sudo chown -R 1000:1000 data

```
- `docker-compose up -d`

## Setup Kibana dashboard
Access to Kibana on port 5601 with `elastic / password`. Make index `logstash-*` but choose timestamp `timestamp` without @.
Go to Management -> Kibana -> Saved Objects and import all 3 .json files from `kibana-dashboard` folder. Open `Load Balancing Dashboard` in Dashboard.

## Delete old Indices
Delete on machine/container where is elasticsearch service. 
```
curl -XDELETE 'elastic:password@localhost:9200/logstash-2018.01.01'

```
Or use `elasticsearch-curator`.