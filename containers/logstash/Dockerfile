ARG ELK_VERSION

# https://www.docker.elastic.co/
FROM docker.elastic.co/logstash/logstash:${ELK_VERSION}

# Add your logstash plugins setup here
RUN logstash-plugin install logstash-input-beats
RUN logstash-plugin install logstash-filter-grok
RUN logstash-plugin install logstash-integration-kafka
RUN logstash-plugin install logstash-filter-geoip
RUN logstash-plugin install logstash-filter-useragent
