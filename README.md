# Introduction
## What is Baskerville?

Manual identification and mitigation of (DDoS) attacks on websites is a difficult and time-consuming task with many challenges. This is why Baskerville was created, to identify the attacks directed to Deflect protected 
websites as they happen and give the infrastructure the time to respond properly. Baskerville is an analytics engine that leverages Machine Learning to distinguish between normal and abnormal web traffic behavior. 

In short, Baskerville is a Layer 7(application layer) DDoS attack mitigation tool.

## What is Baskerville client?

Baskerville client is a client module which:
* Processes nginx web server logs and calculates statistical features.
* Sends features to a clearing house instance of Baskerville.
* Receives predictions from a clearing house for every IP.
* Issues challenge commands for every malicious IP in a separate Kafka topic.
* Monitors attacks in Grafana dashboards.

# System requirements
* Linux server with Docker already installed.
* A minimum of 8GB of RAM is suggested.
* Allow TCP network traffic on port `29092` for the Kafka connection from the clearing house to the client.
* Allow TCP network traffic on port `3000` for access to the Grafana dashboard.

# Installation
Download the Baskerville client software into the current working directory and change directories:
```commandline
git clone https://github.com/deflect-ca/baskerville_client.git && cd baskerville_client
```

## Configuration

* Ensure the following directories exist:
```commandline
mkdir -p /var/log/nginx && mkdir -p /var/log/banjax-next
```

* Create the `.env` file:
```commandline
cp dot_env.sh .env
```
* Make the following modifications to the `.env` file:
    * Set `CLEARING_HOUSE_KAFKA` variable to your Baskerville clearing house URL.
    * Set `KAFKA_HOST` variable to your server's IP address.
    * Set provided passwords `KAFKA_KEYSTORE_PASSWORD` and `KAFKA_TRUSTSTORE_PASSWORD`.

* Add TLS keys provided by clearing house into the `./clearing_house_connection` directory. 
```commandline
./clearing_house_connection/caroot.pem
./clearing_house_connection/certificate.pem
./clearing_house_connection/key.pem
```

* Add local Kafka keys provided by clearing house into the `./kafka_local` directory:
```commandline
./conf/kafka_local/kafka.keystore.jks
./conf/kafka_local/kafka.truststore.jks
```

* Create your client id and set it in `./conf/preprocessing.yaml` and `./conf/postprocessing.yaml`:
```yaml
engine:
  id_client: '...'
```

* Provide clearing house with your `client_id` and your kafka external URL:
```yaml
your_ip:29092
```

* Change your postgres password in `.env`:
```commandline
BASKERVILLE_POSTGRES_PASSWORD=changeme
```

* Change your postgres password in `containers/grafana/datasources/postgresql.yaml`:
```yaml
datasources:
  secureJsonData:
    password: ...
```

* Change your Grafana `admin` password in `containers/grafana/Dockerfile`:
```commandline
ENV GF_SECURITY_ADMIN_PASSWORD ...
```
# Post installation

## Run Baskerville client
* To launch the software, run the following command:
```
docker-compose up -d
```

* Open the Grafana dashboard at `localhost:3000` and log in with your Grafana admin password. 

* Open dashboards `baskerville/Attack` and `baskerville/TrafficLight`.

## Troubleshooting
### Containers fail to start
In case the `baskerville_preprocessing` and `baskerville_postprocessing` containers fail to start because the `baskerville` database does not exist:
```bash
docker-compose exec postgres bash
psql
CREATE DATABASE baskerville;
\q
exit

docker-compose restart baskerville_preprocessing baskerville_postprocessing
```

### Ensure kafka is healthy 
To verify kafka is receiving logs:
- First open your browser at `http://localhost/test` (or any other similar URL) and refresh a few times
- Verify that your requests were written to the right file:
  ```bash
  tail -f /var/log/banjax-next/nginx-logstash-format.log
  ```
- Check your filebeat and logstash logs for any errors if necessary:
```bash
docker-compose logs -f logstash
docker-compose logs -f filebeat 
```
- Then log in to the kafka service and consume a few messages from the logs topic, e.g. `deflect.logs`:
```bash
docker-compose exec kafka bash
# this will provide the message count, e.g. partition 0 has 6131 messages
# so not zero message count means we are receiving correctly
/opt/bitnami/kafka/bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list localhost:9092 --topic deflect.logs
>>> deflect.logs:0:6131

# the following will consume / display a few messages, just to make sure all is well
/opt/bitnami/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic deflect.logs --offset 6131 --partition 0
```

### Dashboard
Notes for the Baskerville dashboard:
- The Dockerfile is heavy, as it is a multistage Dockerfile. It uses Nginx internally but, since we already have an nginx service
it would be nice to have a common volume for the front-end to be served and proper networking for the backend to be served also (only for the web-sockets)
- It has Baskerville as a dependency (with all that this entails, like esretriever, iforest pyspark etc, which means different pyspark versions with conflicts and a lot of build time)



### Misc
In case baskerville_preprocessing and baskerville_postprocessing fail to start because `baskerville` database does not exist:
```bash
docker-compose exec postgres bash
psql
CREATE DATABASE baskerville;
\q
exit

docker-compose restart baskerville_preprocessing baskerville_postprocessing
```

### Firewall
- open 29092 port for Kafka connections

