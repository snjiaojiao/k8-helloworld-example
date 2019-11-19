FROM ubuntu
RUN apt-get update
RUN apt-get install nginx -y
COPY server_c.html /var/www/html/helloz/index.html
COPY site-enable-default /etc/nginx/sites-available/default
EXPOSE 80
CMD ["nginx","-g","daemon off;"]
