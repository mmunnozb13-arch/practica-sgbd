# Práctica: Despliegue de Tres SGBD (Oracle 23ai, PostgreSQL y MariaDB)

## 1. Características del Entorno y Versiones
- **Sistema Operativo:** Linux Mint (Base Ubuntu 24.04 LTS "Noble")
- **Recursos de la Máquina Virtual (VirtualBox):**
  - Memoria RAM: 4096 MB (4 GB)
  - Procesadores: 2 vCPU
  - Almacenamiento: 50,00 GB (Disco virtual `Linuxmint.vdi`)
  - Red inicial: NAT (Adaptador Intel PRO/1000 MT Desktop)
- **Versiones de los SGBD instalados:**
  - Oracle Database Free: 23ai Release 23.0.0.0.0 (Versión 23.5.0.24.07 desplegada con Podman 4.9.3)
  - PostgreSQL: 16.15 (Ubuntu 16.15-0ubuntu0.24.04.1)
  - MariaDB: 10.11.14 (10.11.14-MariaDB-0ubuntu0.24.04.1)

### Verificación de recursos:
![Recursos de la VM](evidencias/01-recursos-vm.png)

---

## 2. Pasos de Instalación y Despliegue

### 2.1 Oracle Database 23ai Free (con Podman)
1. Se instaló Podman en el sistema mediante `sudo apt install -y podman`.
2. Se descargó la imagen oficial de Oracle Database Free: `container-registry.oracle.com/database/free:23.5.0.0`.
3. Se creó un volumen persistente denominado `oracle_23ai_datos` para salvaguardar los datos de `/opt/oracle/oradata`.
4. Se desplegó el contenedor `cont-oracle` publicando el puerto `1521:1521` con política `--restart always` y flag `:Z` en el montaje del volumen.
5. Se monitorizó el arranque con `sudo podman logs -f cont-oracle` hasta confirmar el mensaje `DATABASE IS READY TO USE!`.
6. Se accedió vía SQL*Plus como `sysdba`, se cambió a la PDB `FREEPDB1` y se creó el usuario `orauser` concediéndole permisos de sesión, creación de tablas y 100 MB de cuota en el tablespace `USERS`.

### 2.2 PostgreSQL
1. Se instalaron los paquetes `postgresql`, `postgresql-contrib` y `php-pgsql` mediante `apt`.
2. Se habilitó e inició el servicio `postgresql` comprobando su estado activo con `systemctl`.
3. Desde la sesión del usuario del sistema `postgres`, se ejecutó `psql` para crear la base de datos `dbpg`.
4. Se creó el usuario `pguser` con su contraseña y se le concedieron todos los privilegios sobre la base de datos `dbpg`.

### 2.3 MariaDB
1. Se instaló el servidor y el cliente mediante `sudo apt install -y mariadb-server mariadb-client`.
2. Se habilitó y arrancó el servicio `mariadb` mediante `sudo systemctl enable --now mariadb`.
3. Desde la consola `sudo mariadb`, se creó la base de datos `dbm` con codificación `utf8mb4` y cotejamiento `utf8mb4_unicode_ci`.
4. Se dio de alta al usuario `'muser'@'localhost'` con su respectiva clave y se le otorgaron privilegios completos sobre `dbm.*`.

---

## 3. Comprobación del Estado y Conexiones Locales

### Descarga de la imagen de Oracle:
Comprobación de la imagen descargada en el registro local mediante `podman images`:
![Podman Images](evidencias/02-podman-images.png)

### Estado del contenedor de Oracle:
Arranque completado y contenedor en estado saludable (`Up / healthy`) escuchando en `0.0.0.0:1521->1521/tcp`:
![Podman PS](evidencias/03-podman-ps.png)

### Conexión local a Oracle (orauser @ FREEPDB1):
Conexión interactiva a SQL*Plus autenticándose como `orauser` y consultando la PDB activa (`FREEPDB1`):
![Conexión Oracle](evidencias/04-oracle-orauser.png)

### Conexión local a PostgreSQL (pguser @ dbpg):
Conexión mediante el cliente `psql` a `localhost` sobre la base de datos `dbpg` comprobada con `SELECT current_user, current_database();`:
![Conexión PostgreSQL](evidencias/05-postgresql-pguser.png)

### Conexión local a MariaDB (ment Authentication - pg_hba.conf)
- Documentación oficial de MariaDB Server (Configuring MariaDB for Remote Client Access)
