FROM alpine:3.19 as builder

ARG NGINX_VERSION=1.26.1
ARG NGINX_RTMP_VERSION=1.2.2

RUN apk --no-cache add build-base libressl-dev pcre-dev zlib-dev linux-headers pkgconfig

RUN mkdir -p /tmp/build && \
    cd /tmp/build && \
    wget -q -O nginx.tar.gz http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz && \
    tar -zxf nginx.tar.gz && \
    wget -q -O nginx-rtmp-module.tar.gz https://github.com/winshining/nginx-http-flv-module/archive/v${NGINX_RTMP_VERSION}.tar.gz && \
    tar -zxf nginx-rtmp-module.tar.gz && \
    cd nginx-${NGINX_VERSION}

RUN cd /tmp/build/nginx-${NGINX_VERSION} && \
    ./configure \
        --prefix=/etc/nginx \
        --sbin-path=/usr/local/sbin/nginx \
        --conf-path=/etc/nginx/nginx.conf \
        --error-log-path=/var/log/nginx/error.log \
        --http-log-path=/var/log/nginx/access.log \
        --pid-path=/var/run/nginx.pid \
        --lock-path=/var/lock/nginx.lock \
        --http-client-body-temp-path=/tmp/nginx/client-body \
        --user=nginx --group=nginx \
        --without-http_rewrite_module \
        --without-http_gzip_module \
        --with-http_ssl_module \
        --with-http_v2_module \
        --add-module=../nginx-http-flv-module-${NGINX_RTMP_VERSION} && \
    make -j $(getconf _NPROCESSORS_ONLN) && \
    cp objs/nginx /tmp/build/

FROM alpine:3.19

LABEL maintainer="Niklas Grießer <niklas@griesser.me>"

RUN apk update && \
    apk --no-cache add \
        libressl \
        libstdc++ \
        ca-certificates \
        ffmpeg \
        su-exec \
        openssl \
        pcre \
        zlib

RUN addgroup -S nginx && \
    adduser -s /sbin/nologin -G nginx -S -D -H nginx

RUN mkdir -p /etc/nginx /var/log/nginx /var/www && \
    chown -R nginx:nginx /var/log/nginx /var/www && \
    chmod -R 775 /var/log/nginx /var/www

RUN mkdir -p /tmp/nginx/client-body /tmp/nginx/dash /tmp/nginx/hls && \
    chown -R nginx:nginx /tmp/nginx && \
    chmod -R 770 /tmp/nginx

RUN mkdir -p /var/thumbnails /tmp/thumbnail_recordings && \
    chown -R nginx:nginx /var/thumbnails && \
    chmod -R 770 /var/thumbnails && \
    chown -R nginx:nginx /tmp/thumbnail_recordings && \
    chmod -R 770 /tmp/thumbnail_recordings

RUN ln -sf /dev/stdout /var/log/nginx/access.log && \
    ln -sf /dev/stderr /var/log/nginx/error.log

EXPOSE 1935

COPY --from=builder /usr/local/sbin/nginx /usr/local/sbin/nginx
RUN chmod 755 /usr/local/sbin/nginx

COPY nginx.conf /etc/nginx/nginx.conf
RUN chmod 644 /etc/nginx/nginx.conf

COPY create-thumbnail.sh /opt/create-thumbnail.sh
RUN chmod +x /opt/create-thumbnail.sh

CMD ["nginx", "-g", "daemon off;"]
