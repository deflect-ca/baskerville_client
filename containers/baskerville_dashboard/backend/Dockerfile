FROM python:3.6
ARG BASKERVILLE_BRANCH
ARG DASHBOARD_BRANCH

# TODO: The following installs spark-iforest, esretriever and baskerville
# which takes a long time - mostly because of the different pyspark versions.

RUN apt-get update \
    && apt-get -y upgrade \
    && apt-get install git  \
    && pip install --upgrade pip \
    && mkdir /app && cd /app  \
    && git clone https://github.com/titicaca/spark-iforest.git \
    && cd spark-iforest/python \
    && pip install . \
    && cd /app \
    && git clone https://github.com/equalitie/esretriever.git  \
    && cd esretriever \
    && pip install . \
    && cd /app \
    && git clone --branch $BASKERVILLE_BRANCH https://github.com/deflect-ca/baskerville.git \
    && cd baskerville  \
    && pip install . \
    && cd /app \
    && git clone --branch $DASHBOARD_BRANCH https://github.com/deflect-ca/baskerville_dashboard.git \
    && cd baskerville_dashboard/backend  \
    && pip install . \

ENV BASKERVILLE_ROOT '/app/baskerville'
ENV BASKERVILLE_DASH_ROOT '/app/baskerville_dashboard'

COPY ./config.yaml /app/baskerville_dashboard/backend/conf
COPY ./baskerville.yaml /app/baskerville_dashboard/backend/conf

# socketio for Python includes a production grade web server:
RUN cd /app/baskerville_dashboard/backend/src/baskerville_dashboard && python app.py

EXPOSE 5000