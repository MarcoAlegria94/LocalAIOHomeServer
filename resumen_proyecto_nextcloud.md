# 📋 Resumen del Proyecto: Servidor Personal con Nextcloud AIO en Docker

**Sistema:** Ubuntu Server 24 con Docker  
**Versión Nextcloud:** All-In-One (AIO) - Latest  
**Dominio:** marckdevai.org  
**Fecha:** Febrero 2026

---

## 🗂️ Índice de Conversaciones

| Chat | Tema Principal |
|------|----------------|
| Chat 1 | Planificación inicial: Docker Compose para Nextcloud, Jellyfin y N8N |
| Chat 2 | Instalación de Nextcloud AIO y recuperación de datos de usuarios |
| Chat 3 | Migración de archivos y sincronización con occ files:scan |
| Chat 4 | Recuperación post-falla: corrección de permisos en volúmenes Docker |

---

## ✅ Chat 1 — Planificación del Servidor Personal (Nextcloud + Jellyfin + N8N)

### Objetivo
Crear un `docker-compose.yml` completo para montar Nextcloud, Jellyfin + Jellyseerr (con automatización de descargas), N8N, Cloudflared (túneles) y una web de negocio personal.

### Decisiones clave tomadas
- Volúmenes montados bajo `/srv/data/docker/...`
- Red compartida (`services_network`) para comunicación entre contenedores
- Dominio base: `marckdevai.org`
- 16 GB de RAM disponible → suficiente para todos los servicios
- Se acordó usar **Nextcloud All-In-One** en lugar de la imagen estándar

### Resultado exitoso
Se generó un `docker-compose.yml` funcional con límites de memoria por servicio y un `setup.sh` con alias de administración.

### ⚠️ Roadblocks y cómo evitarlos

> **Problema:** Se generó inicialmente un compose general; el usuario necesitaba AIO específicamente.  
> **Consejo:** Al solicitar un compose con Nextcloud, especifica desde el inicio si deseas usar `nextcloud/all-in-one` o la imagen estándar. Son arquitecturas completamente diferentes.

> **Problema:** Se olvidó incluir el túnel de Cloudflare para la web de negocio en el primer borrador.  
> **Consejo:** Lista todos los túneles de Cloudflare que necesitas antes de pedir el compose. Incluye el propósito de cada token para no omitir ninguno.

---

## ✅ Chat 2 — Instalación de Nextcloud AIO y Recuperación de Datos

### Objetivo
Levantar Nextcloud AIO reutilizando datos de usuarios preexistentes (~220 GB) almacenados en `/srv/data/docker/volumes/nextcloud_aio_nextcloud_data/_data/`.

### Situación encontrada
- Había una instalación antigua en `/srv/data/docker/nextcloud/` (sin datos relevantes)
- Los datos reales de usuarios (220 GB) estaban en los volúmenes de Docker AIO: `nextcloud_aio_nextcloud_data`
- Usuarios encontrados: `admin`, `Carmen Guadarrama`, `MarckDevAI`, `Marco Alegria`
- Ningún contenedor de Nextcloud estaba corriendo

### Resultado exitoso
Se levantó Nextcloud AIO con el `docker-compose` correcto apuntando a los volúmenes existentes, preservando los 220 GB de datos sin pérdida.

### Docker Compose exitoso para Nextcloud AIO
```yaml
version: '3.8'
services:
  nextcloud-aio-mastercontainer:
    image: nextcloud/all-in-one:latest
    container_name: nextcloud-aio-mastercontainer
    restart: always
    ports:
      - "192.168.1.100:8080:8080"
    volumes:
      - nextcloud_aio_mastercontainer:/mnt/docker-aio-config
      - /var/run/docker.sock:/var/run/docker.sock:ro
    environment:
      - NEXTCLOUD_DATADIR=/srv/data/docker/volumes/nextcloud_aio_nextcloud_data

volumes:
  nextcloud_aio_mastercontainer:
    external: true
```

### ⚠️ Roadblocks y cómo evitarlos

> **Problema:** Se confundió la instalación antigua en `/srv/data/docker/nextcloud/` con la instalación activa de AIO.  
> **Consejo:** Antes de cualquier migración, ejecuta `docker ps -a | grep nextcloud` y `sudo du -sh /srv/data/docker/volumes/nextcloud_aio_*` para identificar qué instalación tiene los datos reales.

> **Problema:** Los datos de usuario (72KB en la carpeta vieja vs 220GB en volumes) generaron confusión inicial.  
> **Consejo:** Verifica siempre el tamaño de los directorios con `du -sh` antes de asumir dónde están los datos.

> **Problema:** Se requirió ajustar los permisos antes de levantar el contenedor.  
> **Consejo:** Antes de iniciar el mastercontainer, siempre ejecuta:
> ```bash
> sudo chown -R www-data:www-data /srv/data/docker/volumes/nextcloud_aio_nextcloud_data/_data
> sudo chmod -R 750 /srv/data/docker/volumes/nextcloud_aio_nextcloud_data/_data
> ```

---

## ✅ Chat 3 — Migración de Archivos y Sincronización con occ

