# Copyright (c) 2020, eQualit.ie inc.
# All rights reserved.
#
# This source code is licensed under the BSD-style license found in the
# LICENSE file in the root directory of this source tree.


FROM grafana/grafana:7.3.0

ARG BASKERVILLE_DB

ENV DS_BASKERVILLE_DB $BASKERVILLE_DB

ENV GF_SECURITY_ADMIN_PASSWORD changeme
# optional:
ENV GF_INSTALL_PLUGINS grafana-clock-panel,grafana-simple-json-datasource

VOLUME /var/lib/grafana

COPY ./dashboards_config /etc/grafana/provisioning/dashboards
COPY ./datasources /etc/grafana/provisioning/datasources
COPY ./dashboards /etc/grafana/provisioning/dashboards/baskerville