# Capacitación en despliegue y administración de Nextcloud

Documentación y guías prácticas para llevar a cabo las prácticas de la capacitación de Nextcloud. 

Dictada en: Laboratorio de Informática Aplicada - Departamento de Informática - Facultad de Ciencias Exactas, Físicas y Naturales - Universidad Nacional de San Juan.  

## 1. Resumen 

El presente espacio reúne el material técnico necesario para llevar a cabo las prácticas de implementación, configuración y mantenimiento de una infraestructura de una nube privada y local utilizando Nextcloud mediante contenedores.

El objetivo de esta capacitación es dotar a los asistentes de los conocimientos fundamentales para desplegar un entorno de almacenamiento y colaboración autoalojado, garantizando la soberanía de los datos, la disponibilidad del servicio y la correcta administración del ciclo de vida de la aplicación en entornos tanto de desarrollo como de producción.

## 2. Tutoriales de instalación

En esta sección se encuentran los procedimientos paso a paso para la preparación del entorno, el despliegue inicial de los contenedores y el restablecimiento del sistema en diferentes sistemas operativos.

Puede acceder al Directorio de Instalación completo o dirigirse directamente a la guía correspondiente a su entorno de trabajo:

Entorno Linux: Consulte la Guía de instalación en Ubuntu para el despliegue nativo mediante Docker en distribuciones basadas en Debian/Ubuntu.

Entorno Windows: Consulte la Guía de instalación en Windows para la configuración del entorno mediante WSL2 y Docker Desktop.

Restablecimiento del entorno: En caso de requerir eliminar los contenedores, volúmenes y directorios creados durante las prácticas para iniciar desde cero, siga el procedimiento de Limpieza total del entorno.

## 3. Suite ofimática: Nextcloud Office y Collabora Online

Para extender las capacidades de almacenamiento hacia un entorno de trabajo colaborativo en tiempo real, se aborda la integración de herramientas de edición de documentos.

En la Guía de instalación de Nextcloud Office y Collabora Online Built-in CODE Server se detallan los requerimientos técnicos, la activación de las aplicaciones correspondientes dentro del ecosistema de Nextcloud y los parámetros de configuración necesarios para poner en funcionamiento el servidor de desarrollo integrado (Built-in CODE Server).

## 4. Mantenimiento: Copias de seguridad y actualizaciones

La continuidad operativa de cualquier servicio en producción depende de una estrategia sólida de resguardo de información y de la aplicación periódica de parches de seguridad.

Acceda a la Guía de mantenimiento, copias de seguridad y actualizaciones para consultar los protocolos estandarizados sobre:

- Activación del modo de mantenimiento en la instancia.

- Resguardo y restauración de la base de datos y directorios de datos de usuario.

- Procedimientos seguros de actualización de imágenes de contenedores y migración de versiones de Nextcloud.

- Estructura sugerida del repositorio

Para que todos los enlaces de este documento funcionen correctamente, se organizan los archivos y directorios del repositorio bajo el siguiente esquema:

``
Capacitacion-Nextcloud/
├── README.md
├── instalacion/
│   ├── tutorial-instalacion-ubuntu.md
│   ├── tutorial-instalacion-windows.md
│   └── limpieza-total.md
├── office-collabora/
│   └── guia-instalacion-office.md
└── mantenimiento/
    └── guia-mantenimiento.md
```


Laboratorio de Informática Aplicada

Material de uso académico.
