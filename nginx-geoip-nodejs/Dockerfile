# v0.1.5
ARG NGINX_VERSION=1.22.1

# Stage 1: Build GeoIP2 module
FROM nginx:$NGINX_VERSION-alpine AS builder

ARG NGINX_VERSION=1.22.1
ARG GEOIP2_VERSION=3.4

# Install build dependencies
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

# Download and build GeoIP2 module
RUN cd /opt \
    && git clone --depth 1 -b $GEOIP2_VERSION --single-branch https://github.com/leev/ngx_http_geoip2_module.git \
    && wget -O - http://nginx.org/download/nginx-$NGINX_VERSION.tar.gz | tar zxfv - \
    && mv /opt/nginx-$NGINX_VERSION /opt/nginx \
    && cd /opt/nginx \
    && ./configure --with-compat --add-dynamic-module=/opt/ngx_http_geoip2_module \
    && make modules

# Stage 2: Final image# Stage 2: Final image
FROM node:22-alpine

# Install Nginx runtime dependencies
RUN apk --update --no-cache add \
        bash \
        libmaxminddb \
        pcre2 \
        zlib \
        openssl

# Copy Nginx and GeoIP2 module from the builder stage
COPY --from=builder /usr/sbin/nginx /usr/sbin/nginx
COPY --from=builder /etc/nginx /etc/nginx
COPY --from=builder /usr/lib/nginx /usr/lib/nginx
COPY --from=builder /var/log/nginx /var/log/nginx
COPY --from=builder /var/cache/nginx /var/cache/nginx
COPY --from=builder /opt/nginx/objs/ngx_http_geoip2_module.so /usr/lib/nginx/modules

# Load GeoIP2 module and set permissions
RUN chmod -R 644 /usr/lib/nginx/modules/ngx_http_geoip2_module.so \
    && sed -i '1iload_module \/usr\/lib\/nginx\/modules\/ngx_http_geoip2_module.so;' /etc/nginx/nginx.conf

# Copy GeoIP2 configuration and database
COPY nginx-geoip2.conf /etc/nginx/conf.d/geoip2.conf
COPY dbip-country-lite.mmdb /etc/nginx/conf.d/dbip-country-lite.mmdb

# Verify Node.js installation
RUN node -v && npm -v

# Expose ports
EXPOSE 80

# Start Nginx
CMD ["nginx", "-g", "daemon off;"]