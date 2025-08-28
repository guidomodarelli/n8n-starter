Estoy usando una VPS de Hostinger con la instalación automática del panel **Dokploy** sobre Ubuntu.

* ¿Dónde queda instalado Dokploy en el sistema?
* ¿Se ejecuta dentro de **Docker/Docker Compose** o se instala como servicios nativos del sistema?
* ¿En qué rutas están los **binarios**, la **configuración** y los **logs**?
* Sé que **Traefik** está en `/etc/dokploy/traefik`, pero no encuentro Dokploy. ¿También se ubica bajo `/etc/dokploy`?
* ¿Cómo se realizó exactamente la instalación (contenedores, paquetes, script), para poder reproducirla?

---

Resumen corto: en el template de Hostinger “Ubuntu + Dokploy” **todo corre en Docker (con Docker Swarm)**. No hay
binarios “del sistema” tipo `/usr/bin/dokploy`. En el host vas a ver **config bajo `/etc/dokploy`**, y los **datos**
(Postgres/Redis) en **volúmenes de Docker**. Traefik es un contenedor aparte y su config vive en `/etc/dokploy/traefik`.
([support.hostinger.com][1], [docs.dokploy.com][2])

### ¿Se instala como Docker o como servicios nativos?

* **Docker + Swarm**: el instalador oficial inicializa Swarm, crea la red `dokploy-network` y levanta **servicios**
  `dokploy`, `dokploy-postgres`, `dokploy-redis`, y un contenedor `dokploy-traefik`. ([docs.dokploy.com][3])

  * Lo podés ver con:

    ```bash
    docker service ls
    docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}"
    docker network ls | grep dokploy
    ```

### ¿Dónde están binarios, configuración y logs?

* **Binarios**: no hay binarios instalados en el host; la app es la imagen `dokploy/dokploy` corriendo como servicio de
  Swarm, y Traefik/Redis/Postgres son contenedores. ([docs.dokploy.com][3])
* **Configuración (host)**:

  * Carpeta raíz: `/etc/dokploy` (se crea en la instalación y se monta al contenedor principal). ([docs.dokploy.com][3])
  * **Traefik**:

    * Estático: `/etc/dokploy/traefik/traefik.yml`
    * Dinámico: `/etc/dokploy/traefik/dynamic/`
    * Certs de Let’s Encrypt: `/etc/dokploy/traefik/dynamic/acme.json` (recomendado **chmod 600**). ([docs.dokploy.com][3], [GitHub][4])
* **Datos (volúmenes de Docker)**:

  * Postgres: volumen `dokploy-postgres-database`
  * Redis: volumen `redis-data-volume`
  * Físicamente (si necesitás verlos): `/var/lib/docker/volumes/<nombre>/_data` (mantené esto solo para inspección/backup). ([docs.dokploy.com][5])
* **Logs**:

  * Operativamente:

    ```bash
    docker service logs dokploy           # UI
    docker service logs dokploy-postgres  # DB
    docker service logs dokploy-redis     # Redis
    docker logs dokploy-traefik           # Traefik (este es 'docker run', no servicio)
    ```

    ([docs.dokploy.com][6])
  * En disco (driver por defecto `json-file`):
    `/var/lib/docker/containers/<id>/<id>-json.log`. ([Docker Documentation][7], [Sematext][8])

### ¿Dokploy también se ubica bajo `/etc/dokploy`?

Sí. Esa carpeta es **el** punto de montaje que usa Dokploy; dentro vas a encontrar el subárbol de Traefik y cualquier
archivo que Dokploy escriba allí. (Tu ruta de Traefik es la correcta). ([docs.dokploy.com][3])

### ¿Cómo fue la instalación “automática” y cómo reproducirla?

* Hostinger entrega un VPS con Dokploy **preinstalado** y accesible en `http://IP:3000`. ([support.hostinger.com][1])
* Debajo de eso, utilizan (o equivalen a) **el instalador oficial**, que básicamente hace esto: instala Docker si falta,
  **inicializa Swarm**, crea `/etc/dokploy`, levanta los servicios y **arranca Traefik** con los binds a esa carpeta.
  Para reproducirlo en cualquier Ubuntu limpio:

  ```bash
  curl -sSL https://dokploy.com/install.sh | sh
  ```

  (Ese script crea `/etc/dokploy`, `dokploy-network`, los servicios/volúmenes y publica la UI en el **puerto 3000**). ([docs.dokploy.com][2])

### Chequeos rápidos en tu VPS

```bash
# 1) Servicios y contenedores
docker service ls
docker ps | egrep 'dokploy|traefik|postgres|redis'

# 2) Config local
sudo ls -la /etc/dokploy
sudo ls -la /etc/dokploy/traefik /etc/dokploy/traefik/dynamic

# 3) Volúmenes
docker volume ls | egrep 'dokploy|redis'
docker volume inspect dokploy-postgres-database | jq '.[0].Mountpoint'  # si tenés jq

# 4) Logs
docker service logs --tail=200 dokploy
docker logs --tail=200 dokploy-traefik
```

[1]: https://support.hostinger.com/en/articles/9822596-how-to-use-the-dokploy-vps-template "How to Use the Dokploy VPS Template at Hostinger | Hostinger Help Center"
[2]: https://docs.dokploy.com/docs/core/installation "Installation | Dokploy"
[3]: https://docs.dokploy.com/docs/core/manual-installation "Manual Installation | Dokploy"
[4]: https://github.com/Dokploy/dokploy/issues/259?utm_source=chatgpt.com "Traefik certificate provisioning fails when acme.json has ..."
[5]: https://docs.dokploy.com/docs/core/uninstall "Uninstall | Dokploy"
[6]: https://docs.dokploy.com/docs/core/troubleshooting "Troubleshooting | Dokploy"
[7]: https://docs.docker.com/engine/logging/configure/?utm_source=chatgpt.com "Configure logging drivers"
[8]: https://sematext.com/blog/docker-logs-location/?utm_source=chatgpt.com "Docker Logs Location: Where Are Container Logs Stored"
