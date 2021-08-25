ARG ELK_VERSION=7.13.1
ARG DEFLECT_DNET=dnet1

FROM docker.elastic.co/beats/filebeat:7.13.1

ENV ELK_VERSION $ELK_VERSION
ENV DEFLECT_DNET $DEFLECT_DNET

USER root

# COPY filebeat.yml /usr/share/filebeat/filebeat.yml
# COPY output.json /usr/share/filebeat/output.json
# USER root
# RUN chown root:filebeat /usr/share/filebeat/filebeat.yml
# RUN mkdir /usr/share/filebeat/dockerlogs
# USER filebeat