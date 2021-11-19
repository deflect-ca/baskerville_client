FROM node:14.8.0-alpine AS build
ARG DASHBOARD_BRANCH
ARG API_BASE_URL
ARG SOCKET_URL

RUN apk update \
#    && apk upgrade \
    && apk add git \
    && mkdir /app && cd /app \
    && git clone --branch $DASHBOARD_BRANCH https://github.com/deflect-ca/baskerville_dashboard.git  \
    && cd baskerville_dashboard/front-end \
    && npm install \
    && npm install -g @angular/cli@11.1.0

# provide values for the following
ENV API_BASE_URL $API_BASE_URL
ENV SOCKET_URL $SOCKET_URL

# this builds the front-end with provided configuration and copies the result in /var/www for nginx
RUN cd /app/baskerville_dashboard/front-end  \
    && npm run config  \
    && ng build --prod


FROM nginx:1.17.1-alpine
COPY nginx.conf /etc/nginx/nginx.conf
COPY frontend.conf /etc/nginx/sites.d/frontend.conf
COPY --from=build /app/baskerville_dashboard/front-end/dist/baskerville_dashboard_frontend/ /var/www/baskerville_dashboard_frontend/

#RUN /usr/sbin/nginx -s reload
#RUN #/etc/init.d/nginx reload

CMD ["nginx", "-g", "daemon off;"]
EXPOSE 80 5000