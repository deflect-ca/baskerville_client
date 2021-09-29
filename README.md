## What is Baskerville?

Manual identification and mitigation of (DDoS) attacks on websites is a difficult and time-consuming task with many challenges. This is why Baskerville was created, to identify the attacks directed to Deflect protected 
websites as they happen and give the infrastructure the time to respond properly. Baskerville is an analytics engine that leverages Machine Learning to distinguish between normal and abnormal web traffic behavior. 
In short, Baskerville is a Layer 7(application layer) DDoS attack mitigation tool.

## What is Baskerville client

Baskeville client is a client module which 
* processes web server logs and calculates statistical features 
* send features to a clearing house instance of Baskerville
* receives predictions from clearing house for every IP
* issues challenge commands for every malicious IP in a separate Kafka topic
* monitors attacks in Grafana dashboards

## Installation

* create `.env` file:
```commandline
cp dot_env.sh .env
```
* set `CLEARING_HOUSE_KAFKA` variable to your Baskerville clearing house url
* set `KAFKA_HOST` variable to your ip

* add TLS keys provided by clearing house into 'clearing_house_connection' subfolder. 
```commandline
./clearing_house_connection/caroot.pem
./clearing_house_connection/certificate.pem
./clearing_house_connection/key.pem
```

* By default, filebeat is using `./logs` folder. 
You probably need to mount the proper weblog directory in `docker-compose.yaml':
```yaml
  filebeat:
    volumes:
      - type: bind
        source: ./logs/
        target: /var/log/
        read_only: true
```
* Create your client id and set it in `./conf/preprocessing.yaml` and `./conf/postprocessing.yaml`
```yaml
engine:
 id_client: '...'
```

* Provide clearing house with your `client_id` and your kafka external url:
```yaml
your_ip:29092
```

* Change your postgres password in `.env`
```commandline
BASKERVILLE_POSTGRES_PASSWORD=changeme
```

* Change your postgres password in `containers/grafana/datasources/postgresql.yaml`
```yaml
datasources:
  secureJsonData:
    password: ...
```

* Change your Grafana `admin` password in `containers/grafana/Dockerfile`
```commandline
ENV GF_SECURITY_ADMIN_PASSWORD ...
```

* Launch Baskerville:
```
docker-compose up -d
```

* Open Grafana dashboard at `localhost:3000` and login with your Grafana admin password. 

* Open dashboard `baskerville/Attack` and `baskerville/TrafficLight

To verify kafka is receiving logs:
- First open your browser at `http://localhost/test` (or any other similar url) and refresh a few times
- Verify that your requests were written to the right file:
  ```bash
  tail -f /var/log/banjax/nginx-logstash-format.log
  ```
- Check your filebeat and logstash logs for any errors if necessary:
```bash
docker-compose logs -f logstash
docker-compose logs -f filebeat 
```
- Then login to the kafka service and consume a few messages from the logs topic, e.g. `deflect.logs`:
```bash
docker-compose exec kafka bash
# this will provide the message count, e.g. partition 0 has 6131 messages
# so not zero message count means we are receiving correctly
/opt/bitnami/kafka/bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list localhost:9092 --topic deflect.logs
>>> deflect.logs:0:6131

# the following will consume / display a few messages, just to make sure all is well
/opt/bitnami/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic deflect.logs --offset 6131 --partition 0
```

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
