# v0.1.6 - Added headers-more-nginx-module for setting cookies in redirects
ARG NGINX_VERSION=1.22.1

# Stage 1: Build GeoIP2 and headers-more modules
FROM nginx:$NGINX_VERSION-alpine AS builder

ARG NGINX_VERSION=1.22.1
ARG GEOIP2_VERSION=3.4
ARG HEADERS_MORE_VERSION=0.37

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

# Download and build modules
RUN cd /opt \
    && git clone --depth 1 -b $GEOIP2_VERSION --single-branch https://github.com/leev/ngx_http_geoip2_module.git \
    && git clone --depth 1 -b v$HEADERS_MORE_VERSION --single-branch https://github.com/openresty/headers-more-nginx-module.git \
    && wget -O - http://nginx.org/download/nginx-$NGINX_VERSION.tar.gz | tar zxfv - \
    && mv /opt/nginx-$NGINX_VERSION /opt/nginx \
    && cd /opt/nginx \
    && ./configure --with-compat \
        --add-dynamic-module=/opt/ngx_http_geoip2_module \
        --add-dynamic-module=/opt/headers-more-nginx-module \
    && make modules

# Stage 2: Final image
FROM node:22-alpine

# Install Nginx runtime dependencies
RUN apk --update --no-cache add \
        bash \
        libmaxminddb \
        pcre2 \
        zlib \
        openssl

# Create the nginx user and group
RUN addgroup -S nginx && adduser -S nginx -G nginx

# Create the default Nginx static file directory
RUN mkdir -p /usr/share/nginx/html

# Copy default Nginx static files from the nginx:alpine image
COPY --from=nginx:alpine /usr/share/nginx/html /usr/share/nginx/html

# Copy Nginx and modules from the builder stage
COPY --from=builder /usr/sbin/nginx /usr/sbin/nginx
COPY --from=builder /etc/nginx /etc/nginx
COPY --from=builder /usr/lib/nginx /usr/lib/nginx
COPY --from=builder /var/log/nginx /var/log/nginx
COPY --from=builder /var/cache/nginx /var/cache/nginx
COPY --from=builder /opt/nginx/objs/ngx_http_geoip2_module.so /usr/lib/nginx/modules
COPY --from=builder /opt/nginx/objs/ngx_http_headers_more_filter_module.so /usr/lib/nginx/modules

# Load modules and set permissions
RUN chmod -R 644 /usr/lib/nginx/modules/*.so \
    && sed -i '1iload_module \/usr\/lib\/nginx\/modules\/ngx_http_geoip2_module.so;' /etc/nginx/nginx.conf \
    && sed -i '2iload_module \/usr\/lib\/nginx\/modules\/ngx_http_headers_more_filter_module.so;' /etc/nginx/nginx.conf

# Copy GeoIP2 configuration and database
COPY nginx-geoip2.conf /etc/nginx/conf.d/geoip2.conf
COPY dbip-country-lite.mmdb /etc/nginx/conf.d/dbip-country-lite.mmdb

# Verify Node.js installation
RUN node -v && npm -v

# Expose ports
EXPOSE 80

# Start Nginx
CMD ["nginx", "-g", "daemon off;"]