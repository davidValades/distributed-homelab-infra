# 🚀 Hybrid Distributed Homelab: Storage Pool & Media Automation

![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white) ![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black) ![Raspberry Pi](https://img.shields.io/badge/-Raspberry_Pi-C51A4A?style=for-the-badge&logo=Raspberry-Pi&logoColor=white) ![Samba](https://img.shields.io/badge/Samba-FECA23?style=for-the-badge&logo=Samba&logoColor=black)

Este repositorio documenta la arquitectura técnica de mi infraestructura doméstica híbrida. El proyecto combina una **Raspberry Pi 5** (como orquestador de descargas y pool de datos) y un **NUCBox M7 Ultra** (como servidor de media de alto rendimiento), creando un sistema de almacenamiento unificado y automatizado.

---

## 🏗️ Arquitectura del Sistema

El sistema se basa en un modelo de **almacenamiento distribuido** y **separación de cargas de trabajo** (Workload Separation) para maximizar la eficiencia del hardware disponible.

### 💾 Gestión de Almacenamiento: MergerFS (The "Infinite Pool")

Utilizo `fuse.mergerfs` para unificar múltiples unidades físicas y remotas en un único punto de montaje lógico (`/mnt/storage`). Esto permite gestionar el almacenamiento como una sola entidad, independientemente de dónde residan los datos físicamente.

**Unidades que componen el Pool:**

- **Local HDD (2TB EXT4):** Almacenamiento principal en Raspberry Pi.
- **Remote NUCBox Partition (800GB CIFS/Samba):** Almacenamiento expandido a través de la red local.
- **Local SSD (OS):** Espacio reservado para configuraciones y datos de acceso rápido.
- **Remote HDD (500GB CIFS/Samba):** Unidad secundaria de respaldo.

**Estrategia de escritura:** `category.create=mfs` (Most Free Space), lo que optimiza el uso del espacio disponible entre todos los discos.

---

### 🌐 Esquema de Red e Infraestructura

### 📂 Estructura de Datos Unificada

Para que los contenedores operen sin conflictos (Atomic Moves), se mantiene una estructura coherente en el pool:

````text
/mnt/storage/data/
├── downloads/      # Descargas completadas
├── media/          # Biblioteca final organizada (Movies, TV, Music)
├── torrents/       # Archivos .torrent activos
└── nextcloud/      # Almacenamiento de nube personal

---

## 🐳 Stack de Servicios (Dockerized)

La infraestructura se divide según la potencia de procesamiento requerida:

### 📍 Nodo A: Raspberry Pi 5 (Gestión y Descargas)

Responsable de la lógica de automatización y el tráfico de red constante.

- **Prowlarr:** Indexador de trackers.
- **Radarr/Sonarr/Lidarr:** Gestión automatizada de películas, series y música.
- **qBittorrent:** Cliente de descargas configurado en `network_mode: host` para evitar cuellos de botella en la red.

### 📍 Nodo B: NUCBox M7 Ultra (Streaming y Transcoding)

Debido a su potente CPU/GPU, este nodo se encarga de las tareas pesadas:

- **Jellyfin:** Servidor de medios con **Hardware Acceleration** para streaming 4K remoto.
- **Jellyseerr:** Interfaz web de usuario para peticiones de contenido.

---

## 🛠️ Configuración de bajo nivel

### Integración en `/etc/fstab`

La persistencia de los montajes remotos y el pool de MergerFS se gestiona mediante directivas de `systemd` para asegurar que el sistema no falle si hay microcortes de red.

```bash
# Ejemplo de montaje remoto con automount
//192.168.1.58/NucBox_Media /mnt/nucbox_remote cifs credentials=/path/.smbcredentials,x-systemd.automount 0 0

# Pool unificado con MergerFS
/mnt/disk1:/mnt/disk_os:/mnt/nucbox_remote:/mnt/hdd_500gb /mnt/storage fuse.mergerfs defaults,allow_other,use_ino,category.create=mfs 0 0
````

Estructura de Datos Unificada
Para que los contenedores operen sin conflictos (Atomic Moves), se mantiene una estructura coherente en el pool:

```bash
/mnt/storage/data/
├── downloads/      # Descargas completadas
├── media/          # Biblioteca final organizada (Movies, TV, Music)
├── torrents/       # Archivos .torrent activos
└── nextcloud/      # Almacenamiento de nube personal
```

## 🛡️ Seguridad y Buenas Prácticas

Principio de Menor Privilegio: Todos los contenedores se ejecutan bajo un usuario no-root mediante variables de entorno PUID y PGID.

Gestión de Credenciales: Uso de archivos .smbcredentials con permisos 600 para evitar contraseñas en texto plano en archivos de configuración global.

Red Aislada: Los servicios se comunican internamente a través de redes Docker personalizadas.

---

_⭐ Proyecto implementado por [David Valadés Navarro](https://github.com/davidValades)._
