# GLOBAL ARGS:
ARG DOCKER_KAFKA_HOST
ARG DASHBOARD_BRANCH
# for front-end
ARG API_BASE_URL
ARG SOCKET_URL
# for backend:
ARG BASKERVILLE_BRANCH
ARG REDIS_HOST

#FROM nginx:1.17.1-alpine AS NGINX
FROM node:14.8.0-alpine AS FRONTEND
ARG DASHBOARD_BRANCH
ARG API_BASE_URL
ARG SOCKET_URL

ENV DASHBOARD_BRANCH $DASHBOARD_BRANCH
ENV API_BASE_URL $API_BASE_URL
ENV SOCKET_URL $SOCKET_URL

RUN apk update \
#    && apk upgrade \
    && apk add git \
    && mkdir /app && cd /app \
    && git clone --branch $DASHBOARD_BRANCH https://github.com/deflect-ca/baskerville_dashboard.git  \
    && cd baskerville_dashboard/front-end \
    && npm install \
    && npm install -g @angular/cli@11.1.0

RUN echo $SOCKET_URL, $API_BASE_URL && echo "<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<"

# this builds the front-end with provided configuration and copies the result in /var/www for nginx
RUN cd /app/baskerville_dashboard/front-end  \
    && npm run config  \
    && ng build --prod

FROM openjdk:8 AS OJDK8
FROM python:3.6 AS BACKEND

ARG DOCKER_KAFKA_HOST
ARG BASKERVILLE_BRANCH
ARG DASHBOARD_BRANCH
ARG REDIS_HOST

ENV DOCKER_KAFKA_HOST $DOCKER_KAFKA_HOST
ENV DASHBOARD_BRANCH $DASHBOARD_BRANCH
ENV BASKERVILLE_BRANCH $BASKERVILLE_BRANCH
ENV REDIS_HOST $REDIS_HOST
ENV BASKERVILLE_ROOT '/app/baskerville'
ENV BASKERVILLE_DASH_ROOT '/app/baskerville_dashboard'


# Get jdk8 from previous stage https://docs.docker.com/develop/develop-images/multistage-build/
COPY --from=OJDK8 /usr/local/openjdk-8 /usr/local/openjdk-8
COPY --from=FRONTEND /app/baskerville_dashboard/front-end/dist/baskerville_dashboard_frontend/ /var/www/baskerville_dashboard_frontend/

# Set java path
ENV JAVA_HOME /usr/local/openjdk-8
ENV PATH $PATH:$JAVA_HOME/bin

# TODO: The following installs spark-iforest, esretriever and baskerville
# which takes a long time - mostly because of the different pyspark versions.
RUN apt-get clean && apt-get update \
#    && apt-get -y upgrade \
    && apt-get install -y nginx=1.18.*  \
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
    && cd /app/baskerville_dashboard/backend/src/baskerville_dashboard

# copy all related configurations
COPY ./nginx.conf /etc/nginx/nginx.conf
COPY ./config.yaml /app/baskerville_dashboard/backend/conf
COPY ./baskerville.yaml /app/baskerville_dashboard/backend/conf

WORKDIR /app/baskerville_dashboard/backend/src/baskerville_dashboard

# socketio for Python includes a production grade web server.
# start.sh starts nginx and runs dashboard backend.
COPY start.sh start.sh
CMD ["sh", "./start.sh"]
EXPOSE 80 81 5000