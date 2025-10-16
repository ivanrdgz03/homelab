# Homelab - Docker Compose

Este repositorio contiene la composición docker-compose para mi homelab. El objetivo del README es explicar qué contiene, cómo desplegarlo y qué variables de entorno deben configurarse, tomando como referencia el archivo `docker-compose.yml`.

---

## Contenido principal (servicios destacados)

Basado en el `docker-compose.yml`, estos son los servicios incluidos y sus puertos y funciones principales:

- Portainer
  - Gestión UI de Docker
  - Puerto: 9000
  - Sirve para administrar contenedores

- DuckDNS
  - Actualiza el registro dinámico (dominio)
  - Usa token y subdominio (variables .env)

- Jellyfin
  - Servidor multimedia
  - Puerto: 8096
  - Usa `/dev/dri` para transcoding (sólo si la GPU está disponible)

- Nginx Proxy Manager (NPM)
  - Proxy reverso y gestión de certificados automática (Let's Encrypt)
  - Puertos: 80, 443, panel 81
  - Depende de `nginx_db` y `authelia` para funcionar

- Authelia
  - Autenticación/SSO para servicios detrás del proxy
  - Puerto mapeado: 9092 (host) -> 9091 (contenedor)

- nginx_db (MariaDB)
  - Base de datos para Nginx Proxy Manager
  - Puerto: 3306

- Sonarr y Radarr
  - Gestión automática de series y películas
  - Sonarr: 8989
  - Radarr: 7878

- Jellyseerr
  - Interface para solicitudes de contenido
  - Puerto: 5055

- qBittorrent
  - Cliente torrent con interfaz web
  - WebUI: 9093
  - Puertos de transferencia: 6881 TCP/UDP

- Torproxy
  - Proxy sobre Tor para tener una capa extra de privacidad
  - Puertos: 8118, 9050

- Jackett
  - Indexadores para Sonarr y Radarr
  - Puerto: 9117

- Pi-hole
  - Servidor DNS configurado para bloquear publicidad, también tiene funciones de servidor DHCP
  - Puertos: 53 TCP/UDP y 82 (host) mapeado a 80 (contenedor)

- Speedtest Tracker
  - Monitor de velocidad periódica
  - Puerto: 85

- WireGuard (wg-easy)
  - Gestión rápida de WireGuard con una interfaz web simple
  - Puerto UDP: 51820 y 51821 TCP auxiliar

- Watchtower
  - Auto-actualización de imágenes (notificaciones por Telegram configuradas). También se puede configurar para que aplique solo las actualizaciones pero es mejor aplicarlas manualmente para evitar posibles errores.

- RustDesk (hbbs / hbbr)
  - Servidores para RustDesk (host network)


## Requisitos

- Docker
- docker-compose
- Un archivo `.env` con las variables necesarias (ver plantilla abajo)
- Reglas de firewall / NAT para exponer puertos ¡MUCHO CUIDADO CON ESTO!
---

## Preparación y despliegue rápido

1. Clona el repositorio:
   `git clone https://github.com/ivanrdgz03/homelab.git`

2. Copia la plantilla de variables de entorno y edítala:
   `cp .env.example .env`
   - Rellena con tus valores (usuarios, IDs, rutas, tokens, dominios, etc.)

3. Levanta los servicios:
   `docker compose up -d`
   
4. Para actualizar imágenes:
   `docker compose pull
   docker compose up -d`

## Variables de entorno (.env) — plantilla

A continuación una plantilla con las variables observadas en `docker-compose.yml`. Revisa y adapta según tus rutas, UID/GID y preferencias.

```env
# Sistema
TIMEZONE=Europe/Madrid
EMAIL=tu@correo.com
PASSWORD=changeme

# PORTAINER
PORTAINER_FOLDER=/path/to/portainer

# DuckDNS
DUCKDNS_USER=1000
DUCKDNS_GROUP=1000
DOMINIO=subdominio
TOKEN_DUCKDNS=token_de_duckdns
DUCKDNS_FOLDER=/path/to/duckdns

# Medi
MEDIA=/path/to/media

# Jellyfin
JELLYFIN_USER=1000
JELLYFIN_GROUP=1000
JELLYFIN_CONFIG=/path/to/jellyfin/config
JELLYFIN_CACHE=/path/to/jellyfin/cache

# Nginx Proxy Manager
DNS_NGINX=1.1.1.1
DB_NGINX_USER=npm_user        
NGINX_DB_USER=npm_user       
NGINX_CONFIG=/path/to/npm

# Authelia
AUTHELIA_FOLDER=/path/to/authelia

# MariaDB (NPM)
NGINX_DB_FOLDER=/path/to/mysql

# Sonarr / Radarr
UID_SONARR=1000
PGID_SONARR=1000
SONARR_FOLDER=/path/to/sonarr

UID_RADARR=1000
PGID_RADARR=1000
RADARR_FOLDER=/path/to/radarr

# Jellyseerr
UID_JELLYSEERR=1000
PGID_JELLYSEERR=1000
JELLYSEERR_FOLDER=/path/to/jellyseerr

# qBittorrent
UID_QBIT=1000
PGID_QBIT=1000
QBIT_FOLDER=/path/to/qbittorrent

# Torproxy
TORPROXY_PASSWORD=secret_password

# Jackett
UID_SWAG=1000
GID_SWAG=1000
JACKETT_FOLDER=/path/to/jackett

# Pi-hole
UID_PIHOLE=1000
PGID_PIHOLE=1000
PASSWORD_PIHOLE=piholedbpassword
PIHOLE_FOLDER=/path/to/pihole

# Speedtest Tracker
UID_SPEEDTEST=1000
PGID_SPEEDTEST=1000
SPEEDTEST_FOLDER=/path/to/speedtest
URL_SPEEDTEST=https://speedtest.example.com
SPEEDTEST_KEY=clave_app_speedtest
PASSWORD_SPEEDTEST=adminpassword

# Telegram bot
TELEGRAM_TOKEN=123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11
TELEGRAM_CHATID=12345678

# DNS
SERVIDOR_DNS=1.1.1.1

# WireGuard (wg-easy)
WG_DNS=1.1.1.1
WG_HOST=vpn.example.com
PASSWORD_HASH=hash_de_contraseña_wg
WG_FOLDER=/path/to/wg

# RustDesk
RD_FOLDER=/path/to/rustdesk
```
Consejo: Usa UID/PGID del usuario del host que debe tener acceso a los datos para evitar problemas de permisos.
- Permisos en volúmenes: Si ves errores de permisos, revisa PUID/PGID y propietarios en el host.
- Conflictos de puertos: Pi-hole mapea 80 a 82 en host para evitar conflicto con NPM. Revisa si algún servicio usa el mismo puerto y modifica según convenga.
