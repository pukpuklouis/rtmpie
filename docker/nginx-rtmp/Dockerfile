FROM datarhei/nginx-rtmp:latest

# Copy custom NGINX config
COPY nginx.conf /usr/local/nginx/conf/nginx.conf

# Create directories for thumbnails and recordings
RUN mkdir -p /var/thumbnails /tmp/thumbnail_recordings && \
    chmod -R 775 /var/thumbnails /tmp/thumbnail_recordings

# Copy and make executable the thumbnail script
COPY create-thumbnail.sh /usr/local/bin/create-thumbnail.sh
RUN chmod +x /usr/local/bin/create-thumbnail.sh

# Expose RTMP and HTTP ports
EXPOSE 1935 8080

# Default CMD from base image should be fine
