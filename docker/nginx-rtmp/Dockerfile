FROM ngrie/nginx-rtmp:latest

RUN apk update && apk --no-cache add ffmpeg

RUN mkdir -p /var/thumbnails /tmp/thumbnail_recordings && \
    chown -R nginx:nginx /var/thumbnails && \
    chmod -R 770 /var/thumbnails && \
    chown -R nginx:nginx /tmp/thumbnail_recordings && \
    chmod -R 770 /tmp/thumbnail_recordings

COPY nginx.conf /etc/nginx/nginx.conf

COPY create-thumbnail.sh /opt/create-thumbnail.sh
RUN chmod +x /opt/create-thumbnail.sh
