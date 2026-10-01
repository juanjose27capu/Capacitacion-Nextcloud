# Cómo eliminar el servidor y limpiar las reglas al finalizar la práctica

En caso de haber cometido error en el despliegue del contedor, volumen de datos, etc. y se quisiera volver a empezar, estos comandos limpian todo registro de actividad y modificaciones que se hicieron durante la práctica. 
**Nota:** Al apagar y eliminar el proyecto, Docker deshace automáticamente el mapeo del puerto `8080` en el sistema y borra las configuraciones de IP asociadas.
## En Ubuntu /Debian

1. Entrar a la carpeta del proyecto y destruir los contenedores, redes y volúmenes de datos:
   ```bash
   cd ~/Escritorio/nube-local
   docker compose down -v --remove-orphans
   ```
2. *(Opcional)* Eliminar la carpeta del proyecto:
   ```bash
   cd ..
   rm -rf nube-local
   ```
3. *(Solo si se abrió manualmente el puerto en el firewall UFW)* Eliminar la regla del puerto `8080`:
   ```bash
   sudo ufw delete allow 8080/tcp
   ```

## En Windows

Para borrar por completo el servidor, las configuraciones de IP y cerrar el puerto en el Firewall de Windows:

1. En PowerShell, dentro de la carpeta del proyecto, destruir los contenedores, redes y volúmenes de datos:
   ```powershell
   cd $HOME\Desktop\nube-local
   docker compose down -v --remove-orphans
   ```
2. *(Opcional)* Eliminar la carpeta del proyecto:
   ```powershell
   cd ..
   Remove-Item -Recurse -Force nube-local
   ```
3. **Si se ejecutó la regla del Firewall de Windows**, abrir PowerShell **como Administrador** y eliminar el permiso del puerto `8080`:
   ```powershell
   Remove-NetFirewallRule -DisplayName "Nextcloud Docker"
   ```
