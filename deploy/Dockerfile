# Container image that serves the static site with nginx.
# Build from the project root (the folder containing index.html):
#   docker build -f deploy/Dockerfile -t hendersonshapiro .
#   docker run -p 8080:80 hendersonshapiro      # http://localhost:8080
FROM nginx:1.27-alpine
COPY . /usr/share/nginx/html/
COPY deploy/nginx.docker.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
