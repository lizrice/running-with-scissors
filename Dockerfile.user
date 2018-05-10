FROM nginx:capsh

# Serve from port 8080 since you can't bind to port 80 as non-root
COPY ./default.conf /etc/nginx/conf.d/default.conf

RUN touch /var/run/nginx.pid && \
  chown -R 1001:1001 /var/run/nginx.pid && \
  chown -R 1001:1001 /var/cache/nginx

USER 1001