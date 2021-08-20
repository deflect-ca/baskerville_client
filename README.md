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
