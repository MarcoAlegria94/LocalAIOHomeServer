# Proyecto: Implementación de Nextcloud AIO sobre Ubuntu Server con RAID

**Fecha de generación:** 2026-02-19 15:57:03

---

## 1️⃣ Configuración inicial del servidor (Ubuntu Server + RAID)

### ✅ Logros alcanzados
- Instalación correcta de Ubuntu Server.
- Montaje exitoso del RAID en `/srv/data`.
- Confirmación mediante `df -h` y `mount | grep /srv/data`.
- Verificación de escritura real sobre `/dev/md0`.

### 🚧 Roadblocks encontrados
- Dudas sobre si Docker escribía fuera del RAID.
- Confusión por múltiples entradas `overlay` en `df -h`.

### 💡 Cómo evitar estos problemas
- Validar siempre con `df -h | grep /srv/data`.
- Confirmar que la primera línea muestre el RAID real.
- Recordar que `overlay` pertenece a Docker.

---

## 2️⃣ Configuración de Docker y estructura correcta

### ✅ Logros alcanzados
- Estructura organizada:
```
/srv/data/docker/
├── nextcloud-aio/
└── nextcloud-data/
```
- Separación correcta entre configuración y datos.
- Uso correcto de `docker compose`.

### 🚧 Roadblocks encontrados
- Mezcla inicial entre config y data.
- Errores de permisos.
- Uso innecesario de `sudo` con Docker.

### 💡 Cómo evitar estos problemas
- No mezclar configuración con datos.
- Agregar usuario al grupo docker.
- Evitar ejecutar Docker como root.

---

## 3️⃣ Permisos y ACL

### ✅ Logros alcanzados
- Uso correcto de ACL con `setfacl`.
- Mantener `www-data` como owner.
- Acceso controlado para usuario humano.

### 🚧 Roadblocks encontrados
- Intentos de cambiar owner con `chown`.
- Bloqueos por permisos tipo `drwxr-x--- www-data root`.

### 💡 Cómo evitar estos problemas
- Nunca cambiar owner en Nextcloud AIO.
- Usar ACL en lugar de `chmod 777`.
- Verificar con `getfacl`.

---

## 4️⃣ Loop de instalación en Nextcloud AIO

### ✅ Logros alcanzados
- Identificación del error de instalación inicial.
- Confirmación de Redis y PostgreSQL activos.
- Corrección de estructura de carpetas.

### 🚧 Roadblocks encontrados
- Uso incorrecto de un único directorio para config y data.
- Permisos inconsistentes.
- Docker iniciado antes del RAID.

### 💡 Cómo evitar estos problemas
- Separar siempre config y data.
- Confirmar montaje del RAID antes de Docker.
- Validar permisos antes de levantar servicios.

---

# 🏁 Resumen Final

Se logró:
- Implementar Ubuntu Server sobre hardware reciclado.
- Configurar RAID correctamente.
- Instalar Docker con estructura profesional.
- Implementar Nextcloud AIO correctamente.
- Configurar permisos seguros con ACL.
- Corregir loops de instalación.
- Dejar infraestructura lista para producción.

---

# 🚀 Método Más Eficiente (Checklist)

1. Instalar Ubuntu Server.
2. Configurar RAID y validar con `df -h`.
3. Configurar `/etc/fstab`.
4. Instalar Docker.
5. Crear estructura separada para config y data.
6. Configurar permisos base.
7. Aplicar ACL si es necesario.
8. Guardar `docker-compose.yml` en carpeta dedicada.
9. Ejecutar `docker compose up -d`.
10. Validar logs.

---

# 🎯 Resultado

Infraestructura:
- Segura
- Escalable
- Organizada
- Lista para crecimiento futuro

Proyecto completado con éxito.
