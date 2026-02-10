# JasperReports Server (JRS) - Docker Compose Stack

Stack completo de Docker Compose para ejecutar JasperReports Server con MariaDB, incluyendo una base de datos de ejemplo Sakila para pruebas y desarrollo.

## 📋 Descripción

Este proyecto proporciona una configuración lista para usar de **JasperReports Server** utilizando Docker Compose. Incluye:

- **JasperReports Server**: Servidor de reportes empresariales basado en Bitnami
- **MariaDB para JasperReports**: Base de datos principal donde se almacenan los reportes, usuarios y configuraciones
- **MariaDB Sakila**: Base de datos de ejemplo (Sakila) para pruebas y desarrollo, inicializada automáticamente

## 🏗️ Arquitectura

El stack está compuesto por tres servicios principales:

### 1. `jasperdb` (MariaDB para JasperReports)
- **Imagen**: `mariadb:latest`
- **Propósito**: Base de datos principal de JasperReports Server
- **Volumen persistente**: `jasperdb_data`
- **Configuración**: Usuario y base de datos configurables mediante variables de entorno

### 2. `jrs` (JasperReports Server)
- **Imagen**: `gach24/jasper-report-server`
- **Puertos expuestos**:
  - `8080`: HTTP
  - `8443`: HTTPS
- **Volumen persistente**: `jrs_data`
- **Dependencias**: Requiere que `jasperdb` esté disponible
- **Configuración**: Credenciales de administrador configurables

### 3. `sakiladb` (MariaDB Sakila)
- **Imagen**: `mariadb:latest`
- **Propósito**: Base de datos de ejemplo para pruebas
- **Puerto expuesto**: `3306` (accesible desde el host)
- **Volumen persistente**: `sakiladb_data`
- **Inicialización**: Scripts SQL en `./sakila-maria/` se ejecutan automáticamente al crear el contenedor

## 📦 Requisitos Previos

- Docker Engine 20.10+
- Docker Compose v2.0+
- Al menos 2GB de RAM disponible
- Puertos 8080, 8443 y 3306 libres

## 🚀 Instalación y Configuración

### 1. Clonar o descargar el proyecto

```bash
git clone <repository-url>
cd jrs
```

### 2. Configurar variables de entorno

Copia el archivo `.env` y ajusta las variables según tus necesidades:

```bash
cp .env.example .env  # Si existe un ejemplo
# O edita directamente .env
```

**Variables disponibles en `.env`:**

```env
# Configuración de MariaDB
MARIADB_USER=jasperreports          # Usuario de la base de datos
MARIADB_PASSWORD=jasperreports      # Contraseña del usuario
MARIADB_DATABASE=jasperreportsdb    # Nombre de la base de datos
ALLOW_EMPTY_PASSWORD=yes            # Permitir contraseña vacía para root

# Configuración de JasperReports Server
MARIADB_PORT_NUMBER=3306            # Puerto de MariaDB
JASPERREPORTS_USERNAME=admin        # Usuario administrador de JRS
JASPERREPORTS_PASSWORD=admin        # Contraseña del administrador
```

### 3. Iniciar los servicios

```bash
docker compose up -d
```

Esto iniciará todos los contenedores en segundo plano. La primera vez puede tardar varios minutos mientras descarga las imágenes e inicializa las bases de datos.

### 4. Verificar el estado

```bash
docker compose ps
```

### 5. Ver los logs

```bash
# Todos los servicios
docker compose logs -f

# Servicio específico
docker compose logs -f jrs
docker compose logs -f jasperdb
docker compose logs -f sakiladb
```

## 🌐 Acceso a los Servicios

### JasperReports Server

- **URL HTTP**: http://localhost:8080
- **URL HTTPS**: https://localhost:8443
- **Credenciales por defecto**:
  - Usuario: `admin` (configurable en `.env`)
  - Contraseña: `admin` (configurable en `.env`)

### MariaDB Sakila

- **Host**: `localhost`
- **Puerto**: `3306`
- **Base de datos**: `sakila`

**Ejemplo de conexión desde MySQL client:**

```bash
mysql -h localhost -P 3306 -u root -p sakila
```

## 📝 Uso Común

### Iniciar servicios

```bash
docker compose up -d
```

### Detener servicios

```bash
docker compose down
```

### Reiniciar un servicio específico

```bash
docker compose restart jrs
```

### Actualizar imágenes

```bash
docker compose pull
docker compose up -d
```

### Limpiar todo (contenedores, imágenes y volúmenes)

```bash
./clean.sh
```

⚠️ **Advertencia**: El script `clean.sh` elimina todos los contenedores, imágenes y volúmenes de Docker. Esto borrará todos los datos almacenados en las bases de datos.

## 🔧 Scripts Disponibles

### `clean.sh`

Script de limpieza completa que:
1. Detiene todos los contenedores (`docker compose down`)
2. Elimina todas las imágenes no utilizadas (`docker system prune -a -f`)
3. Elimina todos los volúmenes no utilizados (`docker volume prune -a -f`)

**Uso:**
```bash
chmod +x clean.sh
./clean.sh
```

## 💾 Persistencia de Datos

Los datos se almacenan en volúmenes de Docker:

- `jasperdb_data`: Datos de la base de datos de JasperReports
- `jrs_data`: Configuración y datos de JasperReports Server
- `sakiladb_data`: Datos de la base de datos Sakila

Estos volúmenes persisten incluso después de eliminar los contenedores. Para eliminarlos completamente, usa `docker volume rm` o el script `clean.sh`.

## 🔍 Troubleshooting

### Error: "Database is uninitialized and password option is not specified"

**Causa**: MariaDB requiere una contraseña para el usuario root o la variable `MARIADB_ALLOW_EMPTY_ROOT_PASSWORD=yes`.

**Solución**: Asegúrate de que `MARIADB_ALLOW_EMPTY_ROOT_PASSWORD=yes` esté configurado en `docker-compose.yml` para los servicios de MariaDB.

### Error: "Access denied for user 'jasperreports'"

**Causa**: El usuario de la base de datos no tiene contraseña configurada o la contraseña no coincide.

**Solución**: 
1. Verifica que `MARIADB_PASSWORD` esté configurado en `.env`
2. Asegúrate de que `JASPERREPORTS_DATABASE_PASSWORD` en `docker-compose.yml` use la misma variable
3. Si el volumen ya existe con configuración incorrecta, elimínalo: `docker volume rm jrs_jasperdb_data`

### Los servicios no se conectan entre sí

**Causa**: Los contenedores pueden estar iniciándose en orden incorrecto.

**Solución**: El servicio `jrs` tiene `depends_on: jasperdb`, pero si aún así hay problemas, espera unos segundos después de iniciar y verifica los logs.

### Puerto ya en uso

**Causa**: Otro servicio está usando los puertos 8080, 8443 o 3306.

**Solución**: 
- Detén el servicio que usa el puerto
- O modifica los puertos en `docker-compose.yml` (formato: `PUERTO_HOST:PUERTO_CONTENEDOR`)

## 📚 Recursos Adicionales

- [Documentación de JasperReports Server](https://community.jaspersoft.com/documentation)
- [Documentación de MariaDB](https://mariadb.com/docs/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)

