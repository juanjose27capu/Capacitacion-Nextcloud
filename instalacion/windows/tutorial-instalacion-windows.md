# Despliegue de nuestra nube local y privada con Nextcloud en Windows

En esta práctica desplegaremos un servidor de almacenamiento en la nube privado y colaborativo dentro de la red local utilizando **Docker Desktop**, **Nextcloud** y **MariaDB** desde la terminal de Windows (**PowerShell**).

---

## Prerrequisitos e Instalación de Docker Desktop

Si aún no se tiene Docker instalado en el sistema:

1. Descargar e instalar **Docker Desktop para Windows** desde su sitio oficial (asegurándose de dejar marcada la opción **Use WSL 2** durante la instalación).
2. Reiniciar la computadora si el instalador lo solicita.
3. **Importante:** Abrir la aplicación **Docker Desktop** desde el menú Inicio y esperar a que el ícono de la ballena abajo a la izquierda indique que el motor está en ejecución (*Engine running*) antes de usar la terminal.

*(Opcional) Si se desea instalar desde la terminal de Windows (PowerShell), se puede ejecutar:*
```powershell
winget install -e --id Docker.DockerDesktop
```

---

## Paso 1: Obtener la IP y validar la red antes de empezar

1. **Verificación de red:** 
   * Asegurarse de que tanto la PC que hará de servidor como las demás computadoras y dispositivos móviles estén conectados a una red sin aislamiento de clientes (*AP Isolation*). En caso de trabajar en una máquina virtual (VirtualBox/VMware), es importante que la placa de red esté configurada en modo **Adaptador Puente (Bridged)** y no en **NAT**.
   * **Perfil de red en Windows:** Ir a la configuración de Red de Windows y verificar que el perfil de la red esté marcado como Red Privada (si está en *Red Pública*, el Firewall de Windows bloqueará la conexión desde el celular).
2. Abrir **PowerShell** y ejecutar el siguiente comando para obtener únicamente la dirección IP local real de la PC (ignorando los adaptadores virtuales de Docker/WSL):

```powershell
(Get-NetIPConfiguration | Where-Object { $_.IPv4DefaultGateway -ne $null }).IPv4Address.IPAddress
```
*(Alternativamente, se puede ejecutar `ipconfig` y buscar la **Dirección IPv4** correspondiente al **Adaptador de LAN inalámbrica Wi-Fi** o **Ethernet**).*

![Paso 1 - Obtener la IP local](./instalacion/img/windows/paso1.png)

---

## Paso 2: Preparar la carpeta del proyecto

1. En la ventana de **PowerShell**, crear y entrar a la carpeta del taller en el Escritorio:

```powershell
cd $HOME\Desktop
mkdir nube-local
cd nube-local
```

![Paso 2 - Crear carpeta del proyecto](./instalacion/img/windows/paso2.png)

---

## Paso 3: Crear el archivo de configuración `.env`

En PowerShell, ejecutar el siguiente comando para crear el archivo `.env` (evitando que Windows le agregue la extensión `.txt` por error) y abrirlo en el Bloc de notas:

```powershell
New-Item .env -ItemType File -Force; notepad .env
```

Pegar el siguiente contenido en el Bloc de notas (reemplazando `TU_IP` por la IP obtenida en el Paso 1), guardar los cambios (`Ctrl + G` o *Archivo -> Guardar*) y cerrar la ventana:

```env
# Dirección IP local de esta PC (cambiar según lo obtenido en el Paso 1)
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

![Paso 3 - Configurar archivo .env](./instalacion/img/windows/paso3.png)

---

## Paso 4: Crear el archivo `compose.yaml`

Ejecutar en PowerShell el siguiente comando para crear el archivo `compose.yaml` y abrirlo en el Bloc de notas:

```powershell
New-Item compose.yaml -ItemType File -Force; notepad compose.yaml
```

Pegar el siguiente texto, guardar los cambios y cerrar el Bloc de notas:

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

![Paso 4 - Crear compose.yaml](./instalacion/img/windows/paso4.png)

---

## Paso 5: Levantar los contenedores

> **Importante (Verificación previa):** Asegurarse de que la aplicación **Docker Desktop** esté abierta y ejecutándose en segundo plano (verificar que el motor indique *Engine running* con el ícono de la ballena en verde) antes de correr el siguiente comando.

1. En PowerShell, dentro de la carpeta `nube-local`, ejecutar:

```powershell
docker compose up -d
```

*(Si aparece una ventana emergente del Firewall de Windows Defender solicitando permisos para Docker, marcar las casillas de redes privadas/públicas y hacer clic en **Permitir acceso**).*

![Paso 5 - Levantar contenedores](./instalacion/img/windows/paso51.png)

> **Nota sobre los tiempos:** La primera vez puede tardar entre 2 y 3 minutos mientras descarga las imágenes e inicializa la base de datos. Es crucial esperar a que finalice antes de intentar entrar desde el navegador.

2. Para verificar que Nextcloud haya terminado de instalarse, ejecutar el siguiente comando y comprobar que en las últimas líneas aparezca `apache2 -D FOREGROUND`:
   ```powershell
   docker compose logs --tail 5 app
   ```
3. Verificar que los dos contenedores estén en estado `Up` (`running`):
   ```powershell
   docker compose ps
   ```

![Paso 5 - Verificación de contenedores](./instalacion/img/windows/paso52.png)

---

## Paso 6: Conectar desde la PC y el Dispositivo Móvil

1. **Desde el navegador de la PC:**
   * Ingresar a `http://TU_IP:8080` (por ejemplo, `http://10.0.7.13:8080`).
   * Iniciar sesión con el usuario **`admin`** y la contraseña **`admin123`**.

![Paso 6 - Inicio de sesión desde el navegador de la PC](./instalacion/img/windows/paso61.png)

2. **Desde el celular (Navegador o App oficial de Nextcloud):**
   * Asegurarse de estar conectado a la misma red Wi-Fi que el servidor.
   * Escribir la dirección completa incluyendo `http://` y el puerto `:8080`:
     `http://TU_IP:8080`

![Paso 6 - Conexión desde dispositivo móvil](./instalacion/img/windows/paso62.jpeg)

---

## Comandos útiles para administrar el servidor

* **Pausar el servidor al terminar el día:**
  ```powershell
  docker compose stop
  ```
* **Volver a iniciar el servidor:**
  ```powershell
  docker compose up -d
  ```
* **Ver los registros/logs en vivo ante cualquier duda:**
  ```powershell
  docker compose logs -f
  ```
* **¿Que ocurre si cambio de red o de IP?**
  1. Obtener la nueva IP con ``hostname -I | awk '{print $1}'`` 
  2. Editar la variable `HOST_IP` en tu archivo `.env` con la nueva IP 
  3. Ejecutar estos dos comandos para actualizar el contenedor y autorizar el nuevo dominio:
  ```powershell
  docker compose up -d
  docker compose exec --user www-data app php occ config:system:set trusted_domains 4 --value="NUEVA_IP:8080" # < ----- Acá la nueva IP
  ``` 
* **¿El Firewall de Windows bloquea la conexión desde el celular?**
  Si no se puede entrar desde el móvil aunque estén en la misma red, abrir PowerShell **como Administrador** y ejecutar esta regla para habilitar el puerto `8080`:
  ```powershell
  New-NetFirewallRule -DisplayName "Nextcloud Docker" -Direction Inbound -LocalPort 8080 -Protocol TCP -Action Allow
  ```
