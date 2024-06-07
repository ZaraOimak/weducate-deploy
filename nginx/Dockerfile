FROM nginx:latest

# Install Certbot
RUN apt-get update && apt-get install -y certbot python3-certbot-nginx

# Copy the initial Nginx configuration for Certbot validation
COPY nginx_temp.conf /etc/nginx/conf.d/default.conf

# Copy the final Nginx configuration
COPY nginx.conf /etc/nginx/nginx_ssl.conf

# Create the necessary directory for Certbot
RUN mkdir -p /var/www/certbot /var/log/certbot

# Start Nginx, run Certbot, and reload Nginx with logging
CMD ["sh", "-c", "
    echo 'Starting Nginx' > /var/log/certbot/step.log;
    nginx;
    echo 'Nginx started' >> /var/log/certbot/step.log;
    sleep 5;
    echo 'Running Certbot' >> /var/log/certbot/step.log;
    certbot certonly --webroot -w /var/www/certbot -d weducate.ru -d www.weducate.ru --email emaildlyaege@gmail.com --agree-tos --non-interactive >> /var/log/certbot/certbot.log 2>&1;
    echo 'Certbot completed' >> /var/log/certbot/step.log;
    cp /etc/nginx/nginx_ssl.conf /etc/nginx/nginx.conf;
    echo 'Nginx configuration updated' >> /var/log/certbot/step.log;
    nginx -s reload;
    echo 'Nginx reloaded' >> /var/log/certbot/step.log;
    tail -f /var/log/certbot/certbot.log
"]