To organize all Nginx-related files into an `nginx` folder and automate the process of obtaining SSL certificates and configuring Nginx, follow these steps:

### Project Structure

Organize your project directory like this:

```
your_project/
├── backend/
├── frontend/
├── nginx/
│   ├── Dockerfile
│   ├── nginx_temp.conf
│   ├── nginx.conf
└── docker-compose.yml
```

### 1. `nginx/Dockerfile`

Create `nginx/Dockerfile` to set up Certbot and Nginx:

```Dockerfile
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
CMD ["sh", "-c", "nginx && sleep 5 && certbot certonly --webroot -w /var/www/certbot -d example.com -d www.example.com --email your-email@example.com --agree-tos --non-interactive && cp /etc/nginx/nginx.conf /etc/nginx/conf.d/default.conf && nginx -s reload && tail -f /dev/null"]
```

### 2. `nginx/nginx_temp.conf`

Create `nginx/nginx_temp.conf`:

```nginx
server {
    listen 80;
    server_name example.com www.example.com;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        return 301 https://$host$request_uri;
    }
}
```

### 3. `nginx/nginx.conf`

Create `nginx/nginx.conf`:

```nginx
server {
    listen 80;
    server_name example.com www.example.com;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        return 301 https://$host$request_uri;
    }
}

server {
    listen 443 ssl;
    server_name example.com www.example.com;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    location / {
        proxy_pass http://frontend:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    listen 8443 ssl;
    server_name example.com www.example.com;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    location / {
        proxy_pass http://backend:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### 4. `docker-compose.yml`

Ensure your `docker-compose.yml` points to the custom Dockerfile and mounts the necessary volumes:

```yaml
version: '3'

services:
  backend:
    environment:
      - SPRING_DATASOURCE_URL=jdbc:postgresql://db:5432/course
      - SPRING_DATASOURCE_USERNAME=admin
      - SPRING_DATASOURCE_PASSWORD=admin
      - ALLOWED_ORIGINS=https://example.com,https://example.com:8443
    build:
      context: ./backend
      dockerfile: Dockerfile
    expose:
      - "8080"
    networks:
      - app-network

  frontend:
    environment:
      - REACT_APP_BACKEND_URL=https://example.com:8443
    build:
      context: ./frontend
      dockerfile: Dockerfile
    expose:
      - "3000"
    networks:
      - app-network

  db:
    image: postgres:16.3
    environment:
      POSTGRES_DB: course
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: admin
    ports:
      - "5432:5432"
    networks:
      - app-network

  nginx:
    build:
      context: ./nginx
      dockerfile: Dockerfile
    ports:
      - "80:80"
      - "443:443"
      - "8443:8443"
    volumes:
      - /etc/letsencrypt:/etc/letsencrypt
      - /var/www/certbot:/var/www/certbot
    depends_on:
      - frontend
      - backend
    networks:
      - app-network

networks:
  app-network:
    driver: bridge
```

### Running the Setup

Build and run your Docker Compose services:

```sh
docker-compose up --build -d
```

This setup will:
1. Install Certbot and configure Nginx.
2. Obtain SSL certificates when the Nginx container starts.
3. Apply the certificates and reload Nginx to serve your frontend and backend over HTTPS.