### Objetivo
Copiar archivos del directorio `/srv/data/brk/MarckRock` al volumen de Nextcloud y hacerlos visibles en la app de Android.

### Proceso exitoso

#### Paso 1: Copiar archivos sin crear subcarpeta extra
```bash
rsync -av /srv/data/brk/MarckRock/ \
  /srv/data/docker/volumes/nextcloud_aio_nextcloud_data/_data/Marco/files/MarckRock/
```

#### Paso 2: Ajustar permisos
```bash
sudo chown -R 33:33 \
  /srv/data/docker/volumes/nextcloud_aio_nextcloud_data/_data/Marco/files/MarckRock
```

#### Paso 3: Escanear archivos en Nextcloud
```bash
docker exec -u www-data nextcloud-aio-nextcloud php occ files:scan Marco
```

#### Paso 4 (opcional): Generar metadatos
```bash
docker exec -u www-data nextcloud-aio-nextcloud php occ files:scan --generate-metadata Marco
```

### Problema adicional: App Android crasheaba al abrir carpeta de imágenes
**Causa:** `OutOfMemoryError` en el cliente Android al intentar cargar previsualizaciones de muchas imágenes simultáneamente.

**Solución aplicada:**
1. Borrar caché de la app: `Ajustes → Apps → Nextcloud → Almacenamiento → Borrar caché`
2. Desactivar "Precarga de miniaturas" en la app
3. Pre-generar previsualizaciones desde el servidor:
   ```bash
   docker exec -u www-data nextcloud-aio-nextcloud php occ preview:repair
   ```

### ⚠️ Roadblocks y cómo evitarlos

> **Problema:** El comando `preview:generate-all` no existe en versiones recientes de Nextcloud AIO.  
> **Consejo:** Antes de usar comandos de `occ`, verifica los disponibles con:
> ```bash
> docker exec -u www-data nextcloud-aio-nextcloud php occ list preview
> ```

> **Problema:** Al usar `cp` sin la barra final, se creaba una subcarpeta extra dentro del destino.  
> **Consejo:** Usa siempre `rsync` con barra al final del origen (`/srv/.../origen/`) para copiar solo el *contenido* de la carpeta.

> **Problema:** Nextcloud no detecta archivos copiados manualmente hasta que se ejecuta el scan.  
> **Consejo:** Este es un paso **obligatorio e irremplazable** después de cualquier copia manual de archivos al volumen de Nextcloud.

---

## ✅ Chat 4 — Recuperación Post-Falla: Corrección de Permisos

### Objetivo
Recuperar Nextcloud AIO tras una falla del servidor que dejó los contenedores con permisos incorrectos.

### Error inicial
```
Cannot write to /mnt/data       # aio-apache
FATAL: data directory has wrong ownership  # PostgreSQL
```

### Causa raíz
Efecto dominó por permisos incorrectos en el volumen de PostgreSQL:
1. PostgreSQL no inicia → Nextcloud espera indefinidamente la DB → Apache no puede escribir

### Solución completa aplicada

#### Paso 1: Detener todos los contenedores
```bash
docker stop nextcloud-aio-apache nextcloud-aio-nextcloud \
  nextcloud-aio-database nextcloud-aio-redis nextcloud-aio-database-dump
```

#### Paso 2: Corregir permisos por volumen
```bash
# PostgreSQL (UID 999)
sudo chown -R 999:999 /srv/data/docker/volumes/nextcloud_aio_database/_data
sudo chmod -R 700 /srv/data/docker/volumes/nextcloud_aio_database/_data

sudo chown -R 999:999 /srv/data/docker/volumes/nextcloud_aio_database_dump/_data
sudo chmod -R 755 /srv/data/docker/volumes/nextcloud_aio_database_dump/_data

# Nextcloud data (UID 33 = www-data)
sudo chown -R 33:0 /srv/data/docker/volumes/nextcloud_aio_nextcloud_data/_data
sudo chmod -R 770 /srv/data/docker/volumes/nextcloud_aio_nextcloud_data/_data

# Nextcloud app
sudo chown -R 33:0 /srv/data/docker/volumes/nextcloud_aio_nextcloud/_data
sudo chmod -R 755 /srv/data/docker/volumes/nextcloud_aio_nextcloud/_data

# Apache
sudo chown -R root:root /srv/data/docker/volumes/nextcloud_aio_apache/_data
sudo chmod -R 755 /srv/data/docker/volumes/nextcloud_aio_apache/_data

# Redis (UID 999)
sudo chown -R 999:999 /srv/data/docker/volumes/nextcloud_aio_redis/_data
sudo chmod -R 755 /srv/data/docker/volumes/nextcloud_aio_redis/_data
```

#### Paso 3: Configurar Redis (vm.overcommit_memory)
```bash
sudo sysctl vm.overcommit_memory=1
echo "vm.overcommit_memory=1" | sudo tee -a /etc/sysctl.conf
```

#### Paso 4: Reiniciar desde el mastercontainer
```bash
docker restart nextcloud-aio-mastercontainer
# Luego acceder a http://IP:8080 → "Start containers"
```

