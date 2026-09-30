# Despliegue de nuestra nube local y privada con Nextcloud en Ubuntu / Debian

En esta práctica desplegaremos un servidor de almacenamiento en la nube privado y colaborativo dentro de la red local utilizando **Docker**, **Nextcloud** y **MariaDB** en Linux.

---

## Prerrequisitos e Instalación de Docker

Si aún no se tiene Docker instalado en el sistema, ejecutar en la terminal:

```bash
# 1. Actualizar repositorios e instalar Docker + Docker Compose Plugin
sudo apt update
sudo apt install -y docker.io docker-compose-v2

# 2. Agregar tu usuario al grupo docker (para no necesitar 'sudo' en cada comando)
sudo usermod -aG docker $USER

# 3. Aplicar el cambio de grupo a la sesión actual
newgrp docker
```

---

## Paso 1: Obtener la IP y validar la red antes de empezar

1. **Verificación de red:** Asegurarse de que tanto la PC que hará de servidor como las demás computadoras y dispositivos móviles estén conectados a una red sin aislamiento de clientes (*AP Isolation*). En caso de trabajar en una máquina virtual (VirtualBox/VMware), es importante que la placa de red esté configurada en modo **Adaptador Puente (Bridged)** y no en **NAT**.
2. Ejecutar el siguiente comando para obtener únicamente la dirección IP local que se necesita para el proyecto:

```bash
hostname -I | awk '{print $1}'
```

---

## Paso 2: Preparar la carpeta del proyecto

1. Abrir la terminal.
2. Crear y entrar a la carpeta del taller en Escritorio: 

```bash
cd ~/Escritorio # cd ~/Desktop si se está en inglés 
mkdir nube-local
cd nube-local
```

---

## Paso 3: Crear el archivo de configuración `.env`

Crear el archivo `.env` ejecutando `nano .env` y pegar el siguiente contenido (reemplazando `TU_IP` por la IP obtenida en el Paso 1):

```env
# Dirección IP local de esta PC (cambiar según lo que devuelva hostname -I)
HOST_IP=TU_IP

# Contraseñas de la Base de Datos
MYSQL_ROOT_PASSWORD=RootPasswordSegura123
MYSQL_PASSWORD=NextcloudDbPass123
MYSQL_DATABASE=nextcloud
MYSQL_USER=nextcloud

# Usuario Administrador inicial de Nextcloud
NEXTCLOUD_ADMIN_USER=admin
NEXTCLOUD_ADMIN_PASSWORD=admin123
```

> **¿Por qué definimos `HOST_IP` acá?** Centralizar la IP en el archivo `.env` permite inyectarla automáticamente en la configuración inicial del contenedor sin tener que ejecutar múltiples comandos manuales por consola después de la instalación. 

---

## Paso 4: Crear el archivo `compose.yaml`

Ejecutar `nano compose.yaml` y pegar el siguiente texto:

```yaml
services:
  db:
    image: mariadb:10.11
    container_name: nextcloud-mariadb
    restart: always
    command: --transaction-isolation=READ-COMMITTED --log-bin=binlog --binlog-format=ROW
    volumes:
      - db_data:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
      - MYSQL_DATABASE=${MYSQL_DATABASE}
      - MYSQL_USER=${MYSQL_USER}
    networks:
      - nextcloud-net

  app:
    image: nextcloud:apache
    container_name: nextcloud-app
    restart: always
    ports:
      - "8080:80"
    volumes:
      - nextcloud_data:/var/www/html
    environment:
      - MYSQL_HOST=db
      - MYSQL_DATABASE=${MYSQL_DATABASE}
      - MYSQL_USER=${MYSQL_USER}
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
      - NEXTCLOUD_ADMIN_USER=${NEXTCLOUD_ADMIN_USER}
      - NEXTCLOUD_ADMIN_PASSWORD=${NEXTCLOUD_ADMIN_PASSWORD}
      - NEXTCLOUD_TRUSTED_DOMAINS=localhost nextcloud-app app localhost:8080 ${HOST_IP}:8080 ${HOST_IP}
      - OVERWRITECLIURL=http://${HOST_IP}:8080
      - OVERWRITEHOST=${HOST_IP}:8080
      - OVERWRITEPROTOCOL=http
    depends_on:
      - db
    networks:
      - nextcloud-net

volumes:
  db_data:
  nextcloud_data:

networks:
  nextcloud-net:
    driver: bridge
```

> **Nota:** La imagen oficial `nextcloud:apache` acepta estas variables de entorno nativas para configurar los dominios de confianza (`NEXTCLOUD_TRUSTED_DOMAINS`) durante la instalación inicial. Además, las variables `OVERWRITE*` crean automáticamente el archivo `reverse-proxy.config.php` (esencial para las conexiones en red local) cada vez que el contenedor arranca. Así, cuando se inicia sesión desde la app de Android o iOS, el servidor sabe que debe devolver el token de acceso a `http://HOST_IP:8080` y no a `localhost`.

---

## Paso 5: Levantar los contenedores

En la terminal, dentro de la carpeta `nube-local`, ejecutar:

```bash
docker compose up -d
```

> **Importante:** La primera vez puede tardar entre 2 y 3 minutos mientras descarga las imágenes e inicializa la base de datos. Es crucial esperar a que finalice antes de intentar entrar desde el navegador.

1. Para verificar que Nextcloud haya terminado de instalarse, ejecutar el siguiente comando y comprobar que en las últimas líneas aparezca `apache2 -D FOREGROUND`:
   ```bash
   docker compose logs app | tail -n 5
   ```
2. Verificar que los **dos** contenedores estén en estado `Up`:
   ```bash
   docker compose ps
   ```

---

## Paso 6: Conectar desde la PC y el Dispositivo Móvil

1. **Desde el navegador de la PC:**
   * Ingresar a `http://TU_IP:8080` (por ejemplo, `http://10.0.7.13:8080`).
   * Iniciar sesión con el usuario **`admin`** y la contraseña **`admin123`**.
1. **Desde el celular (Navegador o App oficial de Nextcloud):**
   * Asegurarse de estar conectado a la misma red Wi-Fi que el servidor.
   * Escribe la dirección completa incluyendo `http://` y el puerto `:8080`:
     `http://TU_IP:8080`

---

## Comandos útiles para administrar el servidor

* **Pausar el servidor al terminar el día:**
  ```bash
  docker compose stop
  ```
* **Volver a iniciar el servidor:**
  ```bash
  docker compose up -d
  ```
* **Ver los registros/logs en vivo ante cualquier duda:**
  ```bash
  docker compose logs -f
  ```
* **¿Que ocurre si cambio de red o de IP?**
  Editá la variable `HOST_IP` en tu archivo `.env` con la nueva IP y ejecuta estos dos comandos para actualizar el contenedor y autorizar el nuevo dominio:
  ```bash
  docker compose up -d
  docker compose exec --user www-data app php occ config:system:set trusted_domains 4 --value="NUEVA_IP:8080" #Acá la nueva IP
  ```
