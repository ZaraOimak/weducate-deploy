FROM nginx:latest

# Install Certbot
RUN apt-get update && apt-get install -y certbot python3-certbot-nginx

# Copy the initial Nginx configuration for Certbot validation
COPY nginx_temp.conf /etc/nginx/conf.d/default.conf

# Copy the final Nginx configuration
COPY nginx.conf /etc/nginx/nginx.conf

# Create the necessary directory for Certbot
RUN mkdir -p /var/www/certbot

# Obtain certificates and reload Nginx
CMD ["sh", "-c", "nginx && sleep 5 && certbot certonly --webroot -w /var/www/certbot -d weducate.ru -d www.weducate.ru --email emaildlyaege@gmail.com --agree-tos --non-interactive && cp /etc/nginx/nginx.conf /etc/nginx/conf.d/default.conf && nginx -s reload && tail -f /dev/null"]