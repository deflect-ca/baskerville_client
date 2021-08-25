ARG ELK_VERSION
ARG DEFLECT_DNET

FROM docker.elastic.co/beats/filebeat:${ELK_VERSION}

USER root

RUN echo $DEFLECT_DNET
# COPY filebeat.yml /usr/share/filebeat/filebeat.yml
# COPY output.json /usr/share/filebeat/output.json
# USER root
# RUN chown root:filebeat /usr/share/filebeat/filebeat.yml
# RUN mkdir /usr/share/filebeat/dockerlogs
# USER filebeat