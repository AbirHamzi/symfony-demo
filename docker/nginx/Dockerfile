#------------------------------------------------------------------------------
# Build assets
#------------------------------------------------------------------------------
FROM node:14.4-alpine3.12 AS assets

WORKDIR /srv

COPY ./package.json ./package-lock.json ./webpack.config.js ./
RUN set -eux \
    && npm install

COPY assets/ ./assets

RUN set -eux \
    && mkdir -p /srv/public/build \
    && npm run build
    
#------------------------------------------------------------------------------

#------------------------------------------------------------------------------
# HTTP server
#
# The OpenResty distribution of NGINX is only needed for Kubernetes compatiblity (dynamic upstream resolution)
#------------------------------------------------------------------------------
FROM openresty/openresty:1.19.3.1-alpine AS nginx

COPY ./docker/nginx/rootfs/etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf
COPY ./docker/nginx/rootfs/usr/local/openresty/nginx/conf/nginx.conf /usr/local/openresty/nginx/conf/nginx.conf

COPY --from=assets /srv/public ./
WORKDIR /srv/public

RUN mkdir -p /var/log/nginx && touch /var/log/nginx/error.log /var/log/nginx/access.log

COPY ./public ./