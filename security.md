# Setting Up HTTPS for Frontend and Backend Services Using Nginx and Docker Compose

## Prerequisites

- Docker and Docker Compose installed on your server.
- OpenSSL installed on your server.

## Steps

### 1. Generate Self-Signed SSL Certificates

Use OpenSSL to generate self-signed SSL certificates. Replace `185.189.167.175` with your actual IP address.

```sh
mkdir -p certs
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout certs/nginx-selfsigned.key -out certs/nginx-selfsigned.crt -subj "/CN=185.189.167.175"
openssl dhparam -out certs/dhparam.pem 2048
```

### 2. Create Nginx Configuration File

Create an Nginx configuration file `nginx.conf` to handle both frontend and backend proxying with HTTPS, using different ports.

**nginx.conf**:

```nginx
server {
    listen 80;
    server_name 185.189.167.175;

    location / {
        return 301 https://$host$request_uri;
    }
}

server {
    listen 443 ssl;
    server_name 185.189.167.175;

    ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
    ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;
    ssl_dhparam /etc/ssl/certs/dhparam.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256';
    ssl_session_timeout 1d;
    ssl_session_cache shared:MozSSL:10m;
    ssl_session_tickets off;

    ssl_stapling on;
    ssl_stapling_verify on;

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
    server_name 185.189.167.175;

    ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
    ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;
    ssl_dhparam /etc/ssl/certs/dhparam.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256';
    ssl_session_timeout 1d;
    ssl_session_cache shared:MozSSL:10m;
    ssl_session_tickets off;

    ssl_stapling on;
    ssl_stapling_verify on;

    location / {
        proxy_pass http://backend:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### 3. Modify Docker Compose Configuration

Update your `docker-compose.yml` file to include the Nginx service and mount the necessary SSL certificate files.

**docker-compose.yml**:

```yaml
version: '3'

services:
  backend:
    environment:
      - SPRING_DATASOURCE_URL=jdbc:postgresql://db:5432/course
      - SPRING_DATASOURCE_USERNAME=admin
      - SPRING_DATASOURCE_PASSWORD=admin
      - ALLOWED_ORIGINS=https://185.189.167.175:8443
    build:
      context: ./backend
      dockerfile: Dockerfile
    expose:
      - "8080"
    networks:
      - app-network

  frontend:
    environment:
      - REACT_APP_BACKEND_URL=https://185.189.167.175:8443
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
    image: nginx:latest
    ports:
      - "80:80"
      - "443:443"
      - "8443:8443"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
      - ./certs/nginx-selfsigned.crt:/etc/ssl/certs/nginx-selfsigned.crt
      - ./certs/nginx-selfsigned.key:/etc/ssl/private/nginx-selfsigned.key
      - ./certs/dhparam.pem:/etc/ssl/certs/dhparam.pem
    depends_on:
      - frontend
      - backend
    networks:
      - app-network

networks:
  app-network:
    driver: bridge
```

### 4. Build and Run Docker Compose

Build and run the Docker Compose services:

```sh
docker-compose up --build -d
```

### 5. Access Your Application

Open a web browser and navigate to:
- `https://185.189.167.175` to access your frontend application over HTTPS.
- `https://185.189.167.175:8443` to access your backend API over HTTPS.
