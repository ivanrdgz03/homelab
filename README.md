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

En el fichero .env.example hay una plantilla con las variables observadas en `docker-compose.yml`. Revisa y adapta según tus rutas, UID/GID y preferencias.

Consejo: Usa UID/PGID del usuario del host que debe tener acceso a los datos para evitar problemas de permisos.
- Permisos en volúmenes: Si ves errores de permisos, revisa PUID/PGID y propietarios en el host.
- Conflictos de puertos: Pi-hole mapea 80 a 82 en host para evitar conflicto con NPM. Revisa si algún servicio usa el mismo puerto y modifica según convenga.
