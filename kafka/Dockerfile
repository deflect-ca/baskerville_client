FROM wurstmeister/kafka:0.10.2.0-1

#ENV PASSWORD="kafkadocker"
#
#COPY /local_cert /certs
#
## -passin pass:$PASSWORD
#RUN ( keytool -keystore /certs/kafka.server.keystore.jks -alias CARoot -storepass $PASSWORD -import -file /certs/ca-cert ) ||  echo 'kafka.server.keystore.jks already exists 1'
#RUN ( keytool -keystore /certs/kafka.server.keystore.jks -alias localhost -import -file /certs/cert-signed ) || echo 'kafka.server.keystore.jks already exists 2'
##RUN keytool -keystore /certs/client.truststore.jks -alias CARoot -import -file /certs/ca-cert
#
#RUN ./

RUN echo "auto.create.topics.enable=true" > /opt/kafka/config/server.properties
#RUN /opt/kafka/bin/kafka-topics.sh --create --zookeeper $ZOOKEEPER --replication-factor 1 --partitions 10 --topic $BASKERVILLE_TOPIC --config retention.ms 120000

EXPOSE 9092