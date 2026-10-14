# Práctica: Despliegue de Tres SGBD (Oracle 23ai, PostgreSQL y MariaDB)

## 1. Características del Entorno y Versiones
- **Sistema Operativo:** Debian 12 / Linux Mint
- **Recursos de la Máquina Virtual:**
  - Memoria RAM: 4 GB
  - Procesadores: 2 vCPU
  - Almacenamiento: 250 GB
- **Versiones de los SGBD instalados:**
  - Oracle Database Free: 23ai (23.5.0.0 sobre Podman)
  - PostgreSQL: 15 / 16
  - MariaDB: 10.11 / 11.x

### Verificación de recursos:
![Recursos de la VM](evidencias/01-recursos-vm.png)

---

## 2. Pasos de Instalación y Despliegue

### 2.1 Oracle Database 23ai Free (con Podman)
1. Se instaló Podman en el sistema mediante `apt`.
2. Se descargó la imagen oficial de Oracle Database Free desde el registro de Oracle.
3. Se creó un volumen persistente (`oracle_23ai_datos`) para conservar la información en `/opt/oracle/oradata`.
4. Se desplegó el contenedor publicando el puerto `1521:1521` con política de reinicio automático.
5. Se accedió vía SQL*Plus para cambiar a la PDB `FREEPDB1`, crear el usuario de trabajo `orauser` con privilegios de sesión, creación de tablas y cuota en el tablespace `USERS`.

### 2.2 PostgreSQL
1. Se instalaron los paquetes `postgresql`, `postgresql-contrib` y extensiones requeridas.
2. Se habilitó e inició el servicio con `systemctl`.
3. Desde la sesión del usuario `postgres`, se ejecutó `psql` para crear la base de datos `dbpg`, el usuario `pguser` con su correspondiente contraseña y se le asignaron todos los privilegios sobre dicha base de datos.

### 2.3 MariaDB
1. Se instaló el servidor y cliente de MariaDB mediante `apt`.
2. Se activó y arrancó el servicio del gestor.
3. Se creó la base de datos `dbm` con codificación `utf8mb4` y el usuario local `muser` con permisos totales sobre ella.

---

## 3. Comprobación del Estado y Conexiones Locales

### Descarga de la imagen de Oracle:
![Podman Images](evidencias/02-podman-images.png)

### Estado del contenedor de Oracle:
![Podman PS](evidencias/03-podman-ps.png)

### Conexión local a Oracle (orauser @ FREEPDB1):
![Conexión Oracle](evidencias/04-oracle-orauser.png)

### Conexión local a PostgreSQL (pguser @ dbpg):
![Conexión PostgreSQL](evidencias/05-postgresql-pguser.png)

### Conexión local a MariaDB (muser @ dbm):
![Conexión MariaDB](evidencias/06-mariadb-muser.png)

### Comprobación de puertos abiertos con Netcat:
Se comprobó la escucha en los puertos 1521 (Oracle), 5432 (PostgreSQL) y 3306 (MariaDB) usando `nc -zv 127.0.0.1 <puerto>`:
![Comprobación Netcat](evidencias/07-puertos-nc.png)

---

## 4. Configuración de Conexiones Remotas

### 4.1 PostgreSQL
- Se editó `postgresql.conf` para habilitar `listen_addresses = '*'`.
- Se configuró `pg_hba.conf` añadiendo la regla de autenticación por contraseña `scram-sha-256` para la IP/subred autorizada del cliente.
- Se abrió el puerto TCP 5432 en el cortafuegos `ufw` exclusivamente para la máquina cliente.
- Se verificó la conexión remota desde el equipo externo mediante `psql -h <IP_VM> -U pguser -d dbpg -W`.

![Conexión Remota PostgreSQL](evidencias/08-remoto-postgres.png)

### 4.2 MariaDB
- Se editó `50-server.cnf` asignando `bind-address = 0.0.0.0`.
- Se dio de alta al usuario con permisos remotos: `'muser'@'IP_DEL_CLIENTE'`.
- Se abrió el puerto TCP 3306 en `ufw` restringido a la IP del cliente.
- Se comprobó la conexión desde el cliente con `mariadb -h <IP_VM> -u muser -p dbm`.

![Conexión Remota MariaDB](evidencias/09-remoto-mariadb.png)

---

## 5. Problemas Encontrados y Soluciones
- **Comando de Podman cortado:** Al ejecutar `podman run`, se omitieron inicialmente la ruta de volumen y el nombre de la imagen arrojando el error `requires at least 1 arg(s)`. Se solucionó ejecutando el comando completo en una única línea incluyendo el flag `:Z` para los permisos de volumen.
- **Sintaxis de UFW:** En el cortafuegos se introdujo el texto literal en lugar de la dirección IP numérica, lo que causó el mensaje `ERROR: Dirección de orígen errónea`. Se solucionó averiguando la IP real del cliente con `ip a` e indicándola explícitamente en la regla.
- **Búsqueda de parámetros de red:** En MariaDB se utilizó la herramienta de búsqueda de `nano` (`Ctrl + W`) para ubicar y descomentar con precisión la directiva `bind-address`.

---

## 6. Fuentes Consultadas
- [Oracle Database Free Get Started](https://www.oracle.com/es/database/free/get-started/)
- Documentación oficial de PostgreSQL (Client Authentication - pg_hba.conf)
- Documentación oficial de MariaDB Server (Configuring MariaDB for Remote Client Access)
