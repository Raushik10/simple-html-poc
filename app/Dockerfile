FROM nginx:stable-alpine
RUN apk upgrade --no-cache
COPY index.html /usr/share/nginx/html/index.html
COPY config.js /usr/share/nginx/html/config.js
RUN sed -i "s/__VERSION__/${VERSION}/" /usr/share/nginx/html/index.html
