# v0.1.2
ARG NGINX_VERSION=1.22.1

FROM nginx:$NGINX_VERSION-alpine AS builder

ARG NGINX_VERSION=1.22.1
ARG GEOIP2_VERSION=3.4

RUN apk --update --no-cache add \
        gcc \
        make \
        libc-dev \
        g++ \
        openssl-dev \
        linux-headers \
        pcre-dev \
        zlib-dev \
        libtool \
        automake \
        autoconf \
        libmaxminddb-dev \
        git

RUN cd /opt \
    && git clone --depth 1 -b $GEOIP2_VERSION --single-branch https://github.com/leev/ngx_http_geoip2_module.git \
    && wget -O - http://nginx.org/download/nginx-$NGINX_VERSION.tar.gz | tar zxfv - \
    && mv /opt/nginx-$NGINX_VERSION /opt/nginx \
    && cd /opt/nginx \
    && ./configure --with-compat --add-dynamic-module=/opt/ngx_http_geoip2_module \
    && make modules 

FROM nginx:$NGINX_VERSION-alpine

# Install bash and other dependencies
RUN apk --update --no-cache add \
        bash \
        libmaxminddb \
        nodejs \
        npm

# Copy the GeoIP2 module from the builder stage
COPY --from=builder /opt/nginx/objs/ngx_http_geoip2_module.so /usr/lib/nginx/modules

# Install Node.js using apk (alpine compatible)
RUN apk --update --no-cache add nodejs npm \
    && chmod -R 644 /usr/lib/nginx/modules/ngx_http_geoip2_module.so \
    && sed -i '1iload_module \/usr\/lib\/nginx\/modules\/ngx_http_geoip2_module.so;' /etc/nginx/nginx.conf

# Copy GeoIP2 configuration and database
COPY  deploy/builder/nginx-geoip2.conf /etc/nginx/conf.d/geoip2.conf
COPY deploy/builder/dbip-country-lite.mmdb /etc/nginx/conf.d/dbip-country-lite.mmdb

# Verify Node.js installation
RUN node -v && npm -v