#### Paso 5: Escanear archivos post-recuperación
```bash
docker exec -u www-data nextcloud-aio-nextcloud php occ files:scan --all
docker exec -u www-data nextcloud-aio-nextcloud php occ files:scan --all --generate-metadata
```

### Tabla de referencia de permisos correctos

| Volumen | UID:GID | chmod | Servicio |
|---------|---------|-------|----------|
| `nextcloud_aio_database` | 999:999 | 700 | PostgreSQL |
| `nextcloud_aio_database_dump` | 999:999 | 755 | Backups DB |
| `nextcloud_aio_nextcloud_data` | 33:0 | 770 | Datos de usuarios |
| `nextcloud_aio_nextcloud` | 33:0 | 755 | App Nextcloud |
| `nextcloud_aio_apache` | root:root | 755 | Servidor web |
| `nextcloud_aio_redis` | 999:999 | 755 | Cache |

### ⚠️ Roadblocks y cómo evitarlos

> **Problema:** Errores EXIF durante el escaneo (`exif_read_data(): Illegal format code`).  
> **Consejo:** Estos errores son **no críticos**. Los archivos se escanean correctamente; solo falla la lectura de metadatos de fotos editadas o con EXIF corrupto. Solo preocúpate si ves `Permission denied` o `Database error`.

> **Problema:** Usar `www-data` como nombre de usuario en `chown` en el host fallaba.  
> **Consejo:** Siempre usa el **UID numérico** (33 para www-data, 999 para postgres) al manipular permisos de volúmenes Docker desde el host.

> **Problema:** Los permisos del directorio padre (`/srv`, `/srv/data`, etc.) bloqueaban el acceso.  
> **Consejo:** Verifica toda la cadena de directorios con:
> ```bash
> ls -ld /srv /srv/data /srv/data/docker /srv/data/docker/volumes
> ```
> Todos deben tener al menos `755`.

---

## 🚀 Resumen Final del Proyecto

### ¿Qué se logró?
Se configuró exitosamente un servidor personal Ubuntu con **Nextcloud AIO en Docker** que:
- Preservó **220 GB de datos de 4 usuarios** sin pérdida
- Sobrevivió y se recuperó de una falla de servidor
- Permite acceso remoto vía túnel Cloudflare (`cloud.marckdevai.org`)
- Sincroniza archivos con la app de Android

---

## ⚡ Guía de Instalación Eficiente (Orden Óptimo)

Si tuvieras que hacer esto de nuevo desde cero, este es el orden más eficiente:

### 1. Preparar directorios y permisos base
```bash
sudo mkdir -p /srv/data/docker/volumes
sudo chmod 755 /srv /srv/data /srv/data/docker /srv/data/docker/volumes
```

### 2. Levantar Nextcloud AIO
```bash
docker run -d \
  --name nextcloud-aio-mastercontainer \
  --restart always \
  -p 8080:8080 \
  -v nextcloud_aio_mastercontainer:/mnt/docker-aio-config \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  -e NEXTCLOUD_DATADIR=/srv/data/docker/volumes/nextcloud_aio_nextcloud_data \
  nextcloud/all-in-one:latest
```

### 3. Configurar desde la interfaz web
Acceder a `http://IP:8080` y seguir el wizard de AIO.

### 4. Migrar archivos existentes (si aplica)
```bash
# Copiar archivos
rsync -av /ruta/origen/ /srv/data/docker/volumes/nextcloud_aio_nextcloud_data/_data/USUARIO/files/CARPETA/

# Ajustar permisos
sudo chown -R 33:0 /srv/data/docker/volumes/nextcloud_aio_nextcloud_data/_data
sudo chmod -R 770 /srv/data/docker/volumes/nextcloud_aio_nextcloud_data/_data

# Escanear en Nextcloud
docker exec -u www-data nextcloud-aio-nextcloud php occ files:scan --all
```

### 5. Configurar Redis (hacer permanente)
```bash
echo "vm.overcommit_memory=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

### 6. Verificar estado del sistema
```bash
docker ps | grep nextcloud-aio
docker logs nextcloud-aio-apache --tail 20
docker logs nextcloud-aio-database --tail 20
```

---

## 🛟 Comandos de Emergencia (Referencia Rápida)

```bash
# Ver todos los contenedores de Nextcloud
docker ps -a | grep nextcloud-aio

# Escanear archivos de un usuario
docker exec -u www-data nextcloud-aio-nextcloud php occ files:scan USUARIO

# Modo mantenimiento ON/OFF
docker exec -u www-data nextcloud-aio-nextcloud php occ maintenance:mode --on
docker exec -u www-data nextcloud-aio-nextcloud php occ maintenance:mode --off

# Ver comandos de preview disponibles
docker exec -u www-data nextcloud-aio-nextcloud php occ list preview

# Listar usuarios de Nextcloud
docker exec -u www-data nextcloud-aio-nextcloud php occ user:list

# Ver logs en tiempo real
docker logs -f nextcloud-aio-apache
docker logs -f nextcloud-aio-database
```

---

*Documentación generada a partir de 4 conversaciones de soporte técnico — Febrero 2026*
