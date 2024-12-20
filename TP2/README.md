# I. Packaging de l'app Web

🌞 docker-compose.yml

```
version: '3.1'

services:
  web:
    build: .
    environment:
      DB_HOST : "db"
      PORT: "80"
    ports: 
      - "80:80"

  db:
    image: mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root
    volumes:
      - "./seed.sql:/docker-entrypoint-initdb.d/seed.sql"

  PhpMyAdmin:
    image: phpmyadmin
    restart: always
    ports:
      - 8080:80
```


# TP2 dév : packaging et environnement de dév local

## I. Packaging

### 1. Calculatrice

🌞 Le lien vers ton dépôt

[lien du repo](https://github.com/brendan-vis/Calculatrice)

### 2. Chat room

[lien du repo](https://github.com/brendan-vis/Calculatrice)