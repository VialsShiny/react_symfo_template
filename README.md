# 🚀 Template Fullstack React + Symfony (Docker Ready)

Ce template permet de démarrer rapidement une application fullstack avec :

- ⚛️ Frontend : React (Vite)
- 🧩 Backend : Symfony (API)
- 🐳 Docker intégré (dev + build)
- 🔧 Configuration minimale à faire :
  - installer les dépendances
  - copier `.env.example` → `.env` côté Symfony

---

# 📁 Structure du projet

```

project-root/
│
├── backend/          # Symfony API
│   ├── Dockerfile
│   ├── .env.example
│   ├── composer.json
│   └── ...
│
├── frontend/         # React (Vite)
│   ├── Dockerfile
│   ├── package.json
│   └── ...
│
├── docker-compose.yml
└── README.md

````

---

# 🐳 Docker Compose (root)

```yaml
services:
  frontend:
    build: ./frontend/
    container_name: green_market
    ports:
      - "5173:5173"
    volumes:
      - ./:/frontend
      - /app/node_modules
  backend:
    build: ./backend/
    container_name: symfony_app
    volumes:
      - ./backend/:/var/www/html:delegated
      - /var/www/html/vendor
    depends_on:
      - db
    ports:
      - "8000:80"
  db:
    image: mysql:8.0
    container_name: symfony_db
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: symfony
      MYSQL_USER: user
      MYSQL_PASSWORD: password
    ports:
      - "3306:3306"
    volumes:
      - db_data:/var/lib/mysql
  phpmyadmin:
    image: phpmyadmin:latest
    container_name: symfony_phpmyadmin
    environment:
      PMA_HOST: db
      PMA_USER: user
      PMA_PASSWORD: password
    ports:
      - "8080:80"
    depends_on:
      - db
volumes:
  db_data:
````

---

# ⚙️ Backend Symfony (Dockerfile)

```dockerfile
FROM php:8.3-apache

RUN apt-get update && apt-get install -y \
        libicu-dev libzip-dev zip unzip git \
    && docker-php-ext-install pdo pdo_mysql intl zip opcache

# Opcache + JIT
RUN echo "opcache.enable=1\n\
opcache.memory_consumption=256\n\
opcache.max_accelerated_files=20000\n\
opcache.revalidate_freq=0\n\
opcache.validate_timestamps=0\n\
opcache.jit=tracing\n\
opcache.jit_buffer_size=256M" \
    > /usr/local/etc/php/conf.d/opcache.ini

# APCu
RUN pecl install apcu && docker-php-ext-enable apcu
RUN echo "apc.enable_cli=1" > /usr/local/etc/php/conf.d/apcu.ini

RUN a2enmod rewrite

RUN sed -i 's|/var/www/html|/var/www/html/public|g' \
    /etc/apache2/sites-available/000-default.conf

RUN echo '<Directory /var/www/html/public>\n\
    AllowOverride All\n\
    Require all granted\n\
</Directory>' >> /etc/apache2/sites-available/000-default.conf

RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

WORKDIR /var/www/html

COPY composer.json composer.lock ./
RUN composer install --optimize-autoloader --no-scripts

RUN echo '#!/bin/bash\n\
cat > /var/www/html/public/.htaccess << EOF\n\
DirectoryIndex index.php\n\
<IfModule mod_rewrite.c>\n\
    RewriteEngine On\n\
    RewriteCond %{REQUEST_FILENAME} !-f\n\
    RewriteRule ^ index.php [QSA,L]\n\
</IfModule>\n\
EOF\n\
sleep 2\n\
php bin/console cache:clear --env=dev 2>/dev/null || true\n\
php bin/console cache:warmup --env=dev 2>/dev/null || true\n\
apache2-foreground' > /usr/local/bin/docker-entrypoint.sh \
    && chmod +x /usr/local/bin/docker-entrypoint.sh

ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]
```

---

# ⚛️ Frontend React (Dockerfile)

```dockerfile
FROM node:20-slim

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 5173

CMD ["npm", "run", "dev", "--", "--host", "0.0.0.0"]
```

---

# 🔧 Installation (setup initial)

## 1. Cloner le projet

```bash
git clone https://github.com/VialsShiny/react_symfo_template
cd react_symfo_template
```

---

## 2. Backend Symfony

```bash
cd backend
copy .env.example .env
composer install
```

---

## 3. Frontend React

```bash
cd frontend
npm install
```

---

## 4. Lancer Docker

```bash
docker compose up -d --build
```

---

# 🌍 Accès application

| Service  | URL                                            |
| -------- | ---------------------------------------------- |
| Frontend | [http://localhost:5173](http://localhost:5173) |
| Backend  | [http://localhost:8000](http://localhost:8000) |

---

# 🔥 Features incluses

* Hot reload React (Vite)
* API Symfony accessible directement
* Volumes Docker pour dev
* Setup minimal sans config complexe

---

# 🧠 Notes importantes

* Symfony utilise `.env.example` → à copier en `.env`
* Node modules sont isolés dans Docker volume
* Pas besoin d’installation globale locale
* Compatible Windows / Mac / Linux

---

# 🚀 Améliorations possibles

* PostgreSQL / MySQL ajouté via Docker
* Nginx reverse proxy
* JWT Auth (LexikJWTAuthenticationBundle)
* Docker multi-stage production build
* Symfony Messenger + Redis

---

💡 *Template prêt pour démarrer un projet fullstack rapidement sans configuration complexe.*
