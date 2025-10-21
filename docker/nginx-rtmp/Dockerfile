# syntax=docker/dockerfile:1.4

######################################################################
# Builder stage
######################################################################
FROM alpine:3.19 AS builder

ARG NGINX_VERSION=1.29.2
ARG NGINX_RTMP_VERSION=1.2.12
ARG NGINX_PREFIX=/opt/nginx

WORKDIR /tmp/build

# Install build dependencies
RUN apk add --no-cache \
        build-base \
        openssl-dev \
        pcre-dev \
        zlib-dev \
        linux-headers \
        pkgconfig \
        perl \
        curl

# Download and extract sources
RUN curl -fsSL "https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz" | tar xz && \
    curl -fsSL "https://github.com/winshining/nginx-http-flv-module/archive/refs/tags/v${NGINX_RTMP_VERSION}.tar.gz" | tar xz

WORKDIR /tmp/build/nginx-${NGINX_VERSION}

RUN echo "Nginx version: ${NGINX_VERSION}" && echo "RTMP version: ${NGINX_RTMP_VERSION}" && ls -la /tmp/build/

RUN MODULE_PATH="/tmp/build/nginx-http-flv-module-${NGINX_RTMP_VERSION}" && \
    ls -la "${MODULE_PATH}/config" && \
    ./configure \
        --prefix="${NGINX_PREFIX}" \
        --conf-path="${NGINX_PREFIX}/conf/nginx.conf" \
        --sbin-path="${NGINX_PREFIX}/sbin/nginx" \
        --pid-path=/var/run/nginx.pid \
        --lock-path=/var/lock/nginx.lock \
        --http-client-body-temp-path=/tmp/nginx/client_body \
        --http-proxy-temp-path=/tmp/nginx/proxy \
        --http-fastcgi-temp-path=/tmp/nginx/fastcgi \
        --with-http_ssl_module \
        --with-http_stub_status_module \
        --without-http_rewrite_module \
        --without-http_gzip_module \
        --add-module="${MODULE_PATH}"

RUN make -j"$(nproc)" V=1

RUN make install

######################################################################
# Final runtime stage
######################################################################
FROM alpine:3.19

ARG NGINX_PREFIX=/opt/nginx

# Install runtime dependencies (FFmpeg included)
RUN apk add --no-cache \
        ffmpeg \
        libstdc++ \
        pcre \
        openssl \
        ca-certificates

# Create nginx user and group
RUN addgroup -S nginx && adduser -S -D -H -G nginx nginx

# Copy built artifacts from builder
COPY --from=builder "${NGINX_PREFIX}" "${NGINX_PREFIX}"

# Provide default nginx configuration (can be overridden via volume)
COPY nginx.conf "${NGINX_PREFIX}/conf/nginx.conf"

# Copy helper scripts
COPY create-thumbnail.sh /usr/local/bin/create-thumbnail.sh
RUN chmod +x /usr/local/bin/create-thumbnail.sh

# Prepare writable directories
RUN mkdir -p /var/thumbnails /tmp/thumbnail_recordings /tmp/nginx /opt/nginx/logs /opt/nginx/uwsgi_temp /opt/nginx/proxy_temp /opt/nginx/fastcgi_temp /opt/nginx/scgi_temp && \
    chown -R nginx:nginx /var/thumbnails /tmp/thumbnail_recordings /tmp/nginx /opt/nginx/logs /opt/nginx/uwsgi_temp /opt/nginx/proxy_temp /opt/nginx/fastcgi_temp /opt/nginx/scgi_temp

RUN ln -sf /dev/stdout /var/log/nginx/access.log && \
    ln -sf /dev/stderr /var/log/nginx/error.log

EXPOSE 1935 8080

STOPSIGNAL SIGQUIT

USER nginx

CMD ["nginx", "-g", "daemon off;"]
