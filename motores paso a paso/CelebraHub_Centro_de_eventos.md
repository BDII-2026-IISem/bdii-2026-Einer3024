# Instalación y paso a paso de modelos de base de datos

## Página web del docente

Escribo aquí la dirección de la página web del docente:

https://tecnogua.com/academic/site/bd/introduccion/

## Requisitos e instalación

A continuación, se presenta la instalación paso a paso de cada requisito.

- WSL2 instalado y funcionando
- Docker funcionando dentro de WSL
- Acceso a terminal bash en WSL

### Instalar Docker y Docker Compose


### Actualizar paquetes
``` bash
sudo apt update
```
evidencia

![alt text](./evidencia/image.png)


### Instalar dependencias
``` bash 
sudo apt install -y ca-certificates curl
```

evidencia

![alt text](./evidencia/image-1.png)


### Crear directorio para llaves
``` bash
sudo install -m 0755 -d /etc/apt/keyrings
```

evidencia

![alt text](./evidencia/image-2.png)


### Descargar llave GPG de Docker
``` bash 
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
``` 
evidencia

![alt text](./evidencia/image-3.png)


### Dar permisos
``` bash
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

evidencia

![alt text](./evidencia/image-4.png)

 
### Agregar repositorio
``` bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

evidencia

![alt text](./evidencia/image-5.png)


### Actualizar lista de paquetes
``` bash
sudo apt update
```
evidencia

![alt text](./evidencia/image-6.png)

### Instalar Docker
``` bash 
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

evidencia

![alt text](./evidencia/image-7.png)

``` bash 
sudo apt update
sudo apt install -y tree
```

evidencia

![alt text](./evidencia/image-8.png)

Verifica Docker:

``` bash
sudo docker --version
sudo docker compose version
```

evidencia

![alt text](./evidencia/image-9.png)

verifico si tengo instalado lo necesario usando los siguientes comandos en WSL.

```bash
sudo systemctl is-active docker
```
```bash
docker compose version
```
```bash
docker --version
```
Evidencia de que todo está instalado correctamente:

![alt text](./evidencia/image-10.png)


Ejecuta los siguientes comandos para actualizar los paquetes y evitar errores de dependencias:

``` bash
sudo apt update
sudo apt upgrade -y
```
``` bash
sudo apt install --only-upgrade python3-software-properties software-properties-common
```
``` bash
sudo apt full-upgrade -y
```

# Instalo los requisitos para Docker

```bash
sudo apt-get install ca-certificates curl
```

## Agrego la llave oficial de Docker

### Add Docker's official GPG key:
```bash
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```
evidencia

![alt text](./evidencia/image-11.png)

### Agrego el repositorio de Docker

#### Add the repository to Apt sources:
```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

evidencia

![alt text](./evidencia/image-12.png)


## Instalo Docker Compose plugin y compruebo la instalación

```bash
sudo systemctl stop unattended-upgrades
sudo apt install docker-compose-plugin
sudo docker --version
sudo docker compose version
```
evidencia 

![alt text](./evidencia/image-13.png)

# 1. MySQL: instalación inicial

## 1.1 Crear las carpetas de trabajo

Ejecuta los siguientes comandos para crear las carpetas:

``` bash
mkdir -p ~/ia-lab/services/motores-bd/{mysql,postgres,mssql,oracle}
mkdir -p ~/ia-lab/data/{mysql,postgres,mssql,oracle}
```

Verifica la estructura:

``` bash
cd ia-lab
tree
```

Debería verse así:

```         
~/ia-lab/
├── services/
│   └── motores-bd/
│       ├── mysql/
│       ├── postgres/
│       ├── mssql/
│       └── oracle/
└── data/
    ├── mysql/
    ├── postgres/
    ├── mssql/
    └── oracle/
```

evidencia

![alt text](./evidencia/image-14.png)

**Registro:**

Después de instalar los requisitos y Docker Compose, creé la estructura inicial del proyecto. Como se muestra en el diagrama, se crearon correctamente las carpetas `data` y `services`, junto con las carpetas de MySQL, PostgreSQL y SQL Server.

## 1.2 Crear la red Docker compartida

Creo una red Docker compartida para que todos los contenedores puedan comunicarse entre sí. Ejecuto este comando en mi terminal WSL:

```bash
docker network inspect ia-lab-network >/dev/null 2>&1 || docker network create ia-lab-network
```

Si aparece un error, ejecuta los siguientes comandos:

```bash
exit 
```
se entra a poweshell y se escribe esto 
```bash
wsl --shutdown
```
se entra a ubunto y se ejecuta esto 

```bash
groups
```

```bash
docker ps
```
y ya se soluciona 

evidencia 

![alt text](./evidencia/image-32.png)

Verifico que la red se haya creado correctamente:

```bash
docker network ls | grep ia-lab
```

Evidencia

![alt text](./evidencia/image-15.png)



Ejecuté los comandos en WSL. El resultado fue el esperado: se creó la red `ia-lab-network`, se mostró su identificador `6ebf13e6d30a` y apareció como una red `bridge` local al ejecutar la verificación.


# Instalación de Base de Datos

## paso 1.3 Crear el archivo `docker-compose.yml`

Creo el archivo de configuración de MySQL dentro de la carpeta correspondiente usando el siguiente comando:

```bash
cat > ~/ia-lab/services/motores-bd/mysql/docker-compose.yml << 'EOF'
services:
  mysql:
    image: mysql:8.0
    container_name: mysql-server
    restart: unless-stopped
    env_file:
      - .env
    ports:
      - "3306:3306"
    volumes:
      - ../../../data/mysql:/var/lib/mysql
      - /mnt/d/academia/bd:/backups
    command: >
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_unicode_ci
      --bind-address=0.0.0.0
    networks:
      - ia-lab-network
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s

networks:
  ia-lab-network:
    external: true
EOF
```

Intento habilitar UFW y permito el puerto `3306` de MySQL en el host:

```bash
sudo ufw allow 3306/tcp
sudo ufw enable
sudo ufw status
```

evidencia de que todo funciono bien 


![alt text](./evidencia/image-16.png)


## en caso de un error 

si ufw no está instalado. Instálalo primero:

``` bash
sudo apt install ufw -y
```
Luego ejecuta los comandos por separado:

```bash 
sudo ufw allow 3306/tcp
```

```bash 
sudo ufw enable
```

```bash 
sudo ufw status
```

### 1.4 Crear el archivo `.env`

Creo el archivo `.env` para configurar la zona horaria, la contraseña del usuario administrador y el nombre de la base de datos de MySQL:

```bash
cat > ~/ia-lab/services/motores-bd/mysql/.env << 'EOF'
TZ=America/Bogota
MYSQL_ROOT_PASSWORD=einer123
MYSQL_DATABASE=CelebraHub_Centro_de_eventos
EOF



```

Evidencia


![alt text](./evidencia/image-17.png)

Creé correctamente el archivo `.env` con la zona horaria `America/Bogota`, la contraseña `einer123` y la base de datos `CelebraHub_Centro_de_eventos`. La evidencia confirma su creación. Esta es la base de datos del proyecto final del semestre.


### 1.5 Crear `README.md`

Creo el archivo `README.md` en la carpeta de MySQL:

```bash
touch ~/ia-lab/services/motores-bd/mysql/README.md
```

Después abro el archivo para agregar su contenido:

```bash
sudo nano ~/ia-lab/services/motores-bd/mysql/README.md
```

Dentro del archivo escribo lo siguiente:

````markdown
# MySQL 8.0 - Motor de Base de Datos

> **Acceso remoto habilitado.** Puerto expuesto en `0.0.0.0:3306`.
> **Usuario por defecto:** `root`
> **Base de datos inicial:** `CelebraHub_Centro_de_eventos`
> **Password:** `einer123`

---

## Conectar desde WSL (local)

```bash
sudo docker exec -it mysql-server mysql -u root -p
# Password: einer123
```
````

Después de guardar el contenido, muestro el archivo para comprobar que fue creado correctamente:

```bash
cat ~/ia-lab/services/motores-bd/mysql/README.md
```

Evidencia 

![alt text](./evidencia/image-18.png)

### 1.6 Levantar MySQL

Ingreso a la carpeta de MySQL y levanto el contenedor en segundo plano:

```bash
cd ~/ia-lab/services/motores-bd/mysql
sudo docker compose up -d
```

Verifico que el contenedor esté corriendo:

```bash
sudo docker ps | grep mysql-server
```

Después reviso los últimos 20 registros del contenedor:

```bash
sudo docker logs mysql-server --tail 20
```

evidencia 

![alt text](./evidencia/image-19.png)

### 1.7 Conectar localmente a MySQL

A diferencia de la guía del docente, donde se crea una base de datos de prueba (`bd_clase1`), aquí pasamos directo a la base de datos oficial del proyecto final: `CelebraHub_Centro_de_eventos`. No se crea un nuevo esquema porque la conexión ya está declarada en el .`env` mediante `MYSQL_DATABASE=CelebraHub_Centro_de_eventos`, así que simplemente la seleccionamos para trabajar.

La estructura de mi proyecto final es **CelebraHub - Centro de eventos**. La referencia del proyecto es:

![alt text](./evidencia/image-20.png)

Me conecto localmente al motor de MySQL con el siguiente comando:

```bash
sudo docker exec -it mysql-server mysql -u root -p
```
evidencia 

![alt text](./evidencia/image-21.png)

Listo todas las bases de datos:

```sql
SHOW DATABASES;
```

Selecciono la base de datos `CelebraHub_Centro_de_eventos`, que corresponde al proyecto final del semestre:

```sql
USE CelebraHub_Centro_de_eventos;
```

Como la base de datos todavía no tiene las tablas del proyecto, las creo dentro de `CelebraHub_Centro_de_eventos`:

```sql
-- =====================================================================
-- 1. MODELO RBAC (features/auth)
-- =====================================================================
CREATE TABLE users (
  id        INT AUTO_INCREMENT PRIMARY KEY,
  username  VARCHAR(100) NOT NULL,
  email     VARCHAR(150) NOT NULL,
  password  VARCHAR(255) NOT NULL,
  is_active ENUM('ACTIVE','INACTIVE') NOT NULL DEFAULT 'ACTIVE',
  avatar    VARCHAR(255) NULL,
  CONSTRAINT uq_users_username UNIQUE (username),
  CONSTRAINT uq_users_email    UNIQUE (email)
) ENGINE=InnoDB;

CREATE TABLE roles (
  id        INT AUTO_INCREMENT PRIMARY KEY,
  name      VARCHAR(100) NOT NULL,
  is_active ENUM('ACTIVE','INACTIVE') NOT NULL DEFAULT 'ACTIVE',
  CONSTRAINT uq_roles_name UNIQUE (name)
) ENGINE=InnoDB;

CREATE TABLE role_users (
  id        INT AUTO_INCREMENT PRIMARY KEY,
  role_id   INT NOT NULL,
  user_id   INT NOT NULL,
  is_active ENUM('ACTIVE','INACTIVE') NOT NULL DEFAULT 'ACTIVE',
  CONSTRAINT fk_role_users_role FOREIGN KEY (role_id) REFERENCES roles(id),
  CONSTRAINT fk_role_users_user FOREIGN KEY (user_id) REFERENCES users(id),
  CONSTRAINT uq_role_users UNIQUE (role_id, user_id)
) ENGINE=InnoDB;

CREATE TABLE resources (
  id        INT AUTO_INCREMENT PRIMARY KEY,
  path      VARCHAR(255) NOT NULL,
  method    VARCHAR(10)  NOT NULL,
  is_active ENUM('ACTIVE','INACTIVE') NOT NULL DEFAULT 'ACTIVE',
  CONSTRAINT uq_resources_path_method UNIQUE (path, method)
) ENGINE=InnoDB;

CREATE TABLE resource_roles (
  id          INT AUTO_INCREMENT PRIMARY KEY,
  resource_id INT NOT NULL,
  role_id     INT NOT NULL,
  is_active   ENUM('ACTIVE','INACTIVE') NOT NULL DEFAULT 'ACTIVE',
  CONSTRAINT fk_resource_roles_resource FOREIGN KEY (resource_id) REFERENCES resources(id),
  CONSTRAINT fk_resource_roles_role     FOREIGN KEY (role_id)     REFERENCES roles(id),
  CONSTRAINT uq_resource_roles UNIQUE (resource_id, role_id)
) ENGINE=InnoDB;

CREATE TABLE refresh_tokens (
  id          INT AUTO_INCREMENT PRIMARY KEY,
  user_id     INT NOT NULL,
  token       VARCHAR(500) NOT NULL,
  device_info VARCHAR(255) NOT NULL,
  is_valid    ENUM('ACTIVE','INACTIVE') NOT NULL DEFAULT 'ACTIVE',
  expires_at  DATETIME NOT NULL,
  created_at  DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at  DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  CONSTRAINT fk_refresh_tokens_user FOREIGN KEY (user_id) REFERENCES users(id),
  INDEX idx_refresh_tokens_user (user_id)
) ENGINE=InnoDB;

-- =====================================================================
-- 2. MODELO DE NEGOCIO
-- =====================================================================
CREATE TABLE clientes (
  id               INT AUTO_INCREMENT PRIMARY KEY,
  tipo_documento   VARCHAR(10)  NOT NULL,
  numero_documento VARCHAR(30)  NOT NULL,
  nombre           VARCHAR(150) NOT NULL,
  telefono         VARCHAR(30)  NULL,
  email            VARCHAR(150) NULL,
  is_active        ENUM('ACTIVE','INACTIVE') NOT NULL DEFAULT 'ACTIVE',
  CONSTRAINT uq_clientes_documento UNIQUE (numero_documento)
) ENGINE=InnoDB;

CREATE TABLE salones (
  id          INT AUTO_INCREMENT PRIMARY KEY,
  nombre      VARCHAR(120) NOT NULL,
  descripcion VARCHAR(500) NULL,
  capacidad   INT NOT NULL DEFAULT 1,
  precio_base DECIMAL(14,2) NOT NULL DEFAULT 0,
  is_active   ENUM('ACTIVE','INACTIVE') NOT NULL DEFAULT 'ACTIVE',
  created_at  DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at  DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  CONSTRAINT uq_salones_nombre UNIQUE (nombre),
  CONSTRAINT ck_salones_capacidad CHECK (capacidad > 0),
  CONSTRAINT ck_salones_precio CHECK (precio_base >= 0)
) ENGINE=InnoDB;

CREATE TABLE servicios (
  id          INT AUTO_INCREMENT PRIMARY KEY,
  nombre      VARCHAR(120) NOT NULL,
  descripcion VARCHAR(500) NULL,
  precio_base DECIMAL(14,2) NOT NULL DEFAULT 0,
  is_active   ENUM('ACTIVE','INACTIVE') NOT NULL DEFAULT 'ACTIVE',
  created_at  DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at  DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  CONSTRAINT uq_servicios_nombre UNIQUE (nombre),
  CONSTRAINT ck_servicios_precio CHECK (precio_base >= 0)
) ENGINE=InnoDB;

CREATE TABLE proveedores (
  id           INT AUTO_INCREMENT PRIMARY KEY,
  nit          VARCHAR(30)  NOT NULL,
  razon_social VARCHAR(150) NOT NULL,
  contacto     VARCHAR(120) NULL,
  telefono     VARCHAR(30)  NULL,
  email        VARCHAR(150) NULL,
  is_active    ENUM('ACTIVE','INACTIVE') NOT NULL DEFAULT 'ACTIVE',
  CONSTRAINT uq_proveedores_nit UNIQUE (nit)
) ENGINE=InnoDB;

CREATE TABLE reservas (
  id                INT AUTO_INCREMENT PRIMARY KEY,
  cliente_id        INT NOT NULL,
  salon_id          INT NOT NULL,
  fecha_inicio      DATETIME NOT NULL,
  fecha_fin         DATETIME NOT NULL,
  estado            ENUM('PRE_RESERVA','CONFIRMADA','CANCELADA','FINALIZADA') NOT NULL DEFAULT 'PRE_RESERVA',
  bloqueo_expira_at DATETIME NULL,
  observaciones     VARCHAR(500) NULL,
  CONSTRAINT fk_reservas_cliente FOREIGN KEY (cliente_id) REFERENCES clientes(id),
  CONSTRAINT fk_reservas_salon   FOREIGN KEY (salon_id)   REFERENCES salones(id),
  CONSTRAINT ck_reservas_fechas CHECK (fecha_fin > fecha_inicio),
  INDEX idx_reservas_disponibilidad (salon_id, fecha_inicio, fecha_fin)
) ENGINE=InnoDB;

CREATE TABLE eventos (
  id            INT AUTO_INCREMENT PRIMARY KEY,
  reserva_id    INT NOT NULL,                -- referencia_id (FK) hacia reservas
  tipo          VARCHAR(60) NOT NULL,
  fecha         DATETIME NOT NULL,
  cantidad      INT NOT NULL,                -- numero de asistentes
  observaciones VARCHAR(500) NULL,
  estado        ENUM('PLANEADO','EN_CURSO','REALIZADO','CANCELADO') NOT NULL DEFAULT 'PLANEADO',
  CONSTRAINT fk_eventos_reserva FOREIGN KEY (reserva_id) REFERENCES reservas(id),
  CONSTRAINT uq_eventos_reserva UNIQUE (reserva_id),   -- Reserva 0..1 : 1 Evento
  CONSTRAINT ck_eventos_cantidad CHECK (cantidad > 0)
) ENGINE=InnoDB;

CREATE TABLE evento_servicios (
  id              INT AUTO_INCREMENT PRIMARY KEY,
  evento_id       INT NOT NULL,
  servicio_id     INT NOT NULL,
  proveedor_id    INT NOT NULL,
  tipo            VARCHAR(60) NOT NULL,
  fecha           DATETIME NOT NULL,
  cantidad        INT NOT NULL DEFAULT 1,
  precio_unitario DECIMAL(14,2) NOT NULL DEFAULT 0,
  observaciones   VARCHAR(500) NULL,
  estado          ENUM('PENDIENTE','CONFIRMADO','CANCELADO') NOT NULL DEFAULT 'PENDIENTE',
  CONSTRAINT fk_evser_evento    FOREIGN KEY (evento_id)    REFERENCES eventos(id),
  CONSTRAINT fk_evser_servicio  FOREIGN KEY (servicio_id)  REFERENCES servicios(id),
  CONSTRAINT fk_evser_proveedor FOREIGN KEY (proveedor_id) REFERENCES proveedores(id),
  CONSTRAINT ck_evser_cantidad CHECK (cantidad > 0)
) ENGINE=InnoDB;

CREATE TABLE contratos (
  id                  INT AUTO_INCREMENT PRIMARY KEY,
  cliente_id          INT NOT NULL,
  evento_id           INT NOT NULL,
  numero              VARCHAR(30) NOT NULL,
  fecha_inicio        DATE NOT NULL,
  fecha_fin           DATE NOT NULL,
  valor               DECIMAL(14,2) NOT NULL,
  porcentaje_anticipo DECIMAL(5,2) NOT NULL DEFAULT 30.00,
  version_actual      INT NOT NULL DEFAULT 1,
  estado              ENUM('BORRADOR','VIGENTE','MODIFICADO','CANCELADO','FINALIZADO') NOT NULL DEFAULT 'BORRADOR',
  CONSTRAINT fk_contratos_cliente FOREIGN KEY (cliente_id) REFERENCES clientes(id),
  CONSTRAINT fk_contratos_evento  FOREIGN KEY (evento_id)  REFERENCES eventos(id),
  CONSTRAINT uq_contratos_numero UNIQUE (numero),
  CONSTRAINT uq_contratos_evento UNIQUE (evento_id),     -- Evento 1 : 1 Contrato
  CONSTRAINT ck_contratos_fechas CHECK (fecha_fin >= fecha_inicio),
  CONSTRAINT ck_contratos_valor  CHECK (valor >= 0),
  CONSTRAINT ck_contratos_anticipo CHECK (porcentaje_anticipo BETWEEN 0 AND 100)
) ENGINE=InnoDB;

CREATE TABLE pagos (
  id              INT AUTO_INCREMENT PRIMARY KEY,
  contrato_id     INT NOT NULL,
  referencia_tipo VARCHAR(30) NOT NULL DEFAULT 'CONTRATO',
  referencia_id   INT NOT NULL,
  tipo_pago       ENUM('ANTICIPO','ABONO','SALDO','PENALIDAD') NOT NULL DEFAULT 'ABONO',
  metodo          VARCHAR(30) NOT NULL,
  monto           DECIMAL(14,2) NOT NULL,
  fecha           DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  estado          ENUM('PENDIENTE','APLICADO','ANULADO') NOT NULL DEFAULT 'PENDIENTE',
  CONSTRAINT fk_pagos_contrato FOREIGN KEY (contrato_id) REFERENCES contratos(id),
  CONSTRAINT ck_pagos_monto CHECK (monto > 0),
  INDEX idx_pagos_referencia (referencia_tipo, referencia_id)
) ENGINE=InnoDB;

CREATE TABLE cambios_contrato (
  id             INT AUTO_INCREMENT PRIMARY KEY,
  contrato_id    INT NOT NULL,
  version        INT NOT NULL,
  nombre         VARCHAR(120) NOT NULL,
  descripcion    VARCHAR(500) NULL,
  valor_anterior DECIMAL(14,2) NOT NULL,
  valor_nuevo    DECIMAL(14,2) NOT NULL,
  estado         ENUM('SOLICITADO','APROBADO','RECHAZADO') NOT NULL DEFAULT 'SOLICITADO',
  is_active      ENUM('ACTIVE','INACTIVE') NOT NULL DEFAULT 'ACTIVE',
  created_at     DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at     DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  CONSTRAINT fk_cambios_contrato FOREIGN KEY (contrato_id) REFERENCES contratos(id),
  CONSTRAINT uq_cambios_version UNIQUE (contrato_id, version)
) ENGINE=InnoDB;

CREATE TABLE cancelaciones (
  id                  INT AUTO_INCREMENT PRIMARY KEY,
  contrato_id         INT NOT NULL,
  nombre              VARCHAR(120) NOT NULL,
  descripcion         VARCHAR(500) NULL,
  porcentaje_penalidad DECIMAL(5,2) NOT NULL DEFAULT 0,
  valor_penalidad     DECIMAL(14,2) NOT NULL DEFAULT 0,
  estado              ENUM('SOLICITADA','APLICADA','ANULADA') NOT NULL DEFAULT 'SOLICITADA',
  is_active           ENUM('ACTIVE','INACTIVE') NOT NULL DEFAULT 'ACTIVE',
  created_at          DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at          DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  CONSTRAINT fk_cancelaciones_contrato FOREIGN KEY (contrato_id) REFERENCES contratos(id),
  CONSTRAINT ck_cancel_porcentaje CHECK (porcentaje_penalidad BETWEEN 0 AND 100)
) ENGINE=InnoDB;

-- =====================================================================
-- 3. DATOS SEMILLA (RBAC)
-- =====================================================================
INSERT INTO roles (name) VALUES
  ('ADMIN'), ('COMERCIAL'), ('OPERACIONES'), ('PROVEEDOR'), ('CARTERA');

INSERT INTO resources (path, method) VALUES
  ('/reservas', 'POST'),
  ('/contratos', 'POST'),
  ('/cambios-contrato', 'POST'),
  ('/cancelaciones', 'POST');

-- ADMIN accede a todos los recursos
INSERT INTO resource_roles (resource_id, role_id)
SELECT r.id, ro.id FROM resources r CROSS JOIN roles ro WHERE ro.name = 'ADMIN';

-- COMERCIAL: reservas y contratos | OPERACIONES: reservas y cambios | CARTERA: contratos y cancelaciones
INSERT INTO resource_roles (resource_id, role_id)
SELECT r.id, ro.id FROM resources r JOIN roles ro
  ON (ro.name = 'COMERCIAL'   AND r.path IN ('/reservas','/contratos'))
  OR (ro.name = 'OPERACIONES' AND r.path IN ('/reservas','/cambios-contrato'))
  OR (ro.name = 'CARTERA'     AND r.path IN ('/contratos','/cancelaciones'));

-- Usuario administrador de prueba. IMPORTANTE: reemplazar el hash por uno generado con bcrypt desde tu backend.
INSERT INTO users (username, email, password) VALUES
  ('admin', 'admin@celebrahub.com', '$2b$10$REEMPLAZAR_POR_HASH_BCRYPT_GENERADO_EN_EL_BACKEND');

INSERT INTO role_users (role_id, user_id)
SELECT ro.id, u.id FROM roles ro, users u WHERE ro.name = 'ADMIN' AND u.username = 'admin';

-- =====================================================================
-- 4. DATOS SEMILLA (NEGOCIO)
-- =====================================================================
INSERT INTO clientes (tipo_documento, numero_documento, nombre, telefono, email) VALUES
  ('CC', '1001234567', 'Laura Martinez', '3001112233', 'laura@mail.com'),
  ('NIT', '900123456-1', 'Empresa Andina SAS', '6015556677', 'eventos@andina.com');

INSERT INTO salones (nombre, descripcion, capacidad, precio_base) VALUES
  ('Salon Perla',  'Salon principal con terraza', 250, 4500000),
  ('Salon Coral',  'Salon mediano para eventos corporativos', 120, 2500000),
  ('Salon Brisa',  'Salon intimo para celebraciones pequenas', 60, 1500000);

INSERT INTO servicios (nombre, descripcion, precio_base) VALUES
  ('Catering', 'Alimentos y bebidas por persona', 85000),
  ('Mobiliario', 'Mesas, sillas y mantelería', 1200000),
  ('Decoracion', 'Ambientacion tematica', 1800000),
  ('Sonido e iluminacion', 'DJ, sonido e iluminacion', 2000000);

INSERT INTO proveedores (nit, razon_social, contacto, telefono, email) VALUES
  ('800111222-3', 'Sabores del Caribe SAS', 'Carlos Perez', '3109998877', 'ventas@sabores.com'),
  ('800333444-5', 'Eventos Sonido Pro', 'Ana Rojas', '3125554433', 'contacto@sonidopro.com');
```

Después de crear las tablas, las listo para verificar que quedaron registradas:

```sql
SHOW TABLES;
```

Al consultar `CelebraHub_Centro_de_eventos`, verifico las tablas principales de mi proyecto:

```text
cambios_contrato
cancelaciones
clientes
contratos
evento_servicios
eventos
pagos
proveedores
refresh_tokens
reservas
resource_roles
resources
role_users
roles
salones
servicios
users
```


Listo los campos de una tabla:

```sql
DESCRIBE nombre_tabla;
```

También puedo listar los campos con:

```sql
SHOW COLUMNS FROM nombre_tabla;
```

Salgo del motor de MySQL:

```sql
EXIT;
```

evidencia 

![alt text](image-22.png)

![alt text](image-23.png)

![alt text](image-24.png)

Registro:

Me conecté localmente al motor de MySQL, seleccioné la base de datos del proyecto, creé las 17 tablas del sistema CelebraHub - Centro de eventos: users, roles, role_users, resources, resource_roles, refresh_tokens (modelo RBAC) y clientes, salones, servicios, proveedores, reservas, eventos, evento_servicios, contratos, pagos, cambios_contrato y cancelaciones (modelo de negocio). También se establecieron las claves primarias, claves foráneas, restricciones únicas y de validación (CHECK), índices y los datos semilla. Finalmente, verifiqué la creación de las tablas mediante SHOW TABLES y salí correctamente del motor de MySQL.

### 1.8 Crear un usuario propio con acceso remoto

Primero me conecto como root al motor de MySQL que se encuentra ejecutándose en Docker:

``` bash 
sudo docker exec -it mysql-server mysql -u root -p
```

Después de ingresar la contraseña de root, ejecuto las instrucciones SQL correspondientes a mi proyecto de gestión de centro de eventos.

Parte explicativa:

Código original de la guía del docente:

```sql
-- Crear la base de datos
CREATE DATABASE CelebraHub_Centro_de_eventos CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- Crear usuario propio con acceso desde cualquier equipo (%)
CREATE USER 'einer_coa'@'%' IDENTIFIED BY 'einer123';

-- Dar permisos sobre la base de datos
GRANT ALL PRIVILEGES ON CelebraHub_Centro_de_eventos.* TO 'einer_coa'@'%';
FLUSH PRIVILEGES;
```
Código adaptado a mi proyecto:

En mi proyecto, la base de datos se denomina `CelebraHub_Centro_de_eventos`. Para administrarla se utiliza el usuario `einer_coa`, con acceso remoto desde cualquier equipo mediante `%` y la contraseña establecida es `einer123`.

``` sql

-- Crear la base de datos del proyecto
CREATE DATABASE IF NOT EXISTS CelebraHub_Centro_de_eventos
CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;

-- Seleccionar la base de datos del proyecto
USE CelebraHub_Centro_de_eventos;

-- Crear usuario administrador con acceso remoto
CREATE USER 'einer_coa'@'%' IDENTIFIED BY 'einer123';

-- Otorgar todos los privilegios sobre la base de datos del proyecto
GRANT ALL PRIVILEGES ON CelebraHub_Centro_de_eventos.* TO 'einer_coa'@'%';
FLUSH PRIVILEGES;
```
Evidencia:

![alt text](image-25.png)


**Aplicado en mi proyecto:**

Creo el usuario `einer_coa` con la contraseña `einer123` y le otorgo permisos para conectarse remotamente a la base de datos `CelebraHub_Centro_de_eventos`:

```sql

CREATE USER 'einer_coa'@'%' IDENTIFIED BY 'einer123';
GRANT ALL PRIVILEGES ON CelebraHub_Centro_de_eventos
.* TO 'einer_coa'@'%';
FLUSH PRIVILEGES;
```

evidencia 

![alt text](image-26.png)

**Nota:**

El mensaje indica que el usuario `einer_coa` ya existe. El comando siguiente permite corregir la situación.


### 1.9 Conectar remotamente desde cualquier equipo con DBeaver

Para conectarme de forma remota a MySQL utilizo DBeaver. Necesito los siguientes datos:

- **Motor:** MySQL.
- **Servidor:** la dirección IP del equipo donde está ejecutándose Docker.
- **Puerto:** `3306`.
- **Base de datos:** `CelebraHub_Centro_de_eventos`.
- **Usuario:** `einer_coa`.
- **Contraseña:** `einer123`.

Para encontrar y comprobar estos datos, ejecuto los siguientes comandos en WSL.

Busco la dirección IP del equipo donde se ejecuta Docker:

```bash
ip a
```

En el resultado busco la interfaz `eth0` y tomo la dirección que aparece después de `inet`. En mi caso, la dirección es `172.18.50.255`.

Compruebo que el contenedor MySQL esté ejecutándose y que el puerto `3306` esté publicado:

```bash
sudo docker ps --filter "name=mysql-server" --format "table {{.Names}}\t{{.Ports}}"
```

Compruebo que el servidor MySQL esté funcionando y responda correctamente:

```bash
sudo docker exec mysql-server mysqladmin ping -h localhost -u root -peiner123
```

El resultado fue `mysqld is alive`, por lo que confirmé que el servidor MySQL está funcionando correctamente. También apareció una advertencia indicando que usar la contraseña directamente en la línea de comandos puede ser inseguro.

También compruebo que el puerto esté permitido por UFW:

```bash
sudo ufw status
```

Consulto el nombre de la base de datos configurada en el archivo `.env`:

```bash
grep '^MYSQL_DATABASE=' ~/ia-lab/services/motores-bd/mysql/.env
```

Consulto los usuarios de MySQL y sus equipos de acceso:

```bash
sudo docker exec -it mysql-server mysql -u root -p -e "SELECT User, Host FROM mysql.user;"
```

Compruebo que el usuario `einer_coa` tenga permisos sobre `CelebraHub_Centro_de_eventos`:

```bash
sudo docker exec -it mysql-server mysql -u root -p -e "SHOW GRANTS FOR 'einer_coa'@'%';"
```

```bash
sudo docker exec -it mysql-server mysql -u root -p -e "ALTER USER 'einer_coa'@'%' IDENTIFIED WITH mysql_native_password BY 'einer123'; FLUSH PRIVILEGES;"
```

```bash
sudo docker exec -it mysql-server mysql -u root -p -e "SELECT User, Host, plugin FROM mysql.user WHERE User = 'einer_coa';"
```

evidencia

![alt text](image-27.png)

![alt text](image-28.png)

En DBeaver realizo los siguientes pasos:

1. Abro DBeaver.
2. Selecciono **Nueva conexión**.
3. Elijo el controlador **MySQL**.
4. Busco la interfaz `eth0` en el resultado de `ip a` y escribo ` 172.17.55.39` en el campo **Host**.
5. Escribo `3306` en **Port**.
6. Escribo `einer_coa` en **Username**.
7. Escribo `einer123` en **Password**.
8. Escribo `CelebraHub_Centro_de_eventos` en **Database**.
9. Presiono **Test Connection**.
10. Si la prueba es correcta, presiono **Finish** para guardar la conexión.


evidencia de la aplicacion DB

![alt text](image-29.png)


![alt text](image-30.png)

![alt text](image-31.png)

![alt text](image-33.png)


**Registro:**

Después de corregir el método de autenticación del usuario `einer_coa`, probé nuevamente la conexión en DBeaver. El resultado fue exitoso: apareció el mensaje `Conectado (244 ms)` y se confirmó la conexión con MySQL `8.0.46` usando la base de datos `CelebraHub_Centro_de_eventos`.

### 1.10 Backup de la base de datos

Realizo un backup de la base de datos `CelebraHub_Centro_de_eventos` y guardo el archivo en la carpeta de respaldos:

```bash
sudo docker exec mysql-server mysqldump -u root -peiner123 CelebraHub_Centro_de_eventos > /mnt/d/academia/bd/backup_CelebraHub_Centro_de_eventos_$(date +%Y%m%d).sql
```

Verifico que el archivo de backup se haya creado correctamente:

```bash
ls -lh /mnt/d/academia/bd/backup_CelebraHub_Centro_de_eventos_$(date +%Y%m%d).sql
```

**Evidencia (imagen):**

![Evidencia del error de permisos al crear el backup](image-34.png)


**Registro:**

Al ejecutar el backup, apareció el error `Permission denied` porque no tenía permisos para crear el archivo directamente en `/mnt/d/academia/bd/` mediante la redirección `>`. El backup todavía no se creó.

**Diferencia entre la guía del docente y mi solución:**

El comando original de la página web del docente es:

```bash
sudo docker exec mysql-server mysqldump -u root -pMiNiCo57** tecnogua > /mnt/d/academia/bd/backup_tecnogua_$(date +%Y%m%d).sql
```

En ese comando, `mysqldump` se ejecuta dentro del contenedor, pero la redirección `>` intenta crear el archivo desde WSL en `/mnt/d/academia/bd/`. En mi caso, esa redirección produjo el error `Permission denied`.

Mi solución fue crear el backup directamente dentro del contenedor, usando la carpeta `/backups`, que está conectada con `/mnt/d/academia/bd/`. Además, cambié los datos de la guía por los de mi proyecto: la base de datos es `CelebraHub_Centro_de_eventos` y la contraseña de `root` es `einer123`.

Como la carpeta `/mnt/d/academia/bd/` está montada en el contenedor como `/backups`, genero el archivo directamente dentro del contenedor. Así no uso la redirección de Bash en WSL ni `tee`:

```bash
sudo docker exec -it mysql-server mysql -u root -p
```

Dentro de MySQL creo el backup directamente en la carpeta montada:

```sql
SYSTEM mysqldump -u root -peiner123 CelebraHub_Centro_de_eventos > /backups/backup_CelebraHub_Centro_de_eventos_$(date +%Y%m%d).sql;
```

Salgo del motor:

```sql
EXIT;
```

Después verifico que el backup se haya creado correctamente:

```bash
ls -lh /mnt/d/academia/bd/backup_CelebraHub_Centro_de_eventos_$(date +%Y%m%d).sql
```

**Evidencia (imagen):**

![Evidencia del backup creado correctamente](image-35.png)


**Registro:**

Creé el backup de la base de datos `CelebraHub_Centro_de_eventos` directamente dentro del contenedor y verifiqué desde WSL que el archivo se generó correctamente en `/mnt/d/academia/bd/`.

### 1.10 Variables clave del `.env`

Estas son las variables principales que configuré en el archivo `.env`:

| Variable | Descripción |
| --- | --- |
| `TZ` | Zona horaria: `America/Bogota` |
| `MYSQL_ROOT_PASSWORD` | Contraseña del usuario root: `einer123` |
| `MYSQL_DATABASE` | Base de datos inicial: `CelebraHub_Centro_de_eventos` |



**Registro:**

Registré las variables principales del archivo `.env`: la zona horaria, la contraseña del usuario root y la base de datos inicial del proyecto final, `CelebraHub_Centro_de_eventos`.

Con este registro finalizo la instalación del motor MySQL para mi proyecto `CelebraHub_Centro_de_eventos`. Dejé el contenedor funcionando, configuré la base de datos, creé sus tablas, habilité el acceso remoto, comprobé la conexión desde DBeaver y generé el backup correspondiente.

## 2. PostgreSQL

Ahora continúo con la instalación y configuración del motor PostgreSQL siguiendo el mismo procedimiento de registro paso a paso.

### 2.1 Crear `docker-compose.yml`

Creo el archivo de configuración de PostgreSQL en la carpeta correspondiente:

```bash
cat > ~/ia-lab/services/motores-bd/postgres/docker-compose.yml << 'EOF'
services:
  postgres:
    image: postgres:17
    container_name: postgres-server
    restart: unless-stopped

    env_file:
      - .env

    ports:
      - "5432:5432"

    volumes:
      # Datos persistentes dentro de WSL
      - ../../../data/postgres:/var/lib/postgresql/data

      # Backups accesibles desde Windows (D:)
      - /mnt/d/academia/bd:/backups

    command:
      - postgres
      - -c
      - "listen_addresses=*"

    networks:
      - ia-lab-network

    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $$POSTGRES_USER -d $$POSTGRES_DB"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 20s

networks:
  ia-lab-network:
    external: true
EOF
```

Habilito UFW y permito el puerto `5432` de PostgreSQL en el host:

```bash
sudo ufw allow 5432/tcp
sudo ufw enable
sudo ufw status
```

**Evidencia (imagen):**



![Evidencia de la creación de docker-compose de PostgreSQL](image-36.png)

![Evidencia de la configuración de PostgreSQL](image-37.png)

**Registro:**

Creé el archivo `docker-compose.yml` de PostgreSQL con la imagen `postgres:17`, el puerto `5432`, el almacenamiento persistente, la carpeta de backups y la red compartida `ia-lab-network`. También permití el puerto `5432` en UFW.

### 2.2 Crear el archivo `.env`

Creo el archivo `.env` para configurar PostgreSQL con los datos de mi proyecto `CelebraHub_Centro_de_eventos`:

```bash
cat > ~/ia-lab/services/motores-bd/postgres/.env << 'EOF'
TZ=America/Bogota
POSTGRES_USER=postgres
POSTGRES_PASSWORD=einer123
POSTGRES_DB=CelebraHub_Centro_de_eventos
EOF
```

Para comprobar que el archivo `.env` quedó creado con los datos correctos, muestro su contenido:

```bash
cat ~/ia-lab/services/motores-bd/postgres/.env
```

Debo verificar que aparezcan estas variables:

```text
TZ=America/Bogota
POSTGRES_USER=postgres
POSTGRES_PASSWORD=einer123
POSTGRES_DB=CelebraHub_Centro_de_eventos
```

**Evidencia (imagen):**



**Registro:**

![Evidencia de la verificación del archivo .env de PostgreSQL](image-38.png)

**Registro:**

Creé el archivo `.env` de PostgreSQL con el usuario `postgres`, la contraseña `einer123` y la base de datos `CelebraHub_Centro_de_eventos`.

### 2.2.1 Si se quiere cambiar la contraseña del `.env`

Para editar la clave del archivo `.env` en clase, ingreso a la carpeta de PostgreSQL y abro el archivo:

```bash
cd ~/ia-lab/services/motores-bd/postgres
sudo nano .env
```

Cambio la clave por:

```env
POSTGRES_PASSWORD=123456
```

Guardo los cambios con:

```text
Ctrl + O  ==>  para guardar
```

Después presiono Enter para confirmar el nombre del archivo y salgo del editor con:

```text
Ctrl + X  ==>  para salir
```

### 2.3 Crear `README.md`

Creo el archivo `README.md` en la carpeta de PostgreSQL:

```bash
touch ~/ia-lab/services/motores-bd/postgres/README.md
```

Después abro el archivo para agregar su contenido:

```bash
sudo nano ~/ia-lab/services/motores-bd/postgres/README.md
```

Dentro del archivo escribo lo siguiente:

````markdown
# PostgreSQL 17 - Motor de Base de Datos

> **Acceso remoto habilitado.** Puerto expuesto en `0.0.0.0:5432`.
> **Usuario por defecto:** `postgres`
> **Base de datos inicial:** `CelebraHub_Centro_de_eventos`
> **Password:** `einer123`

---

## Conectar desde WSL (local)

```bash
sudo docker exec -it postgres-server psql -U postgres -d CelebraHub_Centro_de_eventos
# Password: einer123
```
````

Después de guardar el contenido, muestro el archivo para comprobar que fue creado correctamente:

```bash
cat ~/ia-lab/services/motores-bd/postgres/README.md
```

**Evidencia (imagen):**



**Registro:**

![Evidencia de la creación del README de PostgreSQL](image-39.png)

**Registro:**

Creé el archivo `README.md` de PostgreSQL y comprobé su contenido con el comando `cat`. Registré el puerto `5432`, el usuario `postgres`, la base de datos `CelebraHub_Centro_de_eventos` y el comando para conectarme desde WSL.

### 2.4 Levantar PostgreSQL

Ingreso a la carpeta de PostgreSQL y levanto el contenedor en segundo plano:

```bash
cd ~/ia-lab/services/motores-bd/postgres
```

```bash
sudo docker compose up -d
```

Verifico que el contenedor esté corriendo:

```bash
sudo docker ps | grep postgres-server
```

Después reviso los últimos 20 registros del contenedor:

```bash
sudo docker logs postgres-server --tail 20
```

**Evidencia (imagen):**

![Evidencia del error al levantar PostgreSQL](image-40.png)

**Registro:**

Al ejecutar `sudo docker compose up -d`, PostgreSQL no se levantó porque apareció el error `mapping key "networks" already defined at line 25` en la línea 35 del archivo YAML. El contenedor `postgres-server` todavía no pudo iniciarse.

### 2.4.1 Revisar y solucionar el error del archivo YAML

#### Código original

La parte original del archivo tenía la configuración de `networks` indentada con tabulaciones:

```yaml
services:
  postgres:
    networks:
      - ia-lab-network

networks:
  ia-lab-network:
    external: true
```

#### Error encontrado

El archivo produjo el error `mapping key "networks" already defined` porque YAML no permite tabulaciones para la indentación y las claves `networks` terminaron siendo interpretadas en el mismo nivel. La primera clave pertenece al servicio `postgres` y la segunda debe pertenecer al nivel principal del archivo.

Entro al archivo para reemplazar manualmente el código original:

```bash
cd ~/ia-lab/services/motores-bd/postgres
sudo nano docker-compose.yml
```

#### Código corregido

Después de entrar al archivo, reemplazo el contenido por el siguiente YAML, usando espacios:

```yaml
services:
  postgres:
    image: postgres:17
    container_name: postgres-server
    restart: unless-stopped
    env_file:
      - .env
    ports:
      - "5432:5432"
    volumes:
      - ../../../data/postgres:/var/lib/postgresql/data
      - /mnt/d/academia/bd:/backups
    command:
      - postgres
      - -c
      - "listen_addresses=*"
    networks:
      - ia-lab-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $$POSTGRES_USER -d $$POSTGRES_DB"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 20s

networks:
  ia-lab-network:
    external: true
```

Guardo con `Ctrl + O`, presiono Enter para confirmar y salgo con `Ctrl + X`.

Después valido el archivo corregido:

```bash
sudo docker compose config
```

Si no aparecen errores, vuelvo a levantar PostgreSQL y verifico el contenedor:

```bash
sudo docker compose up -d
sudo docker ps | grep postgres-server
sudo docker logs postgres-server --tail 20
```

**Evidencia (imagen):**

![Evidencia de PostgreSQL funcionando](image-41.png)

![alt text](image-42.png)

**Registro:**

Corregí la estructura del archivo `docker-compose.yml`, validé la configuración y levanté PostgreSQL correctamente. El contenedor `postgres-server` quedó funcionando y los registros confirmaron su inicio.

### 2.5 Conectar localmente a PostgreSQL

Este paso es diferente al de la guía del docente porque trabajo directamente con `CelebraHub_Centro_de_eventos`, la base de datos de mi proyecto final, en lugar de crear otra base de datos de práctica.

La estructura de mi proyecto final es **CelebraHub - Centro de eventos**:

![alt text](image-20.png)

Me conecto localmente al motor de PostgreSQL:

```bash
sudo docker exec -it postgres-server psql -U postgres -d CelebraHub_Centro_de_eventos
```

Listo todas las bases de datos:

```sql
\l
```

Selecciono la base de datos del proyecto:

```sql
\c CelebraHub_Centro_de_eventos
```

Como PostgreSQL todavía no tiene las tablas del proyecto, las creo dentro de `CelebraHub_Centro_de_eventos`:

```sql
CREATE SCHEMA IF NOT EXISTS celebrahub;
SET search_path TO celebrahub;

-- =====================================================================
-- 1. MODELO RBAC (features/auth)
-- =====================================================================
CREATE TABLE users (
  id        INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  username  VARCHAR(100) NOT NULL,
  email     VARCHAR(150) NOT NULL,
  password  VARCHAR(255) NOT NULL,
  is_active VARCHAR(8) NOT NULL DEFAULT 'ACTIVE',
  avatar    VARCHAR(255) NULL,
  CONSTRAINT uq_users_username UNIQUE (username),
  CONSTRAINT uq_users_email    UNIQUE (email),
  CONSTRAINT ck_users_active   CHECK (is_active IN ('ACTIVE','INACTIVE'))
);

CREATE TABLE roles (
  id        INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  name      VARCHAR(100) NOT NULL,
  is_active VARCHAR(8) NOT NULL DEFAULT 'ACTIVE',
  CONSTRAINT uq_roles_name   UNIQUE (name),
  CONSTRAINT ck_roles_active CHECK (is_active IN ('ACTIVE','INACTIVE'))
);

CREATE TABLE role_users (
  id        INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  role_id   INTEGER NOT NULL,
  user_id   INTEGER NOT NULL,
  is_active VARCHAR(8) NOT NULL DEFAULT 'ACTIVE',
  CONSTRAINT fk_role_users_role FOREIGN KEY (role_id) REFERENCES roles(id),
  CONSTRAINT fk_role_users_user FOREIGN KEY (user_id) REFERENCES users(id),
  CONSTRAINT uq_role_users UNIQUE (role_id, user_id),
  CONSTRAINT ck_role_users_active CHECK (is_active IN ('ACTIVE','INACTIVE'))
);

CREATE TABLE resources (
  id        INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  path      VARCHAR(255) NOT NULL,
  method    VARCHAR(10)  NOT NULL,
  is_active VARCHAR(8) NOT NULL DEFAULT 'ACTIVE',
  CONSTRAINT uq_resources_path_method UNIQUE (path, method),
  CONSTRAINT ck_resources_active CHECK (is_active IN ('ACTIVE','INACTIVE'))
);

CREATE TABLE resource_roles (
  id          INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  resource_id INTEGER NOT NULL,
  role_id     INTEGER NOT NULL,
  is_active   VARCHAR(8) NOT NULL DEFAULT 'ACTIVE',
  CONSTRAINT fk_resource_roles_resource FOREIGN KEY (resource_id) REFERENCES resources(id),
  CONSTRAINT fk_resource_roles_role     FOREIGN KEY (role_id)     REFERENCES roles(id),
  CONSTRAINT uq_resource_roles UNIQUE (resource_id, role_id),
  CONSTRAINT ck_resource_roles_active CHECK (is_active IN ('ACTIVE','INACTIVE'))
);

CREATE TABLE refresh_tokens (
  id          INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  user_id     INTEGER NOT NULL,
  token       VARCHAR(500) NOT NULL,
  device_info VARCHAR(255) NOT NULL,
  is_valid    VARCHAR(8) NOT NULL DEFAULT 'ACTIVE',
  expires_at  TIMESTAMP NOT NULL,
  created_at  TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at  TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  CONSTRAINT fk_refresh_tokens_user FOREIGN KEY (user_id) REFERENCES users(id),
  CONSTRAINT ck_refresh_tokens_valid CHECK (is_valid IN ('ACTIVE','INACTIVE'))
);
CREATE INDEX idx_refresh_tokens_user ON refresh_tokens (user_id);
CREATE INDEX idx_role_users_user ON role_users (user_id);
CREATE INDEX idx_resource_roles_role ON resource_roles (role_id);

-- =====================================================================
-- 2. MODELO DE NEGOCIO
-- =====================================================================
CREATE TABLE clientes (
  id               INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  tipo_documento   VARCHAR(10)  NOT NULL,
  numero_documento VARCHAR(30)  NOT NULL,
  nombre           VARCHAR(150) NOT NULL,
  telefono         VARCHAR(30)  NULL,
  email            VARCHAR(150) NULL,
  is_active        VARCHAR(8) NOT NULL DEFAULT 'ACTIVE',
  CONSTRAINT uq_clientes_documento UNIQUE (numero_documento),
  CONSTRAINT ck_clientes_active CHECK (is_active IN ('ACTIVE','INACTIVE'))
);

CREATE TABLE salones (
  id          INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  nombre      VARCHAR(120) NOT NULL,
  descripcion VARCHAR(500) NULL,
  capacidad   INTEGER NOT NULL DEFAULT 1,
  precio_base NUMERIC(14,2) NOT NULL DEFAULT 0,
  is_active   VARCHAR(8) NOT NULL DEFAULT 'ACTIVE',
  created_at  TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at  TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  CONSTRAINT uq_salones_nombre UNIQUE (nombre),
  CONSTRAINT ck_salones_capacidad CHECK (capacidad > 0),
  CONSTRAINT ck_salones_precio CHECK (precio_base >= 0),
  CONSTRAINT ck_salones_active CHECK (is_active IN ('ACTIVE','INACTIVE'))
);

CREATE TABLE servicios (
  id          INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  nombre      VARCHAR(120) NOT NULL,
  descripcion VARCHAR(500) NULL,
  precio_base NUMERIC(14,2) NOT NULL DEFAULT 0,
  is_active   VARCHAR(8) NOT NULL DEFAULT 'ACTIVE',
  created_at  TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at  TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  CONSTRAINT uq_servicios_nombre UNIQUE (nombre),
  CONSTRAINT ck_servicios_precio CHECK (precio_base >= 0),
  CONSTRAINT ck_servicios_active CHECK (is_active IN ('ACTIVE','INACTIVE'))
);

CREATE TABLE proveedores (
  id           INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  nit          VARCHAR(30)  NOT NULL,
  razon_social VARCHAR(150) NOT NULL,
  contacto     VARCHAR(120) NULL,
  telefono     VARCHAR(30)  NULL,
  email        VARCHAR(150) NULL,
  is_active    VARCHAR(8) NOT NULL DEFAULT 'ACTIVE',
  CONSTRAINT uq_proveedores_nit UNIQUE (nit),
  CONSTRAINT ck_proveedores_active CHECK (is_active IN ('ACTIVE','INACTIVE'))
);

CREATE TABLE reservas (
  id                INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  cliente_id        INTEGER NOT NULL,
  salon_id          INTEGER NOT NULL,
  fecha_inicio      TIMESTAMP NOT NULL,
  fecha_fin         TIMESTAMP NOT NULL,
  estado            VARCHAR(15) NOT NULL DEFAULT 'PRE_RESERVA',
  bloqueo_expira_at TIMESTAMP NULL,
  observaciones     VARCHAR(500) NULL,
  CONSTRAINT fk_reservas_cliente FOREIGN KEY (cliente_id) REFERENCES clientes(id),
  CONSTRAINT fk_reservas_salon   FOREIGN KEY (salon_id)   REFERENCES salones(id),
  CONSTRAINT ck_reservas_fechas CHECK (fecha_fin > fecha_inicio),
  CONSTRAINT ck_reservas_estado CHECK (estado IN ('PRE_RESERVA','CONFIRMADA','CANCELADA','FINALIZADA'))
);
CREATE INDEX idx_reservas_disponibilidad ON reservas (salon_id, fecha_inicio, fecha_fin);
CREATE INDEX idx_reservas_cliente ON reservas (cliente_id);

CREATE TABLE eventos (
  id            INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  reserva_id    INTEGER NOT NULL,            -- referencia_id (FK) hacia reservas
  tipo          VARCHAR(60) NOT NULL,
  fecha         TIMESTAMP NOT NULL,
  cantidad      INTEGER NOT NULL,            -- numero de asistentes
  observaciones VARCHAR(500) NULL,
  estado        VARCHAR(15) NOT NULL DEFAULT 'PLANEADO',
  CONSTRAINT fk_eventos_reserva FOREIGN KEY (reserva_id) REFERENCES reservas(id),
  CONSTRAINT uq_eventos_reserva UNIQUE (reserva_id),   -- Reserva 0..1 : 1 Evento
  CONSTRAINT ck_eventos_cantidad CHECK (cantidad > 0),
  CONSTRAINT ck_eventos_estado CHECK (estado IN ('PLANEADO','EN_CURSO','REALIZADO','CANCELADO'))
);

CREATE TABLE evento_servicios (
  id              INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  evento_id       INTEGER NOT NULL,
  servicio_id     INTEGER NOT NULL,
  proveedor_id    INTEGER NOT NULL,
  tipo            VARCHAR(60) NOT NULL,
  fecha           TIMESTAMP NOT NULL,
  cantidad        INTEGER NOT NULL DEFAULT 1,
  precio_unitario NUMERIC(14,2) NOT NULL DEFAULT 0,
  observaciones   VARCHAR(500) NULL,
  estado          VARCHAR(15) NOT NULL DEFAULT 'PENDIENTE',
  CONSTRAINT fk_evser_evento    FOREIGN KEY (evento_id)    REFERENCES eventos(id),
  CONSTRAINT fk_evser_servicio  FOREIGN KEY (servicio_id)  REFERENCES servicios(id),
  CONSTRAINT fk_evser_proveedor FOREIGN KEY (proveedor_id) REFERENCES proveedores(id),
  CONSTRAINT ck_evser_cantidad CHECK (cantidad > 0),
  CONSTRAINT ck_evser_estado CHECK (estado IN ('PENDIENTE','CONFIRMADO','CANCELADO'))
);
CREATE INDEX idx_evser_evento ON evento_servicios (evento_id);
CREATE INDEX idx_evser_servicio ON evento_servicios (servicio_id);
CREATE INDEX idx_evser_proveedor ON evento_servicios (proveedor_id);

CREATE TABLE contratos (
  id                  INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  cliente_id          INTEGER NOT NULL,
  evento_id           INTEGER NOT NULL,
  numero              VARCHAR(30) NOT NULL,
  fecha_inicio        DATE NOT NULL,
  fecha_fin           DATE NOT NULL,
  valor               NUMERIC(14,2) NOT NULL,
  porcentaje_anticipo NUMERIC(5,2) NOT NULL DEFAULT 30.00,
  version_actual      INTEGER NOT NULL DEFAULT 1,
  estado              VARCHAR(15) NOT NULL DEFAULT 'BORRADOR',
  CONSTRAINT fk_contratos_cliente FOREIGN KEY (cliente_id) REFERENCES clientes(id),
  CONSTRAINT fk_contratos_evento  FOREIGN KEY (evento_id)  REFERENCES eventos(id),
  CONSTRAINT uq_contratos_numero UNIQUE (numero),
  CONSTRAINT uq_contratos_evento UNIQUE (evento_id),     -- Evento 1 : 1 Contrato
  CONSTRAINT ck_contratos_fechas CHECK (fecha_fin >= fecha_inicio),
  CONSTRAINT ck_contratos_valor  CHECK (valor >= 0),
  CONSTRAINT ck_contratos_anticipo CHECK (porcentaje_anticipo BETWEEN 0 AND 100),
  CONSTRAINT ck_contratos_estado CHECK (estado IN ('BORRADOR','VIGENTE','MODIFICADO','CANCELADO','FINALIZADO'))
);
CREATE INDEX idx_contratos_cliente ON contratos (cliente_id);

CREATE TABLE pagos (
  id              INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  contrato_id     INTEGER NOT NULL,
  referencia_tipo VARCHAR(30) NOT NULL DEFAULT 'CONTRATO',
  referencia_id   INTEGER NOT NULL,
  tipo_pago       VARCHAR(15) NOT NULL DEFAULT 'ABONO',
  metodo          VARCHAR(30) NOT NULL,
  monto           NUMERIC(14,2) NOT NULL,
  fecha           TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  estado          VARCHAR(15) NOT NULL DEFAULT 'PENDIENTE',
  CONSTRAINT fk_pagos_contrato FOREIGN KEY (contrato_id) REFERENCES contratos(id),
  CONSTRAINT ck_pagos_monto CHECK (monto > 0),
  CONSTRAINT ck_pagos_tipo CHECK (tipo_pago IN ('ANTICIPO','ABONO','SALDO','PENALIDAD')),
  CONSTRAINT ck_pagos_estado CHECK (estado IN ('PENDIENTE','APLICADO','ANULADO'))
);
CREATE INDEX idx_pagos_contrato ON pagos (contrato_id);
CREATE INDEX idx_pagos_referencia ON pagos (referencia_tipo, referencia_id);

CREATE TABLE cambios_contrato (
  id             INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  contrato_id    INTEGER NOT NULL,
  version        INTEGER NOT NULL,
  nombre         VARCHAR(120) NOT NULL,
  descripcion    VARCHAR(500) NULL,
  valor_anterior NUMERIC(14,2) NOT NULL,
  valor_nuevo    NUMERIC(14,2) NOT NULL,
  estado         VARCHAR(15) NOT NULL DEFAULT 'SOLICITADO',
  is_active      VARCHAR(8) NOT NULL DEFAULT 'ACTIVE',
  created_at     TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at     TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  CONSTRAINT fk_cambios_contrato FOREIGN KEY (contrato_id) REFERENCES contratos(id),
  CONSTRAINT uq_cambios_version UNIQUE (contrato_id, version),
  CONSTRAINT ck_cambios_estado CHECK (estado IN ('SOLICITADO','APROBADO','RECHAZADO')),
  CONSTRAINT ck_cambios_active CHECK (is_active IN ('ACTIVE','INACTIVE'))
);

CREATE TABLE cancelaciones (
  id                   INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  contrato_id          INTEGER NOT NULL,
  nombre               VARCHAR(120) NOT NULL,
  descripcion          VARCHAR(500) NULL,
  porcentaje_penalidad NUMERIC(5,2) NOT NULL DEFAULT 0,
  valor_penalidad      NUMERIC(14,2) NOT NULL DEFAULT 0,
  estado               VARCHAR(15) NOT NULL DEFAULT 'SOLICITADA',
  is_active            VARCHAR(8) NOT NULL DEFAULT 'ACTIVE',
  created_at           TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at           TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  CONSTRAINT fk_cancelaciones_contrato FOREIGN KEY (contrato_id) REFERENCES contratos(id),
  CONSTRAINT ck_cancel_porcentaje CHECK (porcentaje_penalidad BETWEEN 0 AND 100),
  CONSTRAINT ck_cancel_estado CHECK (estado IN ('SOLICITADA','APLICADA','ANULADA')),
  CONSTRAINT ck_cancel_active CHECK (is_active IN ('ACTIVE','INACTIVE'))
);
CREATE INDEX idx_cancelaciones_contrato ON cancelaciones (contrato_id);

-- =====================================================================
-- 3. DATOS SEMILLA (RBAC)
-- =====================================================================
INSERT INTO roles (name) VALUES
  ('ADMIN'), ('COMERCIAL'), ('OPERACIONES'), ('PROVEEDOR'), ('CARTERA');

INSERT INTO resources (path, method) VALUES
  ('/reservas', 'POST'),
  ('/contratos', 'POST'),
  ('/cambios-contrato', 'POST'),
  ('/cancelaciones', 'POST');

-- ADMIN accede a todos los recursos
INSERT INTO resource_roles (resource_id, role_id)
SELECT r.id, ro.id FROM resources r CROSS JOIN roles ro WHERE ro.name = 'ADMIN';

-- COMERCIAL: reservas y contratos | OPERACIONES: reservas y cambios | CARTERA: contratos y cancelaciones
INSERT INTO resource_roles (resource_id, role_id)
SELECT r.id, ro.id FROM resources r JOIN roles ro
  ON (ro.name = 'COMERCIAL'   AND r.path IN ('/reservas','/contratos'))
  OR (ro.name = 'OPERACIONES' AND r.path IN ('/reservas','/cambios-contrato'))
  OR (ro.name = 'CARTERA'     AND r.path IN ('/contratos','/cancelaciones'));

-- Usuario administrador de prueba. IMPORTANTE: reemplazar el hash por uno generado con bcrypt desde tu backend.
INSERT INTO users (username, email, password) VALUES
  ('admin', 'admin@celebrahub.com', '$2b$10$REEMPLAZAR_POR_HASH_BCRYPT_GENERADO_EN_EL_BACKEND');

INSERT INTO role_users (role_id, user_id)
SELECT ro.id, u.id FROM roles ro, users u WHERE ro.name = 'ADMIN' AND u.username = 'admin';

-- =====================================================================
-- 4. DATOS SEMILLA (NEGOCIO)
-- =====================================================================
INSERT INTO clientes (tipo_documento, numero_documento, nombre, telefono, email) VALUES
  ('CC', '1001234567', 'Laura Martinez', '3001112233', 'laura@mail.com'),
  ('NIT', '900123456-1', 'Empresa Andina SAS', '6015556677', 'eventos@andina.com');

INSERT INTO salones (nombre, descripcion, capacidad, precio_base) VALUES
  ('Salon Perla', 'Salon principal con terraza', 250, 4500000),
  ('Salon Coral', 'Salon mediano para eventos corporativos', 120, 2500000),
  ('Salon Brisa', 'Salon intimo para celebraciones pequenas', 60, 1500000);

INSERT INTO servicios (nombre, descripcion, precio_base) VALUES
  ('Catering', 'Alimentos y bebidas por persona', 85000),
  ('Mobiliario', 'Mesas, sillas y manteleria', 1200000),
  ('Decoracion', 'Ambientacion tematica', 1800000),
  ('Sonido e iluminacion', 'DJ, sonido e iluminacion', 2000000);

INSERT INTO proveedores (nit, razon_social, contacto, telefono, email) VALUES
  ('800111222-3', 'Sabores del Caribe SAS', 'Carlos Perez', '3109998877', 'ventas@sabores.com'),
  ('800333444-5', 'Eventos Sonido Pro', 'Ana Rojas', '3125554433', 'contacto@sonidopro.com');
```

**Evidencia (imagen):**

![Evidencia de la creación de tablas en PostgreSQL](image-43.png)

**Registro:**

Me conecté localmente a PostgreSQL, seleccioné `CelebraHub_Centro_de_eventos` y creé las tablas del proyecto final correctamente dentro del esquema `celebrahub`.

### 2.5.1 Consultar, editar o eliminar información

Para comprobar qué tablas existen dentro de `CelebraHub_Centro_de_eventos`, ejecuto:

```sql
\dt celebrahub.*
```

Consulto los campos de una tabla:

```sql
\d celebrahub.clientes
```

También puedo cambiar de base de datos con:

```sql
\c CelebraHub_Centro_de_eventos
```

Para eliminar una base de datos, primero cambio a `postgres` y luego ejecuto:

```sql
\c postgres
DROP DATABASE CelebraHub_Centro_de_eventos;
```

Salgo del motor:

```sql
\q
```
### 2.5.2 Crear un usuario propio con acceso remoto

Me conecto primero como `postgres` al motor de PostgreSQL:

```bash
sudo docker exec -it postgres-server psql -U postgres -d CelebraHub_Centro_de_eventos
```

Después de ingresar la contraseña de `postgres`, ejecuto las instrucciones SQL.

**Código original de la guía del docente:**

Reemplaza `usuario`, `password` y `nombre_bd` por los valores de tu proyecto:

```sql
CREATE DATABASE nombre_bd;
CREATE USER usuario WITH PASSWORD 'password';
GRANT ALL PRIVILEGES ON DATABASE nombre_bd TO usuario;
ALTER DATABASE nombre_bd OWNER TO usuario;
```

**Código que colocaré en mi caso:**

La base de datos utilizada es `CelebraHub_Centro_de_eventos`, el usuario propio es `einer_coa` y la contraseña es `einer123`. Como la base de datos ya fue creada, solo creo el usuario y le asigno permisos de superusuario para permitir el acceso remoto y la administración del proyecto:

```sql
CREATE USER einer_coa WITH PASSWORD 'einer123';
ALTER USER einer_coa WITH SUPERUSER;
```

También puedo verificar que el usuario fue creado correctamente con:

```sql
\du
```

**Evidencia (imagen):**

![Evidencia de la creación del usuario einer_coa en PostgreSQL](image-44.png)

**Registro:**

Me conecté como `postgres` y creé el usuario propio `einer_coa` con la contraseña `einer123` para trabajar con la base de datos `CelebraHub_Centro_de_eventos` y acceder al motor PostgreSQL de forma remota.

### 2.6 Conectar remotamente desde cualquier equipo

Para conectarme de forma remota a PostgreSQL utilizo DBeaver. Necesito los siguientes datos:

- **Motor:** PostgreSQL.
- **Servidor:** la dirección IP del equipo donde está ejecutándose Docker.
- **Puerto:** `5432`.
- **Base de datos:** `CelebraHub_Centro_de_eventos`.
- **Usuario:** `einer_coa`.
- **Contraseña:** `einer123`.

Para encontrar y comprobar estos datos, ejecuto los siguientes comandos en WSL.

Busco la dirección IP del equipo donde se ejecuta Docker:

```bash
ip a
```

En el resultado busco la interfaz `eth0` y tomo la dirección que aparece después de `inet`.

Compruebo que el contenedor PostgreSQL esté ejecutándose y que el puerto `5432` esté publicado:

```bash
sudo docker ps --filter "name=postgres-server" --format "table {{.Names}}\t{{.Ports}}"
```

Compruebo que el servidor PostgreSQL esté funcionando y acepte conexiones:

```bash
sudo docker exec postgres-server pg_isready -U postgres -d CelebraHub_Centro_de_eventos
```

También compruebo que el puerto esté permitido por UFW:

```bash
sudo ufw status
```

**Evidencia (imagen):**

![Evidencia de la configuración para la conexión remota a PostgreSQL](image-45.png)

**Registro:**

Comprobé que la interfaz `eth0` tiene la dirección IP `172.18.50.255`. También confirmé que el contenedor `postgres-server` está activo y publica el puerto `5432` mediante `0.0.0.0:5432->5432/tcp`. El comando `pg_isready` mostró `/var/run/postgresql:5432 - accepting connections`, por lo que PostgreSQL está funcionando correctamente. Además, UFW está activo y permite el puerto `5432` para conexiones IPv4 e IPv6.

Con estos resultados, en DBeaver debo seleccionar el controlador **PostgreSQL** y utilizar los siguientes datos:

- **Host:** `172.18.50.255`
- **Port:** `5432`
- **Database:** `CelebraHub_Centro_de_eventos`
- **Username:** `einer_coa`
- **Password:** `einer123`

### 2.6.1 Hacerlo desde DBeaver

En DBeaver realizo los siguientes pasos:

1. Abro DBeaver.
2. Selecciono **Nueva conexión**.
3. Elijo el controlador **PostgreSQL**.
4. Escribo `172.18.50.255` en el campo **Host**.
5. Escribo `5432` en **Port**.
6. Escribo `einer_coa` en **Username**.
7. Escribo `einer123` en **Password**.
8. Escribo `CelebraHub_Centro_de_eventos` en **Database**.
9. Presiono **Test Connection**.
10. Si la prueba es correcta, presiono **Finish** para guardar la conexión.

**Evidencia (imagen):**

![Evidencia de la conexión a PostgreSQL desde DBeaver](image-46.png)

![Evidencia adicional de la conexión a PostgreSQL desde DBeaver](image-47.png)

**Registro:**

Configuré y probé en DBeaver la conexión remota a PostgreSQL utilizando el host `172.18.50.255`, el puerto `5432`, la base de datos `CelebraHub_Centro_de_eventos` y el usuario `einer_coa` con la contraseña `einer123`.

### 2.7 Backup de una base de datos

**Código original de la guía del docente:**

El siguiente comando pertenece únicamente al ejemplo de la guía del docente. No utilizo la base de datos `tecnogua`; en mi proyecto trabajo con `CelebraHub_Centro_de_eventos`.

```bash
sudo docker exec postgres-server pg_dump -U postgres -d tecnogua > /mnt/d/academia/bd/backup_tecnogua_$(date +%Y%m%d).sql
```

**Código que colocaré en mi caso:**

En mi proyecto utilizo la base de datos `CelebraHub_Centro_de_eventos`, la contraseña `einer123` y el nombre de archivo `backup_CelebraHub_Centro_de_eventos`. Para evitar el error de permisos de la redirección de WSL, genero el archivo directamente dentro del contenedor. La carpeta `/backups` está conectada con `/mnt/d/academia/bd/`:

```bash
sudo docker exec postgres-server pg_dump -U postgres -d CelebraHub_Centro_de_eventos -f /backups/backup_CelebraHub_Centro_de_eventos_$(date +%Y%m%d).sql
```

Compruebo el archivo generado desde WSL:

```bash
ls -lh /mnt/d/academia/bd/backup_CelebraHub_Centro_de_eventos_$(date +%Y%m%d).sql
```

Para ver lo que está guardado en el archivo `.env` de PostgreSQL, ejecuto:

```bash
cat ~/ia-lab/services/motores-bd/postgres/.env
```

El contenido esperado es:

```env
TZ=America/Bogota
POSTGRES_USER=postgres
POSTGRES_PASSWORD=einer123
POSTGRES_DB=CelebraHub_Centro_de_eventos
```

**Evidencia (imagen):**

![Evidencia del backup de CelebraHub_Centro_de_eventos y la consulta del archivo .env](image-48.png)


**Registro:**

Realicé el backup únicamente de la base de datos `CelebraHub_Centro_de_eventos` y verifiqué el archivo generado en `/mnt/d/academia/bd/`. También mostré el contenido del archivo `.env`, donde están guardados el usuario `postgres`, la contraseña `einer123` y la base de datos `CelebraHub_Centro_de_eventos`.

## 3. MS SQL Server

### 3.1 Crear el archivo `docker-compose.yml`

Creo el archivo `docker-compose.yml` para MS SQL Server:

```bash
cat > ~/ia-lab/services/motores-bd/mssql/docker-compose.yml << 'EOF'
services:
  mssql:
    image: mcr.microsoft.com/mssql/server:2022-latest
    container_name: mssql-server
    restart: unless-stopped
    user: root
    env_file:
      - .env
    ports:
      - "0.0.0.0:1433:1433"
    volumes:
      - ../../../data/mssql:/var/opt/mssql
      - /mnt/d/academia/bd:/backups
    networks:
      - ia-lab-network
    healthcheck:
      test: ["CMD-SHELL", "/opt/mssql-tools18/bin/sqlcmd -S localhost -U SA -P $$MSSQL_SA_PASSWORD -C -Q 'SELECT 1' || exit 1"]
      interval: 10s
      timeout: 5s
      retries: 10
      start_period: 40s

networks:
  ia-lab-network:
    external: true
EOF
```

Para ver el contenido del archivo `docker-compose.yml`, ejecuto:

```bash
cat ~/ia-lab/services/motores-bd/mssql/docker-compose.yml
```

Para entrar a editar el archivo, ejecuto:

```bash
sudo nano ~/ia-lab/services/motores-bd/mssql/docker-compose.yml
```

Habilito UFW y permito el puerto `1433` de SQL Server en el host:

```bash
sudo ufw allow 1433/tcp
sudo ufw enable
sudo ufw status
```

**Evidencia (imagen):**

![Evidencia de la configuración de MS SQL Server](image-49.png)

**Registro:**

Creé el archivo `docker-compose.yml` para MS SQL Server utilizando la imagen `mcr.microsoft.com/mssql/server:2022-latest`, el contenedor `mssql-server`, el puerto `1433`, los volúmenes para datos y backups, la red externa `ia-lab-network` y el healthcheck con `sqlcmd`. Verifiqué el contenido del archivo y utilicé espacios reales en la indentación YAML para evitar errores de formato. También habilité el puerto `1433/tcp` en UFW para permitir las conexiones a SQL Server.

### 3.2 Crear el archivo `.env`

Creo el archivo `.env` para configurar MS SQL Server con los datos de mi proyecto `CelebraHub_Centro_de_eventos`:

```bash
cat > ~/ia-lab/services/motores-bd/mssql/.env << 'EOF'
TZ=America/Bogota
ACCEPT_EULA=Y
MSSQL_SA_PASSWORD=CelebraHub_Centro_de_eventoseiner123!
MSSQL_PID=Developer
EOF
```

`MSSQL_PID` no es un usuario. Es la edición de SQL Server; en este caso utilizo la edición `Developer`.

El usuario administrador que crea el contenedor se llama `SA` (System Administrator). Por lo tanto, al conectarme mediante `sqlcmd` o DBeaver debo utilizar:

- **Usuario:** `SA`
- **Contraseña:** `CelebraHub_Centro_de_eventoseiner123!`
- **Edición:** `Developer`

Para ver el contenido del archivo `.env`, ejecuto:

```bash
cat ~/ia-lab/services/motores-bd/mssql/.env
```

El contenido esperado es:

```env
TZ=America/Bogota
ACCEPT_EULA=Y
MSSQL_SA_PASSWORD=CelebraHub_Centro_de_eventoseiner123!
MSSQL_PID=Developer
```

Para entrar a editar el archivo `.env`, ejecuto:

```bash
cd ~/ia-lab/services/motores-bd/mssql
sudo nano .env
```

Si necesito cambiar la contraseña, modifico esta línea:

```env
MSSQL_SA_PASSWORD=CelebraHub_Centro_de_eventoseiner123!
```

SQL Server exige que la contraseña de `SA` cumpla los requisitos de complejidad: debe incluir mayúsculas, minúsculas, números y símbolos. Por eso `einer123` sola no es válida y utilizo `CelebraHub_Centro_de_eventoseiner123!`.

Guardo los cambios con `Ctrl + O`, presiono Enter para confirmar y salgo con `Ctrl + X`.

**Evidencia (imagen):**

![Evidencia de la configuración del archivo .env de MS SQL Server](image-50.png)


**Registro:**

Creé y verifiqué el archivo `.env` de MS SQL Server con la zona horaria `America/Bogota`, la aceptación de la licencia mediante `ACCEPT_EULA=Y`, la edición `Developer` y el usuario administrador `SA`. Configuré la contraseña `CelebraHub_Centro_de_eventoseiner123!`, que cumple los requisitos de complejidad exigidos por SQL Server, y comprobé el contenido del archivo con el comando `cat`.

### 3.3 Crear `README.md`

Creo el archivo `README.md` para documentar la configuración de MS SQL Server:

````bash
cat > ~/ia-lab/services/motores-bd/mssql/README.md << 'EOF'
# SQL Server 2022 - Motor de Base de Datos

> **Acceso remoto habilitado.** Puerto expuesto en `0.0.0.0:1433`.
> **Edición (MSSQL_PID):** `Developer`
> **Usuario por defecto:** `SA`
> **Base de datos del proyecto:** `CelebraHub_Centro_de_eventos`
> **Password:** `CelebraHub_Centro_de_eventoseiner123!`

---

## Conectar desde WSL (local)

```bash
sudo docker exec -it mssql-server /opt/mssql-tools18/bin/sqlcmd -S localhost -U SA -P 'CelebraHub_Centro_de_eventoseiner123!' -C
```
EOF
````

Para ver el contenido del archivo `README.md`, ejecuto:

```bash
cat ~/ia-lab/services/motores-bd/mssql/README.md
```

**Evidencia (imagen):**

![Evidencia de la creación del README de MS SQL Server](image-51.png)
**Registro:**

Creé el archivo `README.md` de MS SQL Server 2022 y comprobé su contenido con el comando `cat`. Registré el puerto `1433`, la edición `Developer`, el usuario administrador `SA`, la base de datos del proyecto `CelebraHub_Centro_de_eventos` y la contraseña `CelebraHub_Centro_de_eventoseiner123!`. También dejé documentado el comando `sqlcmd` para conectarme localmente desde WSL.

### 3.4 Levantar SQL Server

Ingreso a la carpeta de MS SQL Server y levanto el contenedor en segundo plano:

```bash
cd ~/ia-lab/services/motores-bd/mssql
sudo docker compose up -d
```

Verifico que el contenedor `mssql-server` esté corriendo:

```bash
sudo docker ps | grep mssql-server
```

Consulto los últimos 20 mensajes del registro del contenedor:

```bash
sudo docker logs mssql-server --tail 20
```

**Evidencia (imagen):**

![Evidencia del levantamiento de MS SQL Server](image-52.png)

**Registro:**

Ingresé a la carpeta de MS SQL Server y levanté el contenedor `mssql-server` en segundo plano con `docker compose up -d`. Después verifiqué que el contenedor estuviera ejecutándose con `docker ps` y consulté los últimos 20 mensajes mediante `docker logs` para comprobar el estado del servicio.

### 3.5 Instalar `mssql-tools` en WSL

Estos pasos instalan `mssql-tools18` y `unixODBC` en Ubuntu 24.04 para administrar y conectarse a SQL Server ejecutado mediante Docker.

#### 3.5.1 Actualizar e instalar los paquetes requeridos

```bash
sudo apt update && sudo apt install -y curl ca-certificates gnupg
```

#### 3.5.2 Eliminar repositorios antiguos o duplicados de Microsoft

Esto evita conflictos con configuraciones anteriores de Ubuntu 22.04 (`jammy`) y con el método obsoleto `apt-key`.

```bash
sudo rm -f /etc/apt/sources.list.d/mssql-release.list
sudo rm -f /etc/apt/sources.list.d/microsoft-prod.list
```

#### 3.5.3 Descargar el repositorio oficial de Microsoft para Ubuntu 24.04

```bash
cd /tmp
curl -sSL -O https://packages.microsoft.com/config/ubuntu/24.04/packages-microsoft-prod.deb
```

#### 3.5.4 Instalar el repositorio oficial de Microsoft

```bash
sudo dpkg -i packages-microsoft-prod.deb
```

#### 3.5.5 Actualizar los repositorios

```bash
sudo apt update
```

#### 3.5.6 Verificar el repositorio de Microsoft

```bash
grep -R "packages.microsoft.com" /etc/apt/sources.list /etc/apt/sources.list.d/ 2>/dev/null
```

#### 3.5.7 Instalar Microsoft SQL Server Tools 18 y unixODBC

```bash
sudo ACCEPT_EULA=Y apt install -y mssql-tools18 unixodbc-dev
```

#### 3.5.8 Agregar `mssql-tools18` al PATH

```bash
echo 'export PATH="$PATH:/opt/mssql-tools18/bin"' >> ~/.bashrc
```

#### 3.5.9 Recargar la configuración de Bash

```bash
source ~/.bashrc
```

#### 3.5.10 Verificar que `sqlcmd` esté instalado

```bash
which sqlcmd
```

El resultado esperado es:

```text
/opt/mssql-tools18/bin/sqlcmd
```
si te parece un error metes esto o lo modificas depende de como tu lo tengas 

```bash 
sudo docker exec -it mssql-server /opt/mssql-tools18/bin/sqlcmd -S localhost -U SA -P 'CelebraHub_Centro_de_eventoseiner123!' -C
```
**Evidencia (imagen):**

![Evidencia de la instalación de mssql-tools en WSL](image-53.png)

![alt text](image-54.png)

![alt text](image-55.png)

**Registro:**

Seguí los pasos de instalación tal como aparecen en la página web del docente. En WSL instalé los paquetes requeridos, eliminé los repositorios antiguos de Microsoft y configuré el repositorio oficial para Ubuntu 24.04. Después instalé `mssql-tools18` y `unixODBC`, agregué la ruta de las herramientas al `PATH` y recargué la configuración de Bash. Finalmente verifiqué que `sqlcmd` quedó instalado correctamente en `/opt/mssql-tools18/bin/sqlcmd`.

### 3.6 Conectar localmente a SQL Server

Me conecto localmente al contenedor de SQL Server utilizando el usuario administrador `SA` y la contraseña configurada para el proyecto:

```bash
sudo docker exec -it mssql-server /opt/mssql-tools18/bin/sqlcmd -S localhost -U SA -P 'CelebraHub_Centro_de_eventoseiner123!' -C
```

### 3.6.1 Comandos básicos de referencia

Estos comandos son únicamente de referencia y no los utilizo para este caso durante la instalación ni durante la creación de las tablas de `CelebraHub_Centro_de_eventos`.

**Crear una base de datos**

```sql
CREATE DATABASE nombre_bd;
GO
```

**Listar todas las bases de datos**

```sql
SELECT name FROM sys.databases;
GO
```

**Eliminar una base de datos**

No puedo eliminar la base de datos que estoy utilizando. Cambio a `master` y luego ejecuto:

```sql
USE master;
GO
DROP DATABASE nombre_bd;
GO
```

**Entrar a una base de datos**

```sql
USE nombre_bd;
GO
```

**Listar las tablas**

```sql
SELECT name FROM sys.tables;
GO
```

**Listar los campos de una tabla**

```sql
SELECT COLUMN_NAME, DATA_TYPE, CHARACTER_MAXIMUM_LENGTH
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'nombre_tabla';
GO
```

**Salir del motor**

```sql
QUIT
```

### 3.6.2 Crear las tablas del proyecto final

Primero creo la base de datos `CelebraHub_Centro_de_eventos`, porque las tablas deben guardarse dentro de una base existente. Este paso se realiza desde `master` y solo es necesario una vez:

```bash
sudo docker exec -it mssql-server /opt/mssql-tools18/bin/sqlcmd -S localhost -U SA -P 'CelebraHub_Centro_de_eventoseiner123!' -C -d master
```

Cuando aparezca `1>`, escribo:

```sql
CREATE DATABASE CelebraHub_Centro_de_eventos;
```

Después presiono Enter, escribo manualmente `GO` y presiono Enter. `GO` no se pega junto con el código SQL; se escribe aparte para ejecutar la instrucción.

La base de datos `CelebraHub_Centro_de_eventos` ya fue creada. Para facilitar el proceso y evitar errores al pegar muchas líneas directamente en `sqlcmd`, guardo el código de las 17 tablas en un archivo `.sql` y lo ejecuto desde Bash.

Creo y abro el archivo SQL:

```bash
nano ~/CelebraHub_Centro_de_eventos.sql
```

Dentro del archivo pego el código de las 17 tablas que aparece a continuación. Guardo con `Ctrl + O`, presiono Enter y salgo con `Ctrl + X`.

Utilizo espacios reales para la indentación del código, no tabulaciones. Después de salir de `nano`, ejecuto el archivo contra la base de datos `CelebraHub_Centro_de_eventos`:

```bash
sudo docker exec -i mssql-server /opt/mssql-tools18/bin/sqlcmd -S localhost -U SA -P 'CelebraHub_Centro_de_eventoseiner123!' -C -d CelebraHub_Centro_de_eventos < ~/CelebraHub_Centro_de_eventos.sql
```

El archivo debe contener únicamente las instrucciones SQL de creación de las tablas y cada instrucción debe terminar con `GO`.

Para verificar las tablas, ejecuto desde Bash el comando `sqlcmd` con la base `CelebraHub_Centro_de_eventos` y la opción `-Q`.

```sql
-- =====================================================================
-- 1. MODELO RBAC (features/auth)
-- =====================================================================
CREATE TABLE users (
  id        INT IDENTITY(1,1) PRIMARY KEY,
  username  VARCHAR(100) NOT NULL,
  email     VARCHAR(150) NOT NULL,
  password  VARCHAR(255) NOT NULL,
  is_active VARCHAR(8) NOT NULL DEFAULT 'ACTIVE',
  avatar    VARCHAR(255) NULL,
  CONSTRAINT uq_users_username UNIQUE (username),
  CONSTRAINT uq_users_email    UNIQUE (email),
  CONSTRAINT ck_users_active   CHECK (is_active IN ('ACTIVE','INACTIVE'))
);
GO

CREATE TABLE roles (
  id        INT IDENTITY(1,1) PRIMARY KEY,
  name      VARCHAR(100) NOT NULL,
  is_active VARCHAR(8) NOT NULL DEFAULT 'ACTIVE',
  CONSTRAINT uq_roles_name   UNIQUE (name),
  CONSTRAINT ck_roles_active CHECK (is_active IN ('ACTIVE','INACTIVE'))
);
GO

CREATE TABLE role_users (
  id        INT IDENTITY(1,1) PRIMARY KEY,
  role_id   INT NOT NULL,
  user_id   INT NOT NULL,
  is_active VARCHAR(8) NOT NULL DEFAULT 'ACTIVE',
  CONSTRAINT fk_role_users_role FOREIGN KEY (role_id) REFERENCES roles(id),
  CONSTRAINT fk_role_users_user FOREIGN KEY (user_id) REFERENCES users(id),
  CONSTRAINT uq_role_users UNIQUE (role_id, user_id),
  CONSTRAINT ck_role_users_active CHECK (is_active IN ('ACTIVE','INACTIVE'))
);
GO

CREATE TABLE resources (
  id        INT IDENTITY(1,1) PRIMARY KEY,
  path      VARCHAR(255) NOT NULL,
  method    VARCHAR(10)  NOT NULL,
  is_active VARCHAR(8) NOT NULL DEFAULT 'ACTIVE',
  CONSTRAINT uq_resources_path_method UNIQUE (path, method),
  CONSTRAINT ck_resources_active CHECK (is_active IN ('ACTIVE','INACTIVE'))
);
GO

CREATE TABLE resource_roles (
  id          INT IDENTITY(1,1) PRIMARY KEY,
  resource_id INT NOT NULL,
  role_id     INT NOT NULL,
  is_active   VARCHAR(8) NOT NULL DEFAULT 'ACTIVE',
  CONSTRAINT fk_resource_roles_resource FOREIGN KEY (resource_id) REFERENCES resources(id),
  CONSTRAINT fk_resource_roles_role     FOREIGN KEY (role_id)     REFERENCES roles(id),
  CONSTRAINT uq_resource_roles UNIQUE (resource_id, role_id),
  CONSTRAINT ck_resource_roles_active CHECK (is_active IN ('ACTIVE','INACTIVE'))
);
GO

CREATE TABLE refresh_tokens (
  id          INT IDENTITY(1,1) PRIMARY KEY,
  user_id     INT NOT NULL,
  token       VARCHAR(500) NOT NULL,
  device_info VARCHAR(255) NOT NULL,
  is_valid    VARCHAR(8) NOT NULL DEFAULT 'ACTIVE',
  expires_at  DATETIME2 NOT NULL,
  created_at  DATETIME2 NOT NULL DEFAULT SYSDATETIME(),
  updated_at  DATETIME2 NOT NULL DEFAULT SYSDATETIME(),
  CONSTRAINT fk_refresh_tokens_user FOREIGN KEY (user_id) REFERENCES users(id),
  CONSTRAINT ck_refresh_tokens_valid CHECK (is_valid IN ('ACTIVE','INACTIVE'))
);
GO
CREATE INDEX idx_refresh_tokens_user ON refresh_tokens (user_id);
GO
CREATE INDEX idx_role_users_user ON role_users (user_id);
GO
CREATE INDEX idx_resource_roles_role ON resource_roles (role_id);
GO

-- =====================================================================
-- 2. MODELO DE NEGOCIO
-- =====================================================================
CREATE TABLE clientes (
  id               INT IDENTITY(1,1) PRIMARY KEY,
  tipo_documento   VARCHAR(10)  NOT NULL,
  numero_documento VARCHAR(30)  NOT NULL,
  nombre           VARCHAR(150) NOT NULL,
  telefono         VARCHAR(30)  NULL,
  email            VARCHAR(150) NULL,
  is_active        VARCHAR(8) NOT NULL DEFAULT 'ACTIVE',
  CONSTRAINT uq_clientes_documento UNIQUE (numero_documento),
  CONSTRAINT ck_clientes_active CHECK (is_active IN ('ACTIVE','INACTIVE'))
);
GO

CREATE TABLE salones (
  id          INT IDENTITY(1,1) PRIMARY KEY,
  nombre      VARCHAR(120) NOT NULL,
  descripcion VARCHAR(500) NULL,
  capacidad   INT NOT NULL DEFAULT 1,
  precio_base DECIMAL(14,2) NOT NULL DEFAULT 0,
  is_active   VARCHAR(8) NOT NULL DEFAULT 'ACTIVE',
  created_at  DATETIME2 NOT NULL DEFAULT SYSDATETIME(),
  updated_at  DATETIME2 NOT NULL DEFAULT SYSDATETIME(),
  CONSTRAINT uq_salones_nombre UNIQUE (nombre),
  CONSTRAINT ck_salones_capacidad CHECK (capacidad > 0),
  CONSTRAINT ck_salones_precio CHECK (precio_base >= 0),
  CONSTRAINT ck_salones_active CHECK (is_active IN ('ACTIVE','INACTIVE'))
);
GO

CREATE TABLE servicios (
  id          INT IDENTITY(1,1) PRIMARY KEY,
  nombre      VARCHAR(120) NOT NULL,
  descripcion VARCHAR(500) NULL,
  precio_base DECIMAL(14,2) NOT NULL DEFAULT 0,
  is_active   VARCHAR(8) NOT NULL DEFAULT 'ACTIVE',
  created_at  DATETIME2 NOT NULL DEFAULT SYSDATETIME(),
  updated_at  DATETIME2 NOT NULL DEFAULT SYSDATETIME(),
  CONSTRAINT uq_servicios_nombre UNIQUE (nombre),
  CONSTRAINT ck_servicios_precio CHECK (precio_base >= 0),
  CONSTRAINT ck_servicios_active CHECK (is_active IN ('ACTIVE','INACTIVE'))
);
GO

CREATE TABLE proveedores (
  id           INT IDENTITY(1,1) PRIMARY KEY,
  nit          VARCHAR(30)  NOT NULL,
  razon_social VARCHAR(150) NOT NULL,
  contacto     VARCHAR(120) NULL,
  telefono     VARCHAR(30)  NULL,
  email        VARCHAR(150) NULL,
  is_active    VARCHAR(8) NOT NULL DEFAULT 'ACTIVE',
  CONSTRAINT uq_proveedores_nit UNIQUE (nit),
  CONSTRAINT ck_proveedores_active CHECK (is_active IN ('ACTIVE','INACTIVE'))
);
GO

CREATE TABLE reservas (
  id                INT IDENTITY(1,1) PRIMARY KEY,
  cliente_id        INT NOT NULL,
  salon_id          INT NOT NULL,
  fecha_inicio      DATETIME2 NOT NULL,
  fecha_fin         DATETIME2 NOT NULL,
  estado            VARCHAR(15) NOT NULL DEFAULT 'PRE_RESERVA',
  bloqueo_expira_at DATETIME2 NULL,
  observaciones     VARCHAR(500) NULL,
  CONSTRAINT fk_reservas_cliente FOREIGN KEY (cliente_id) REFERENCES clientes(id),
  CONSTRAINT fk_reservas_salon   FOREIGN KEY (salon_id)   REFERENCES salones(id),
  CONSTRAINT ck_reservas_fechas CHECK (fecha_fin > fecha_inicio),
  CONSTRAINT ck_reservas_estado CHECK (estado IN ('PRE_RESERVA','CONFIRMADA','CANCELADA','FINALIZADA'))
);
GO
CREATE INDEX idx_reservas_disponibilidad ON reservas (salon_id, fecha_inicio, fecha_fin);
GO
CREATE INDEX idx_reservas_cliente ON reservas (cliente_id);
GO

CREATE TABLE eventos (
  id            INT IDENTITY(1,1) PRIMARY KEY,
  reserva_id    INT NOT NULL,                -- referencia_id (FK) hacia reservas
  tipo          VARCHAR(60) NOT NULL,
  fecha         DATETIME2 NOT NULL,
  cantidad      INT NOT NULL,                -- numero de asistentes
  observaciones VARCHAR(500) NULL,
  estado        VARCHAR(15) NOT NULL DEFAULT 'PLANEADO',
  CONSTRAINT fk_eventos_reserva FOREIGN KEY (reserva_id) REFERENCES reservas(id),
  CONSTRAINT uq_eventos_reserva UNIQUE (reserva_id),   -- Reserva 0..1 : 1 Evento
  CONSTRAINT ck_eventos_cantidad CHECK (cantidad > 0),
  CONSTRAINT ck_eventos_estado CHECK (estado IN ('PLANEADO','EN_CURSO','REALIZADO','CANCELADO'))
);
GO

CREATE TABLE evento_servicios (
  id              INT IDENTITY(1,1) PRIMARY KEY,
  evento_id       INT NOT NULL,
  servicio_id     INT NOT NULL,
  proveedor_id    INT NOT NULL,
  tipo            VARCHAR(60) NOT NULL,
  fecha           DATETIME2 NOT NULL,
  cantidad        INT NOT NULL DEFAULT 1,
  precio_unitario DECIMAL(14,2) NOT NULL DEFAULT 0,
  observaciones   VARCHAR(500) NULL,
  estado          VARCHAR(15) NOT NULL DEFAULT 'PENDIENTE',
  CONSTRAINT fk_evser_evento    FOREIGN KEY (evento_id)    REFERENCES eventos(id),
  CONSTRAINT fk_evser_servicio  FOREIGN KEY (servicio_id)  REFERENCES servicios(id),
  CONSTRAINT fk_evser_proveedor FOREIGN KEY (proveedor_id) REFERENCES proveedores(id),
  CONSTRAINT ck_evser_cantidad CHECK (cantidad > 0),
  CONSTRAINT ck_evser_estado CHECK (estado IN ('PENDIENTE','CONFIRMADO','CANCELADO'))
);
GO
CREATE INDEX idx_evser_evento ON evento_servicios (evento_id);
GO
CREATE INDEX idx_evser_servicio ON evento_servicios (servicio_id);
GO
CREATE INDEX idx_evser_proveedor ON evento_servicios (proveedor_id);
GO

CREATE TABLE contratos (
  id                  INT IDENTITY(1,1) PRIMARY KEY,
  cliente_id          INT NOT NULL,
  evento_id           INT NOT NULL,
  numero              VARCHAR(30) NOT NULL,
  fecha_inicio        DATE NOT NULL,
  fecha_fin           DATE NOT NULL,
  valor               DECIMAL(14,2) NOT NULL,
  porcentaje_anticipo DECIMAL(5,2) NOT NULL DEFAULT 30.00,
  version_actual      INT NOT NULL DEFAULT 1,
  estado              VARCHAR(15) NOT NULL DEFAULT 'BORRADOR',
  CONSTRAINT fk_contratos_cliente FOREIGN KEY (cliente_id) REFERENCES clientes(id),
  CONSTRAINT fk_contratos_evento  FOREIGN KEY (evento_id)  REFERENCES eventos(id),
  CONSTRAINT uq_contratos_numero UNIQUE (numero),
  CONSTRAINT uq_contratos_evento UNIQUE (evento_id),     -- Evento 1 : 1 Contrato
  CONSTRAINT ck_contratos_fechas CHECK (fecha_fin >= fecha_inicio),
  CONSTRAINT ck_contratos_valor  CHECK (valor >= 0),
  CONSTRAINT ck_contratos_anticipo CHECK (porcentaje_anticipo BETWEEN 0 AND 100),
  CONSTRAINT ck_contratos_estado CHECK (estado IN ('BORRADOR','VIGENTE','MODIFICADO','CANCELADO','FINALIZADO'))
);
GO
CREATE INDEX idx_contratos_cliente ON contratos (cliente_id);
GO

CREATE TABLE pagos (
  id              INT IDENTITY(1,1) PRIMARY KEY,
  contrato_id     INT NOT NULL,
  referencia_tipo VARCHAR(30) NOT NULL DEFAULT 'CONTRATO',
  referencia_id   INT NOT NULL,
  tipo_pago       VARCHAR(15) NOT NULL DEFAULT 'ABONO',
  metodo          VARCHAR(30) NOT NULL,
  monto           DECIMAL(14,2) NOT NULL,
  fecha           DATETIME2 NOT NULL DEFAULT SYSDATETIME(),
  estado          VARCHAR(15) NOT NULL DEFAULT 'PENDIENTE',
  CONSTRAINT fk_pagos_contrato FOREIGN KEY (contrato_id) REFERENCES contratos(id),
  CONSTRAINT ck_pagos_monto CHECK (monto > 0),
  CONSTRAINT ck_pagos_tipo CHECK (tipo_pago IN ('ANTICIPO','ABONO','SALDO','PENALIDAD')),
  CONSTRAINT ck_pagos_estado CHECK (estado IN ('PENDIENTE','APLICADO','ANULADO'))
);
GO
CREATE INDEX idx_pagos_contrato ON pagos (contrato_id);
GO
CREATE INDEX idx_pagos_referencia ON pagos (referencia_tipo, referencia_id);
GO

CREATE TABLE cambios_contrato (
  id             INT IDENTITY(1,1) PRIMARY KEY,
  contrato_id    INT NOT NULL,
  version        INT NOT NULL,
  nombre         VARCHAR(120) NOT NULL,
  descripcion    VARCHAR(500) NULL,
  valor_anterior DECIMAL(14,2) NOT NULL,
  valor_nuevo    DECIMAL(14,2) NOT NULL,
  estado         VARCHAR(15) NOT NULL DEFAULT 'SOLICITADO',
  is_active      VARCHAR(8) NOT NULL DEFAULT 'ACTIVE',
  created_at     DATETIME2 NOT NULL DEFAULT SYSDATETIME(),
  updated_at     DATETIME2 NOT NULL DEFAULT SYSDATETIME(),
  CONSTRAINT fk_cambios_contrato FOREIGN KEY (contrato_id) REFERENCES contratos(id),
  CONSTRAINT uq_cambios_version UNIQUE (contrato_id, version),
  CONSTRAINT ck_cambios_estado CHECK (estado IN ('SOLICITADO','APROBADO','RECHAZADO')),
  CONSTRAINT ck_cambios_active CHECK (is_active IN ('ACTIVE','INACTIVE'))
);
GO

CREATE TABLE cancelaciones (
  id                   INT IDENTITY(1,1) PRIMARY KEY,
  contrato_id          INT NOT NULL,
  nombre               VARCHAR(120) NOT NULL,
  descripcion          VARCHAR(500) NULL,
  porcentaje_penalidad DECIMAL(5,2) NOT NULL DEFAULT 0,
  valor_penalidad      DECIMAL(14,2) NOT NULL DEFAULT 0,
  estado               VARCHAR(15) NOT NULL DEFAULT 'SOLICITADA',
  is_active            VARCHAR(8) NOT NULL DEFAULT 'ACTIVE',
  created_at           DATETIME2 NOT NULL DEFAULT SYSDATETIME(),
  updated_at           DATETIME2 NOT NULL DEFAULT SYSDATETIME(),
  CONSTRAINT fk_cancelaciones_contrato FOREIGN KEY (contrato_id) REFERENCES contratos(id),
  CONSTRAINT ck_cancel_porcentaje CHECK (porcentaje_penalidad BETWEEN 0 AND 100),
  CONSTRAINT ck_cancel_estado CHECK (estado IN ('SOLICITADA','APLICADA','ANULADA')),
  CONSTRAINT ck_cancel_active CHECK (is_active IN ('ACTIVE','INACTIVE'))
);
GO
CREATE INDEX idx_cancelaciones_contrato ON cancelaciones (contrato_id);
GO

-- =====================================================================
-- 3. DATOS SEMILLA (RBAC)
-- =====================================================================
INSERT INTO roles (name) VALUES
  ('ADMIN'), ('COMERCIAL'), ('OPERACIONES'), ('PROVEEDOR'), ('CARTERA');
GO

INSERT INTO resources (path, method) VALUES
  ('/reservas', 'POST'),
  ('/contratos', 'POST'),
  ('/cambios-contrato', 'POST'),
  ('/cancelaciones', 'POST');
GO

-- ADMIN accede a todos los recursos
INSERT INTO resource_roles (resource_id, role_id)
SELECT r.id, ro.id FROM resources r CROSS JOIN roles ro WHERE ro.name = 'ADMIN';
GO

-- COMERCIAL: reservas y contratos | OPERACIONES: reservas y cambios | CARTERA: contratos y cancelaciones
INSERT INTO resource_roles (resource_id, role_id)
SELECT r.id, ro.id FROM resources r JOIN roles ro
  ON (ro.name = 'COMERCIAL'   AND r.path IN ('/reservas','/contratos'))
  OR (ro.name = 'OPERACIONES' AND r.path IN ('/reservas','/cambios-contrato'))
  OR (ro.name = 'CARTERA'     AND r.path IN ('/contratos','/cancelaciones'));
GO

-- Usuario administrador de prueba. IMPORTANTE: reemplazar el hash por uno generado con bcrypt desde tu backend.
INSERT INTO users (username, email, password) VALUES
  ('admin', 'admin@celebrahub.com', '$2b$10$REEMPLAZAR_POR_HASH_BCRYPT_GENERADO_EN_EL_BACKEND');
GO

INSERT INTO role_users (role_id, user_id)
SELECT ro.id, u.id FROM roles ro, users u WHERE ro.name = 'ADMIN' AND u.username = 'admin';
GO

-- =====================================================================
-- 4. DATOS SEMILLA (NEGOCIO)
-- =====================================================================
INSERT INTO clientes (tipo_documento, numero_documento, nombre, telefono, email) VALUES
  ('CC', '1001234567', 'Laura Martinez', '3001112233', 'laura@mail.com'),
  ('NIT', '900123456-1', 'Empresa Andina SAS', '6015556677', 'eventos@andina.com');
GO

INSERT INTO salones (nombre, descripcion, capacidad, precio_base) VALUES
  ('Salon Perla', 'Salon principal con terraza', 250, 4500000),
  ('Salon Coral', 'Salon mediano para eventos corporativos', 120, 2500000),
  ('Salon Brisa', 'Salon intimo para celebraciones pequenas', 60, 1500000);
GO

INSERT INTO servicios (nombre, descripcion, precio_base) VALUES
  ('Catering', 'Alimentos y bebidas por persona', 85000),
  ('Mobiliario', 'Mesas, sillas y manteleria', 1200000),
  ('Decoracion', 'Ambientacion tematica', 1800000),
  ('Sonido e iluminacion', 'DJ, sonido e iluminacion', 2000000);
GO

INSERT INTO proveedores (nit, razon_social, contacto, telefono, email) VALUES
  ('800111222-3', 'Sabores del Caribe SAS', 'Carlos Perez', '3109998877', 'ventas@sabores.com'),
  ('800333444-5', 'Eventos Sonido Pro', 'Ana Rojas', '3125554433', 'contacto@sonidopro.com');
GO
```

Verifico que las tablas del proyecto se hayan creado correctamente:

```bash
sudo docker exec -it mssql-server /opt/mssql-tools18/bin/sqlcmd -S localhost -U SA -P 'CelebraHub_Centro_de_eventoseiner123!' -C -d CelebraHub_Centro_de_eventos -Q "SELECT name FROM sys.tables;"
```

**Evidencia (imagen):**

![alt text](image-56.png)

**Registro:**

Creé la base de datos `CelebraHub_Centro_de_eventos` y ejecuté el archivo `CelebraHub_Centro_de_eventos.sql` con las 17 tablas del proyecto final. Utilicé espacios reales en el código para evitar errores al copiar y pegar, ejecuté el archivo directamente desde Bash y verifiqué en SQL Server que las tablas se crearon correctamente.

### 3.7 Crear un usuario propio con acceso remoto

Me conecto primero como `SA` al motor de SQL Server:

```bash
sudo docker exec -it mssql-server /opt/mssql-tools18/bin/sqlcmd -S localhost -U SA -P 'CelebraHub_Centro_de_eventoseiner123!' -C -d master
```

**Código original de la guía del docente:**

Reemplaza `usuario`, `password` y `nombre_bd` por los valores correspondientes:

```sql
CREATE DATABASE nombre_bd;
GO

CREATE LOGIN usuario WITH PASSWORD = 'password';
GO

USE nombre_bd;
GO
CREATE USER usuario FOR LOGIN usuario;
GO

ALTER ROLE db_owner ADD MEMBER usuario;
GO
```

**Código que colocaré en mi caso:**

La base de datos `CelebraHub_Centro_de_eventos` ya existe y contiene las tablas del proyecto. Creo el login `einer_coa`, le asigno el rol `sysadmin` y lo habilito para permitir la administración remota. Ejecuto cada comando por separado:

```sql
CREATE LOGIN einer_coa WITH PASSWORD = 'einer123', CHECK_POLICY = OFF;
```

Después de presionar Enter, escribo manualmente `GO` y presiono Enter.

```sql
ALTER SERVER ROLE sysadmin ADD MEMBER einer_coa;
```

Después de presionar Enter, escribo manualmente `GO` y presiono Enter.

```sql
ALTER LOGIN einer_coa ENABLE;
```

Después de presionar Enter, escribo manualmente `GO` y presiono Enter.

`CHECK_POLICY = OFF` desactiva la política de complejidad únicamente para este login; por eso se puede utilizar la contraseña `einer123`. Esta opción no cambia los requisitos de la contraseña de `SA`, que continúa utilizando `CelebraHub_Centro_de_eventoseiner123!` en el archivo `.env`.

**Evidencia (imagen):**

![alt text](image-57.png)
**Registro:**

Al comenzar nuevamente, SQL Server indicó que el login `einer_coa` ya existía. Después ejecuté por separado `ALTER SERVER ROLE sysadmin ADD MEMBER einer_coa;` y `ALTER LOGIN einer_coa ENABLE;`, escribiendo `GO` manualmente después de cada comando. El usuario `einer_coa` quedó habilitado y con permisos de `sysadmin` para administrar remotamente el proyecto `CelebraHub_Centro_de_eventos`.

### 3.8 Conectar remotamente desde cualquier equipo

##### **Hacerlo desde DBeaver**

Para conectarme de forma remota a SQL Server utilizo DBeaver. Necesito los siguientes datos:

Obtengo los datos de conexión de la configuración realizada anteriormente:

- La dirección IP del servidor la consulto con `ip a`, buscando la dirección de la interfaz `eth0`.
- El puerto `1433` lo obtengo del archivo `docker-compose.yml`, donde está publicado como `0.0.0.0:1433:1433`, y de la regla de UFW `1433/tcp`.
- La base de datos `CelebraHub_Centro_de_eventos` corresponde al nombre utilizado al crear las tablas del proyecto.
- El usuario `einer_coa` y la contraseña `einer123` corresponden al login creado en la sección 3.7.

Consulto la dirección IP del servidor:

```bash
ip a
```

Consulto la edición y la contraseña de `SA` configuradas en el archivo `.env`:

```bash
grep -E '^(MSSQL_PID|MSSQL_SA_PASSWORD)=' ~/ia-lab/services/motores-bd/mssql/.env
```

- **Motor:** SQL Server.
- **Servidor:** `172.18.50.255`.
- **Puerto:** `1433`.
- **Base de datos:** `CelebraHub_Centro_de_eventos`.
- **Usuario:** `einer_coa`.
- **Contraseña:** `einer123`.

Antes de abrir DBeaver, compruebo desde WSL que el contenedor esté activo y que el puerto esté publicado:

```bash
sudo docker ps --filter "name=mssql-server" --format "table {{.Names}}\t{{.Ports}}"
```

También compruebo que SQL Server acepte conexiones:

```bash
sudo docker exec mssql-server /opt/mssql-tools18/bin/sqlcmd -S localhost -U SA -P 'CelebraHub_Centro_de_eventoseiner123!' -C -Q "SELECT 1;"
```

**Evidencia (imagen):**

![alt text](image-58.png)

**Registro:**

Comprobé la dirección IP del servidor, el puerto `1433`, el estado del contenedor `mssql-server` y la respuesta de SQL Server antes de configurar la conexión remota en DBeaver. Estos datos me permitieron confirmar que el servidor estaba listo para recibir la conexión.

En DBeaver realizo los siguientes pasos:

1. Abro DBeaver.
2. Selecciono **Nueva conexión**.
3. Elijo el controlador **SQL Server**.
4. Escribo `172.18.50.255` en **Host**.
5. Escribo `1433` en **Port**.
6. Escribo `einer_coa` en **Username**.
7. Escribo `einer123` en **Password**.
8. Escribo `CelebraHub_Centro_de_eventos` en **Database**.
9. Presiono **Test Connection**.
10. Si la prueba es correcta, presiono **Finish** para guardar la conexión.

**Evidencia (imagen):**

![alt text](image-59.png)


![alt text](image-60.png)


**Registro:**

Configuré y probé en DBeaver la conexión remota a SQL Server utilizando el host `172.18.50.255`, el puerto `1433`, la base de datos `CelebraHub_Centro_de_eventos` y el usuario `einer_coa` con la contraseña `einer123`.

### 3.9 Backup de una base de datos

**Código original de la guía del docente:**

```bash
sudo docker exec mssql-server /opt/mssql-tools18/bin/sqlcmd -S localhost -U SA -P 'MiNiCo57**' -C -Q "BACKUP DATABASE [tecnogua] TO DISK = N'/backups/backup_tecnogua.bak' WITH INIT"
```

**Código que colocaré en mi caso:**

En mi proyecto utilizo la base de datos `CelebraHub_Centro_de_eventos`, la contraseña `CelebraHub_Centro_de_eventoseiner123!` y guardo el backup en la carpeta `/backups`, que está conectada con `/mnt/d/academia/bd/`:

```bash
sudo docker exec mssql-server /opt/mssql-tools18/bin/sqlcmd -S localhost -U SA -P 'CelebraHub_Centro_de_eventoseiner123!' -C -Q "BACKUP DATABASE [CelebraHub_Centro_de_eventos] TO DISK = N'/backups/backup_CelebraHub_Centro_de_eventos.bak' WITH INIT"
```

Verifico desde WSL que el archivo se haya creado correctamente:

```bash
ls -lh /mnt/d/academia/bd/backup_CelebraHub_Centro_de_eventos.bak
```

**Evidencia (imagen):**

![alt text](image-61.png)


**Registro:**

Realicé el backup de la base de datos `CelebraHub_Centro_de_eventos` utilizando el usuario `SA` y lo guardé como `backup_CelebraHub_Centro_de_eventos.bak` dentro de `/backups`. Después verifiqué desde WSL que el archivo se creó correctamente en `/mnt/d/academia/bd/`.

## 4. Oracle XE

### 4.1 Crear el archivo `docker-compose.yml`

```bash
cat > ~/ia-lab/services/motores-bd/oracle/docker-compose.yml << 'EOF'
services:
  oracle:
    image: gvenzl/oracle-xe:21-slim
    container_name: oracle-server
    restart: unless-stopped

    env_file:
      - .env

    ports:
      - "0.0.0.0:1521:1521"

    volumes:
      - ../../../data/oracle:/opt/oracle/oradata
      - /mnt/d/academia/bd:/backups

    networks:
      - ia-lab-network

    healthcheck:
      test: ["CMD", "healthcheck.sh"]
      interval: 10s
      timeout: 5s
      retries: 10
      start_period: 120s

networks:
  ia-lab-network:
    external: true
EOF
```

Habilito UFW y permito el puerto `1521` de Oracle en el host:

```bash
sudo ufw allow 1521/tcp
sudo ufw enable
sudo ufw status
```

El listener de Oracle XE en `gvenzl/oracle-xe` ya acepta conexiones remotas, equivalente a `--bind-address=0.0.0.0` en MySQL y `listen_addresses=*` en PostgreSQL. No utilizo `command:` aquí, porque en esta imagen sustituiría el entrypoint y el contenedor no arrancaría.

Los usuarios `SYSTEM` y `einer_coa` no se atan a un host, ya que en Oracle no existe `@'%'` como en MySQL. Con el puerto `1521` publicado y UFW abierto, DBeaver puede entrar desde otro equipo utilizando el **Service Name** `CelebraHub_Centro_de_eventos`.

**Evidencia (imagen):**

![Evidencia de la creación y configuración de Oracle XE](image-62.png)

### 4.2 Crear el archivo `.env`

Creo el archivo `.env` para configurar la zona horaria, la contraseña de Oracle XE y el nombre del PDB de mi proyecto: `CelebraHub_Centro_de_eventos` con la contraseña `einer123`.

```bash
cat > ~/ia-lab/services/motores-bd/oracle/.env << 'EOF'
TZ=America/Bogota
ORACLE_PASSWORD=einer123
ORACLE_DATABASE=CelebraHub_Centro_de_eventos
EOF
```

Para ver lo que guardé dentro del archivo `.env`, ejecuto:

```bash
cat ~/ia-lab/services/motores-bd/oracle/.env
```

`ORACLE_DATABASE` **no es un usuario**. Es el nombre del **PDB** (pluggable database) que crea el contenedor: `CelebraHub_Centro_de_eventos`.

El usuario administrador que crea la imagen se llama **`SYSTEM`**; también existe `SYS`. Al conectarme mediante `sqlplus` o DBeaver utilizo `SYSTEM`, no `CelebraHub_Centro_de_eventos`.

| Variable | Qué es | Valor |
| --- | --- | --- |
| `ORACLE_DATABASE` | PDB / servicio de conexión | `CelebraHub_Centro_de_eventos` |
| `ORACLE_PASSWORD` | Contraseña de `SYSTEM` y `SYS` | `einer123` |

Para editar la clave del archivo `.env` en clase, ingreso a la carpeta de Oracle y abro el archivo:

```bash
cd ~/ia-lab/services/motores-bd/oracle
sudo nano .env
```

Cambio la contraseña por:

```env
ORACLE_PASSWORD=einer123
```

Guardo los cambios con `Ctrl + O` y salgo del editor con `Ctrl + X`.

El primer arranque de Oracle puede tardar varios minutos. La contraseña `einer123` puede ser rechazada por Oracle XE porque el motor exige una contraseña compleja. Si esto ocurre, debo utilizar una contraseña que incluya mayúsculas, minúsculas, números y símbolos.

**Evidencia (imagen):**

![alt text](image-63.png)

**Registro:**

Creé el archivo `.env` de Oracle XE con la zona horaria `America/Bogota`, la contraseña `einer123` y el PDB `CelebraHub_Centro_de_eventos`, que son los datos que estoy utilizando actualmente. También dejé documentado que el usuario administrador para conectarme es `SYSTEM` y que `CelebraHub_Centro_de_eventos` corresponde al nombre del PDB o servicio de conexión.

### 4.3 Crear `README.md`

Creo el archivo `README.md` para documentar la configuración de Oracle XE:

````bash
cat > ~/ia-lab/services/motores-bd/oracle/README.md << 'EOF'
# Oracle XE - Motor de Base de Datos

> **Acceso remoto habilitado.** Puerto expuesto en `0.0.0.0:1521`.
> **Usuario por defecto:** `SYSTEM`
> **PDB / Service Name:** `CelebraHub_Centro_de_eventos`
> **Password:** `einer123`

---

## Conectar desde WSL (local)

```bash
sudo docker exec -it oracle-server sqlplus 'system/einer123@//localhost:1521/CelebraHub_Centro_de_eventos'
```
EOF
````

Para comprobar que el archivo se creó correctamente, ejecuto:

```bash
cat ~/ia-lab/services/motores-bd/oracle/README.md
```

**Evidencia (imagen):**

![alt text](image-64.png)

**Registro:**

Creé el archivo `README.md` de Oracle XE utilizando el comando `cat` y después comprobé su contenido con otro comando `cat`. En el archivo documenté el puerto `1521`, el usuario `SYSTEM`, el PDB `CelebraHub_Centro_de_eventos`, la contraseña `einer123` y el comando para conectarme localmente desde WSL. Finalmente ejecuté `date` para registrar la fecha de la actividad.

### 4.4 Levantar Oracle

Ingreso a la carpeta de Oracle y levanto el contenedor en segundo plano:

```bash
cd ~/ia-lab/services/motores-bd/oracle
sudo docker compose up -d
```

Verifico si el contenedor `oracle-server` está corriendo y reviso sus últimos 30 mensajes:

```bash
sudo docker ps | grep oracle-server
sudo docker logs oracle-server --tail 30
```

**Evidencia del error (imagen):**

![alt text](image-65.png)

**Registro:**

Ingresé a la carpeta de Oracle XE y ejecuté `sudo docker compose up -d`. La imagen `gvenzl/oracle-xe:21-slim` se descargó correctamente y el contenedor `oracle-server` se inició. Sin embargo, al verificarlo con `sudo docker ps`, observé que aparecía como `Restarting (2)`. Al consultar los registros con `sudo docker logs oracle-server --tail 30`, encontré el error `Cannot create folder` y varios mensajes que indicaban que Oracle no podía crear ni abrir los archivos dentro de `/opt/oracle/oradata/XE/XEPDB1`. El problema estaba relacionado con la carpeta de datos montada desde `~/ia-lab/data/oracle`.

#### 4.4.1 Revisar y corregir los permisos de los datos

Primero reviso los permisos y el contenido de la carpeta de datos:

```bash
ls -ld ~/ia-lab/data/oracle
sudo ls -la ~/ia-lab/data/oracle
```

```bash
sudo docker compose down
```

Cambio el propietario de la carpeta al usuario y grupo utilizados por Oracle dentro del contenedor:

```bash
sudo chown -R 54321:54321 ~/ia-lab/data/oracle
```

Después otorgo permisos de lectura y escritura:

```bash
sudo chmod -R 775 ~/ia-lab/data/oracle
```

Vuelvo a iniciar Oracle:

```bash
sudo docker compose up -d
```


#### 4.4.2 Verificar el inicio de Oracle

Compruebo nuevamente el estado del contenedor y el puerto publicado:

```bash
sudo docker ps | grep oracle-server
```

El resultado esperado es similar a:

```text
oracle-server   Up 10 seconds (health: starting)   0.0.0.0:1521->1521/tcp
```

También reviso los últimos 30 mensajes para comprobar el listener:

```bash
sudo docker logs oracle-server --tail 30
```

En los registros busco el mensaje:

```text
Listening on: (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=0.0.0.0)(PORT=1521)))
```

Este mensaje confirma que el listener de Oracle está escuchando en el puerto `1521` y que el contenedor ya está terminando su inicialización.

**Evidencia de la solución (imagen):**

![alt text](./evidencia/image-66.png)

**Registro final:**

Después de corregir los permisos, volví a iniciar Oracle XE. El contenedor `oracle-server` dejó de reiniciarse y apareció como `Up (healthy)`. También comprobé que el puerto `1521` estaba publicado y que el listener escuchaba en `0.0.0.0:1521`. En los registros confirmé que el PDB `CelebraHub_Centro_de_eventos` se abrió en modo lectura y escritura con el mensaje `Pluggable database CelebraHub_Centro_de_eventos opened read write`. Finalmente ejecuté `date` y registré la fecha de la actividad.

### 4.5 Conectar localmente a Oracle

Ingreso al contenedor de Oracle XE:

```bash
sudo docker exec -it oracle-server bash
```

Me conecto a Oracle utilizando el usuario administrador `SYSTEM`, la contraseña `einer123` y el PDB `CelebraHub_Centro_de_eventos`:

```bash
sqlplus 'system/einer123@//localhost:1521/CelebraHub_Centro_de_eventos'
```

Debo esperar a que aparezca el indicador `SQL>` antes de escribir comandos de Oracle. Entrar al contenedor solo muestra el aviso `bash-4.4$`; en ese punto todavía estoy en Bash y no en SQL*Plus.

Si escribo comandos como `SELECT`, `CONN`, `DESC` o `EXIT` mientras aparece `bash-4.4$`, Bash intenta interpretarlos como comandos de Linux y muestra un error. Para ejecutar esos comandos debo estar dentro de SQL*Plus, donde el aviso es `SQL>`.

Ya dentro del motor, creo el usuario `einer_coa`, que utilizaré como esquema del proyecto `CelebraHub_Centro_de_eventos`:

```sql
CREATE USER einer_coa IDENTIFIED BY "einer123" DEFAULT TABLESPACE USERS QUOTA UNLIMITED ON USERS;
ALTER USER einer_coa QUOTA UNLIMITED ON USERS;
GRANT CONNECT, RESOURCE TO einer_coa;
```

**Evidencia (imagen):**

![alt text](./evidencia/image-67.png)


**Registro:**

Ingresé al contenedor `oracle-server` y me conecté localmente al PDB `CelebraHub_Centro_de_eventos` con el usuario `SYSTEM`. Después creé el usuario `einer_coa`, le asigné una cuota ilimitada en el tablespace `USERS` y le otorgué los privilegios `CONNECT` y `RESOURCE`. Oracle confirmó las operaciones con los mensajes `User created`, `User altered` y `Grant succeeded`. Finalmente salí de SQL*Plus, salí del contenedor y ejecuté `date` para registrar la fecha de la actividad.

### 4.5.1 Comandos normales de Oracle

En Oracle el PDB `CelebraHub_Centro_de_eventos` es la base de datos y cada usuario funciona como un esquema. Por eso trabajo dentro del PDB `CelebraHub_Centro_de_eventos` y utilizo el usuario `einer_coa` para los objetos de mi proyecto.

**Listar todos los usuarios o esquemas**

```sql
SELECT username FROM all_users ORDER BY username;
```

**Eliminar el esquema del proyecto**

Si necesito eliminar el esquema `einer_coa`, me conecto como `SYS` o `SYSTEM` y ejecuto:

```sql
DROP USER einer_coa CASCADE;
```

No elimino el usuario `SYSTEM`, porque es el administrador que utilizo para conectarme al PDB `CelebraHub_Centro_de_eventos`.

**Entrar al esquema del proyecto**

Me conecto al usuario `einer_coa` dentro del PDB `CelebraHub_Centro_de_eventos`:

```sql
CONN einer_coa/einer123@//localhost:1521/CelebraHub_Centro_de_eventos
```

**Listar las tablas**

```sql
SELECT owner, table_name
FROM all_tables
WHERE owner = 'EINER_COA'
ORDER BY table_name;
```

**Listar los campos de una tabla**

```sql
DESC nombre_tabla;
```

También consulto los campos con:

```sql
SELECT column_name, data_type, data_length
FROM user_tab_columns
WHERE table_name = 'NOMBRE_TABLA';
```

**Salir del motor**

```sql
EXIT;
```

### 4.6 Crear un usuario propio con acceso remoto

Para crear y administrar mi usuario propio, primero me conecto como `SYSTEM` al PDB `CelebraHub_Centro_de_eventos`:

```bash
sudo docker exec -it oracle-server bash
sqlplus 'system/einer123@//localhost:1521/CelebraHub_Centro_de_eventos'
```

En mi caso, el usuario `einer_coa` ya fue creado en el paso 4.5 con la contraseña `einer123`, cuota ilimitada sobre `USERS` y los privilegios `CONNECT` y `RESOURCE`.

Oracle no utiliza usuarios asociados a un host mediante `@'%'`, como MySQL. Para permitir la conexión remota utilizo el puerto publicado `1521`, el listener activo y el Service Name `CelebraHub_Centro_de_eventos`.

Para conectarme desde otro equipo con DBeaver utilizo los siguientes datos:

- **Motor:** Oracle.
- **Host:** la dirección IP del equipo donde está ejecutándose Docker.
- **Port:** `1521`.
- **Service Name:** `CelebraHub_Centro_de_eventos`.
- **Username:** `einer_coa`.
- **Password:** `einer123`.

Me conecto al esquema `einer_coa` y consulto la información del proyecto:

```sql
CONN einer_coa/einer123@//localhost:1521/CelebraHub_Centro_de_eventos
SELECT username FROM all_users ORDER BY username;
SELECT owner, table_name FROM all_tables WHERE owner = 'EINER_COA' ORDER BY table_name;
DESC nombre_tabla;
EXIT;
```

**Evidencia del intento (imagen):**

![alt text](./evidencia/image-68.png)


**Registro:**

En este intento entré al contenedor, pero no llegué a SQL*Plus, porque no apareció el indicador `SQL>`. Por esa razón, Bash intentó interpretar `CONN`, `SELECT`, `DESC` y `EXIT` como comandos de Linux y mostró el mensaje `command not found`. Después salí del contenedor y ejecuté `sqlplus` directamente en Ubuntu, donde también apareció `sqlplus: command not found`.

#### Procedimiento correcto para repetirlo

Para hacerlo nuevamente, ejecuto los comandos paso a paso y espero el indicador correspondiente antes de continuar:

**Paso 1. Entro al contenedor:**

```bash
sudo docker exec -it oracle-server bash
```

Debo comprobar que aparezca `bash-4.4$`.

**Paso 2. Inicio SQL*Plus como `SYSTEM`:**

```bash
sqlplus 'system/einer123@//localhost:1521/CelebraHub_Centro_de_eventos'
```

Espero a que aparezca `SQL>`. Los siguientes comandos solo se ejecutan después de ver ese indicador.

**Paso 3. Me conecto al usuario `einer_coa`:**

```sql
CONN einer_coa/einer123@//localhost:1521/CelebraHub_Centro_de_eventos
```

**Paso 4. Consulto los usuarios o esquemas:**

```sql
SELECT username FROM all_users ORDER BY username;
```

**Paso 5. Consulto las tablas del usuario `einer_coa`:**

```sql
SELECT owner, table_name
FROM all_tables
WHERE owner = 'EINER_COA'
ORDER BY table_name;
```

**Registro del resultado:**

Después de iniciar SQL*Plus como `SYSTEM`, me conecté correctamente al usuario `einer_coa` dentro del PDB `CelebraHub_Centro_de_eventos`. La consulta de usuarios mostró 28 usuarios, incluido `EINER_COA`. Al consultar las tablas del usuario `einer_coa`, Oracle respondió `no rows selected`, lo que confirma que el esquema está creado correctamente, pero todavía no contiene tablas.

### 4.5.2 Crear las tablas del proyecto CelebraHub_Centro_de_eventos

La imagen corresponde a mi proyecto final **CelebraHub - Centro de eventos**. Como el esquema `einer_coa` todavía no tenía tablas, creo las 17 tablas del modelo dentro del PDB `CelebraHub_Centro_de_eventos`.

**Evidencia del modelo del proyecto (imagen):**

![alt text](./evidencia/image-20.png)

Antes de copiar el código de las tablas, entro al contenedor desde Ubuntu:

```bash
sudo docker exec -it oracle-server bash
```

Dentro del contenedor inicio SQL*Plus como `SYSTEM`:

```bash
sqlplus 'system/einer123@//localhost:1521/CelebraHub_Centro_de_eventos'
```

Cuando aparezca el indicador `SQL>`, me conecto al esquema `einer_coa`:

```sql
CONN einer_coa/einer123@//localhost:1521/CelebraHub_Centro_de_eventos
```

Después de que aparezca nuevamente `SQL>`, ejecuto las siguientes instrucciones para crear las tablas:

```sql
-- =====================================================================
-- 1. MODELO RBAC (features/auth)
-- =====================================================================
CREATE TABLE users (
  id        NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  username  VARCHAR2(100) NOT NULL,
  email     VARCHAR2(150) NOT NULL,
  password  VARCHAR2(255) NOT NULL,
  is_active VARCHAR2(8) DEFAULT 'ACTIVE' NOT NULL,
  avatar    VARCHAR2(255) NULL,
  CONSTRAINT uq_users_username UNIQUE (username),
  CONSTRAINT uq_users_email    UNIQUE (email),
  CONSTRAINT ck_users_active   CHECK (is_active IN ('ACTIVE','INACTIVE'))
);

CREATE TABLE roles (
  id        NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  name      VARCHAR2(100) NOT NULL,
  is_active VARCHAR2(8) DEFAULT 'ACTIVE' NOT NULL,
  CONSTRAINT uq_roles_name   UNIQUE (name),
  CONSTRAINT ck_roles_active CHECK (is_active IN ('ACTIVE','INACTIVE'))
);

CREATE TABLE role_users (
  id        NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  role_id   NUMBER NOT NULL,
  user_id   NUMBER NOT NULL,
  is_active VARCHAR2(8) DEFAULT 'ACTIVE' NOT NULL,
  CONSTRAINT fk_role_users_role FOREIGN KEY (role_id) REFERENCES roles(id),
  CONSTRAINT fk_role_users_user FOREIGN KEY (user_id) REFERENCES users(id),
  CONSTRAINT uq_role_users UNIQUE (role_id, user_id),
  CONSTRAINT ck_role_users_active CHECK (is_active IN ('ACTIVE','INACTIVE'))
);

CREATE TABLE resources (
  id        NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  path      VARCHAR2(255) NOT NULL,
  method    VARCHAR2(10)  NOT NULL,
  is_active VARCHAR2(8) DEFAULT 'ACTIVE' NOT NULL,
  CONSTRAINT uq_resources_path_method UNIQUE (path, method),
  CONSTRAINT ck_resources_active CHECK (is_active IN ('ACTIVE','INACTIVE'))
);

CREATE TABLE resource_roles (
  id          NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  resource_id NUMBER NOT NULL,
  role_id     NUMBER NOT NULL,
  is_active   VARCHAR2(8) DEFAULT 'ACTIVE' NOT NULL,
  CONSTRAINT fk_resource_roles_resource FOREIGN KEY (resource_id) REFERENCES resources(id),
  CONSTRAINT fk_resource_roles_role     FOREIGN KEY (role_id)     REFERENCES roles(id),
  CONSTRAINT uq_resource_roles UNIQUE (resource_id, role_id),
  CONSTRAINT ck_resource_roles_active CHECK (is_active IN ('ACTIVE','INACTIVE'))
);

CREATE TABLE refresh_tokens (
  id          NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  user_id     NUMBER NOT NULL,
  token       VARCHAR2(500) NOT NULL,
  device_info VARCHAR2(255) NOT NULL,
  is_valid    VARCHAR2(8) DEFAULT 'ACTIVE' NOT NULL,
  expires_at  TIMESTAMP NOT NULL,
  created_at  TIMESTAMP DEFAULT SYSTIMESTAMP NOT NULL,
  updated_at  TIMESTAMP DEFAULT SYSTIMESTAMP NOT NULL,
  CONSTRAINT fk_refresh_tokens_user FOREIGN KEY (user_id) REFERENCES users(id),
  CONSTRAINT ck_refresh_tokens_valid CHECK (is_valid IN ('ACTIVE','INACTIVE'))
);
CREATE INDEX idx_refresh_tokens_user ON refresh_tokens (user_id);
CREATE INDEX idx_role_users_user ON role_users (user_id);
CREATE INDEX idx_resource_roles_role ON resource_roles (role_id);

-- =====================================================================
-- 2. MODELO DE NEGOCIO
-- =====================================================================
CREATE TABLE clientes (
  id               NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  tipo_documento   VARCHAR2(10)  NOT NULL,
  numero_documento VARCHAR2(30)  NOT NULL,
  nombre           VARCHAR2(150) NOT NULL,
  telefono         VARCHAR2(30)  NULL,
  email            VARCHAR2(150) NULL,
  is_active        VARCHAR2(8) DEFAULT 'ACTIVE' NOT NULL,
  CONSTRAINT uq_clientes_documento UNIQUE (numero_documento),
  CONSTRAINT ck_clientes_active CHECK (is_active IN ('ACTIVE','INACTIVE'))
);

CREATE TABLE salones (
  id          NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  nombre      VARCHAR2(120) NOT NULL,
  descripcion VARCHAR2(500) NULL,
  capacidad   NUMBER(6) DEFAULT 1 NOT NULL,
  precio_base NUMBER(14,2) DEFAULT 0 NOT NULL,
  is_active   VARCHAR2(8) DEFAULT 'ACTIVE' NOT NULL,
  created_at  TIMESTAMP DEFAULT SYSTIMESTAMP NOT NULL,
  updated_at  TIMESTAMP DEFAULT SYSTIMESTAMP NOT NULL,
  CONSTRAINT uq_salones_nombre UNIQUE (nombre),
  CONSTRAINT ck_salones_capacidad CHECK (capacidad > 0),
  CONSTRAINT ck_salones_precio CHECK (precio_base >= 0),
  CONSTRAINT ck_salones_active CHECK (is_active IN ('ACTIVE','INACTIVE'))
);

CREATE TABLE servicios (
  id          NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  nombre      VARCHAR2(120) NOT NULL,
  descripcion VARCHAR2(500) NULL,
  precio_base NUMBER(14,2) DEFAULT 0 NOT NULL,
  is_active   VARCHAR2(8) DEFAULT 'ACTIVE' NOT NULL,
  created_at  TIMESTAMP DEFAULT SYSTIMESTAMP NOT NULL,
  updated_at  TIMESTAMP DEFAULT SYSTIMESTAMP NOT NULL,
  CONSTRAINT uq_servicios_nombre UNIQUE (nombre),
  CONSTRAINT ck_servicios_precio CHECK (precio_base >= 0),
  CONSTRAINT ck_servicios_active CHECK (is_active IN ('ACTIVE','INACTIVE'))
);

CREATE TABLE proveedores (
  id           NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  nit          VARCHAR2(30)  NOT NULL,
  razon_social VARCHAR2(150) NOT NULL,
  contacto     VARCHAR2(120) NULL,
  telefono     VARCHAR2(30)  NULL,
  email        VARCHAR2(150) NULL,
  is_active    VARCHAR2(8) DEFAULT 'ACTIVE' NOT NULL,
  CONSTRAINT uq_proveedores_nit UNIQUE (nit),
  CONSTRAINT ck_proveedores_active CHECK (is_active IN ('ACTIVE','INACTIVE'))
);

CREATE TABLE reservas (
  id                NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  cliente_id        NUMBER NOT NULL,
  salon_id          NUMBER NOT NULL,
  fecha_inicio      TIMESTAMP NOT NULL,
  fecha_fin         TIMESTAMP NOT NULL,
  estado            VARCHAR2(15) DEFAULT 'PRE_RESERVA' NOT NULL,
  bloqueo_expira_at TIMESTAMP NULL,
  observaciones     VARCHAR2(500) NULL,
  CONSTRAINT fk_reservas_cliente FOREIGN KEY (cliente_id) REFERENCES clientes(id),
  CONSTRAINT fk_reservas_salon   FOREIGN KEY (salon_id)   REFERENCES salones(id),
  CONSTRAINT ck_reservas_fechas CHECK (fecha_fin > fecha_inicio),
  CONSTRAINT ck_reservas_estado CHECK (estado IN ('PRE_RESERVA','CONFIRMADA','CANCELADA','FINALIZADA'))
);
CREATE INDEX idx_reservas_disponibilidad ON reservas (salon_id, fecha_inicio, fecha_fin);
CREATE INDEX idx_reservas_cliente ON reservas (cliente_id);

CREATE TABLE eventos (
  id            NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  reserva_id    NUMBER NOT NULL,             -- referencia_id (FK) hacia reservas
  tipo          VARCHAR2(60) NOT NULL,
  fecha         TIMESTAMP NOT NULL,
  cantidad      NUMBER(8) NOT NULL,          -- numero de asistentes
  observaciones VARCHAR2(500) NULL,
  estado        VARCHAR2(15) DEFAULT 'PLANEADO' NOT NULL,
  CONSTRAINT fk_eventos_reserva FOREIGN KEY (reserva_id) REFERENCES reservas(id),
  CONSTRAINT uq_eventos_reserva UNIQUE (reserva_id),   -- Reserva 0..1 : 1 Evento
  CONSTRAINT ck_eventos_cantidad CHECK (cantidad > 0),
  CONSTRAINT ck_eventos_estado CHECK (estado IN ('PLANEADO','EN_CURSO','REALIZADO','CANCELADO'))
);

CREATE TABLE evento_servicios (
  id              NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  evento_id       NUMBER NOT NULL,
  servicio_id     NUMBER NOT NULL,
  proveedor_id    NUMBER NOT NULL,
  tipo            VARCHAR2(60) NOT NULL,
  fecha           TIMESTAMP NOT NULL,
  cantidad        NUMBER(8) DEFAULT 1 NOT NULL,
  precio_unitario NUMBER(14,2) DEFAULT 0 NOT NULL,
  observaciones   VARCHAR2(500) NULL,
  estado          VARCHAR2(15) DEFAULT 'PENDIENTE' NOT NULL,
  CONSTRAINT fk_evser_evento    FOREIGN KEY (evento_id)    REFERENCES eventos(id),
  CONSTRAINT fk_evser_servicio  FOREIGN KEY (servicio_id)  REFERENCES servicios(id),
  CONSTRAINT fk_evser_proveedor FOREIGN KEY (proveedor_id) REFERENCES proveedores(id),
  CONSTRAINT ck_evser_cantidad CHECK (cantidad > 0),
  CONSTRAINT ck_evser_estado CHECK (estado IN ('PENDIENTE','CONFIRMADO','CANCELADO'))
);
CREATE INDEX idx_evser_evento ON evento_servicios (evento_id);
CREATE INDEX idx_evser_servicio ON evento_servicios (servicio_id);
CREATE INDEX idx_evser_proveedor ON evento_servicios (proveedor_id);

CREATE TABLE contratos (
  id                  NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  cliente_id          NUMBER NOT NULL,
  evento_id           NUMBER NOT NULL,
  numero              VARCHAR2(30) NOT NULL,
  fecha_inicio        DATE NOT NULL,
  fecha_fin           DATE NOT NULL,
  valor               NUMBER(14,2) NOT NULL,
  porcentaje_anticipo NUMBER(5,2) DEFAULT 30 NOT NULL,
  version_actual      NUMBER(6) DEFAULT 1 NOT NULL,
  estado              VARCHAR2(15) DEFAULT 'BORRADOR' NOT NULL,
  CONSTRAINT fk_contratos_cliente FOREIGN KEY (cliente_id) REFERENCES clientes(id),
  CONSTRAINT fk_contratos_evento  FOREIGN KEY (evento_id)  REFERENCES eventos(id),
  CONSTRAINT uq_contratos_numero UNIQUE (numero),
  CONSTRAINT uq_contratos_evento UNIQUE (evento_id),     -- Evento 1 : 1 Contrato
  CONSTRAINT ck_contratos_fechas CHECK (fecha_fin >= fecha_inicio),
  CONSTRAINT ck_contratos_valor  CHECK (valor >= 0),
  CONSTRAINT ck_contratos_anticipo CHECK (porcentaje_anticipo BETWEEN 0 AND 100),
  CONSTRAINT ck_contratos_estado CHECK (estado IN ('BORRADOR','VIGENTE','MODIFICADO','CANCELADO','FINALIZADO'))
);
CREATE INDEX idx_contratos_cliente ON contratos (cliente_id);

CREATE TABLE pagos (
  id              NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  contrato_id     NUMBER NOT NULL,
  referencia_tipo VARCHAR2(30) DEFAULT 'CONTRATO' NOT NULL,
  referencia_id   NUMBER NOT NULL,
  tipo_pago       VARCHAR2(15) DEFAULT 'ABONO' NOT NULL,
  metodo          VARCHAR2(30) NOT NULL,
  monto           NUMBER(14,2) NOT NULL,
  fecha           TIMESTAMP DEFAULT SYSTIMESTAMP NOT NULL,
  estado          VARCHAR2(15) DEFAULT 'PENDIENTE' NOT NULL,
  CONSTRAINT fk_pagos_contrato FOREIGN KEY (contrato_id) REFERENCES contratos(id),
  CONSTRAINT ck_pagos_monto CHECK (monto > 0),
  CONSTRAINT ck_pagos_tipo CHECK (tipo_pago IN ('ANTICIPO','ABONO','SALDO','PENALIDAD')),
  CONSTRAINT ck_pagos_estado CHECK (estado IN ('PENDIENTE','APLICADO','ANULADO'))
);
CREATE INDEX idx_pagos_contrato ON pagos (contrato_id);
CREATE INDEX idx_pagos_referencia ON pagos (referencia_tipo, referencia_id);

CREATE TABLE cambios_contrato (
  id             NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  contrato_id    NUMBER NOT NULL,
  version        NUMBER(6) NOT NULL,
  nombre         VARCHAR2(120) NOT NULL,
  descripcion    VARCHAR2(500) NULL,
  valor_anterior NUMBER(14,2) NOT NULL,
  valor_nuevo    NUMBER(14,2) NOT NULL,
  estado         VARCHAR2(15) DEFAULT 'SOLICITADO' NOT NULL,
  is_active      VARCHAR2(8) DEFAULT 'ACTIVE' NOT NULL,
  created_at     TIMESTAMP DEFAULT SYSTIMESTAMP NOT NULL,
  updated_at     TIMESTAMP DEFAULT SYSTIMESTAMP NOT NULL,
  CONSTRAINT fk_cambios_contrato FOREIGN KEY (contrato_id) REFERENCES contratos(id),
  CONSTRAINT uq_cambios_version UNIQUE (contrato_id, version),
  CONSTRAINT ck_cambios_estado CHECK (estado IN ('SOLICITADO','APROBADO','RECHAZADO')),
  CONSTRAINT ck_cambios_active CHECK (is_active IN ('ACTIVE','INACTIVE'))
);

CREATE TABLE cancelaciones (
  id                   NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  contrato_id          NUMBER NOT NULL,
  nombre               VARCHAR2(120) NOT NULL,
  descripcion          VARCHAR2(500) NULL,
  porcentaje_penalidad NUMBER(5,2) DEFAULT 0 NOT NULL,
  valor_penalidad      NUMBER(14,2) DEFAULT 0 NOT NULL,
  estado               VARCHAR2(15) DEFAULT 'SOLICITADA' NOT NULL,
  is_active            VARCHAR2(8) DEFAULT 'ACTIVE' NOT NULL,
  created_at           TIMESTAMP DEFAULT SYSTIMESTAMP NOT NULL,
  updated_at           TIMESTAMP DEFAULT SYSTIMESTAMP NOT NULL,
  CONSTRAINT fk_cancelaciones_contrato FOREIGN KEY (contrato_id) REFERENCES contratos(id),
  CONSTRAINT ck_cancel_porcentaje CHECK (porcentaje_penalidad BETWEEN 0 AND 100),
  CONSTRAINT ck_cancel_estado CHECK (estado IN ('SOLICITADA','APLICADA','ANULADA')),
  CONSTRAINT ck_cancel_active CHECK (is_active IN ('ACTIVE','INACTIVE'))
);
CREATE INDEX idx_cancelaciones_contrato ON cancelaciones (contrato_id);

-- =====================================================================
-- 3. DATOS SEMILLA (RBAC)
-- =====================================================================
INSERT INTO roles (name) VALUES ('ADMIN');
INSERT INTO roles (name) VALUES ('COMERCIAL');
INSERT INTO roles (name) VALUES ('OPERACIONES');
INSERT INTO roles (name) VALUES ('PROVEEDOR');
INSERT INTO roles (name) VALUES ('CARTERA');

INSERT INTO resources (path, method) VALUES ('/reservas', 'POST');
INSERT INTO resources (path, method) VALUES ('/contratos', 'POST');
INSERT INTO resources (path, method) VALUES ('/cambios-contrato', 'POST');
INSERT INTO resources (path, method) VALUES ('/cancelaciones', 'POST');

-- ADMIN accede a todos los recursos
INSERT INTO resource_roles (resource_id, role_id)
SELECT r.id, ro.id FROM resources r CROSS JOIN roles ro WHERE ro.name = 'ADMIN';

-- COMERCIAL: reservas y contratos | OPERACIONES: reservas y cambios | CARTERA: contratos y cancelaciones
INSERT INTO resource_roles (resource_id, role_id)
SELECT r.id, ro.id FROM resources r JOIN roles ro
  ON (ro.name = 'COMERCIAL'   AND r.path IN ('/reservas','/contratos'))
  OR (ro.name = 'OPERACIONES' AND r.path IN ('/reservas','/cambios-contrato'))
  OR (ro.name = 'CARTERA'     AND r.path IN ('/contratos','/cancelaciones'));

-- Usuario administrador de prueba. IMPORTANTE: reemplazar el hash por uno generado con bcrypt desde tu backend.
INSERT INTO users (username, email, password)
VALUES ('admin', 'admin@celebrahub.com', '$2b$10$REEMPLAZAR_POR_HASH_BCRYPT_GENERADO_EN_EL_BACKEND');

INSERT INTO role_users (role_id, user_id)
SELECT ro.id, u.id FROM roles ro, users u WHERE ro.name = 'ADMIN' AND u.username = 'admin';

-- =====================================================================
-- 4. DATOS SEMILLA (NEGOCIO)
-- =====================================================================
INSERT INTO clientes (tipo_documento, numero_documento, nombre, telefono, email)
VALUES ('CC', '1001234567', 'Laura Martinez', '3001112233', 'laura@mail.com');
INSERT INTO clientes (tipo_documento, numero_documento, nombre, telefono, email)
VALUES ('NIT', '900123456-1', 'Empresa Andina SAS', '6015556677', 'eventos@andina.com');

INSERT INTO salones (nombre, descripcion, capacidad, precio_base)
VALUES ('Salon Perla', 'Salon principal con terraza', 250, 4500000);
INSERT INTO salones (nombre, descripcion, capacidad, precio_base)
VALUES ('Salon Coral', 'Salon mediano para eventos corporativos', 120, 2500000);
INSERT INTO salones (nombre, descripcion, capacidad, precio_base)
VALUES ('Salon Brisa', 'Salon intimo para celebraciones pequenas', 60, 1500000);

INSERT INTO servicios (nombre, descripcion, precio_base)
VALUES ('Catering', 'Alimentos y bebidas por persona', 85000);
INSERT INTO servicios (nombre, descripcion, precio_base)
VALUES ('Mobiliario', 'Mesas, sillas y manteleria', 1200000);
INSERT INTO servicios (nombre, descripcion, precio_base)
VALUES ('Decoracion', 'Ambientacion tematica', 1800000);
INSERT INTO servicios (nombre, descripcion, precio_base)
VALUES ('Sonido e iluminacion', 'DJ, sonido e iluminacion', 2000000);

INSERT INTO proveedores (nit, razon_social, contacto, telefono, email)
VALUES ('800111222-3', 'Sabores del Caribe SAS', 'Carlos Perez', '3109998877', 'ventas@sabores.com');
INSERT INTO proveedores (nit, razon_social, contacto, telefono, email)
VALUES ('800333444-5', 'Eventos Sonido Pro', 'Ana Rojas', '3125554433', 'contacto@sonidopro.com');

COMMIT;
```

Después de ejecutar cada instrucción, Oracle debe mostrar el mensaje `Table created.`. Compruebo que las tablas del esquema `einer_coa` quedaron creadas:

```sql
SELECT owner, table_name
FROM all_tables
WHERE owner = 'EINER_COA'
ORDER BY table_name;
```

**Evidencia (imágenes):**

![Evidencia de la creación de las tablas de CelebraHub_Centro_de_eventos en Oracle XE](./evidencia/image-69.png)
![Evidencia adicional de la creación de las tablas en Oracle XE](./evidencia/image-70.png)

![Evidencia de la verificación de las tablas de CelebraHub_Centro_de_eventos en Oracle XE](./evidencia/image-71.png)
![Evidencia adicional de la verificación de las tablas en Oracle XE](./evidencia/image-72.png)

**Registro:**

Como la consulta anterior mostró `no rows selected`, utilicé la imagen de mi proyecto final **CelebraHub - Centro de eventos** para definir las tablas que necesito en Oracle. Ejecuté el código con las 17 entidades del modelo y sus relaciones principales dentro del esquema `einer_coa`. Después consulté `all_tables` filtrando el propietario `EINER_COA` y Oracle mostró 17 filas, por lo que confirmé que las tablas quedaron creadas correctamente. Finalmente salí de SQL*Plus, salí del contenedor y ejecuté `date` para registrar la fecha de la actividad.

### 4.7 Conectar remotamente desde cualquier equipo

##### **Hacerlo desde DBeaver**

Para conectarme remotamente a Oracle XE utilizo DBeaver. Primero consulto la dirección IP del equipo donde está ejecutándose Docker:

```bash
ip a
```

En el resultado busco la interfaz `eth0`. En mi caso, utilizo la dirección IP `172.18.50.255` como **Host**.

También verifico que el contenedor esté activo y que el puerto `1521` esté publicado:

```bash
sudo docker ps --filter "name=oracle-server" --format "table {{.Names}}\t{{.Ports}}"
```

Compruebo que el puerto `1521` esté permitido por UFW:

```bash
sudo ufw status
```

**Evidencia (imagen):**

![evidencia](./evidencia/image-73.png)

**Registro:**

Consulté la configuración de red con `ip a` y comprobé que la interfaz `eth0` tiene la dirección IP `172.18.50.255`. También verifiqué que el contenedor `oracle-server` está activo y publica el puerto `1521` mediante `0.0.0.0:1521->1521/tcp`. Finalmente comprobé con `sudo ufw status` que el firewall está activo y que el puerto `1521/tcp` está permitido para IPv4 e IPv6. Ejecuté `date` para registrar la fecha de la actividad.

En DBeaver realizo los siguientes pasos:

1. Abro DBeaver.
2. Selecciono **Nueva conexión**.
3. Elijo el controlador **Oracle**.

En la pestaña **Basic** dejo la conexión con estos valores:

4. Escribo `172.18.50.255` en **Host**.
5. Escribo `1521` en **Port**.
6. En el campo **Database**, escribo `CelebraHub_Centro_de_eventos`.
7. En el selector que aparece a la derecha de **Database**, selecciono **Service Name**.
8. Escribo `einer_coa` en **Nombre de usuario**.
9. Escribo `einer123` en **Contraseña**.
10. En **Role**, selecciono **Normal**.
11. Dejo **Authentication** en **Username/password**.
12. Dejo **Local Client** en `<not present>`.
13. Presiono **Probar conexión...**.
14. Si la prueba es correcta, presiono **Finalizar** para guardar la conexión.

En **Role** aparecen tres opciones:

- **Normal:** es la opción que utilizo con el usuario `einer_coa`.
- **SYSDBA:** la utilizo únicamente si me conecto con `SYS` para tareas administrativas.
- **SYSOPER:** la utilizo únicamente para operaciones especiales de administración de la instancia.

Para mi conexión no selecciono **SYSDBA** ni **SYSOPER**.

Para la conexión remota utilizo los siguientes valores:

- **Motor:** Oracle.
- **Host:** `172.18.50.255`.
- **Port:** `1521`.
- **Service Name:** `CelebraHub_Centro_de_eventos`.
- **Username:** `einer_coa`.
- **Password:** `einer123`.

En Oracle utilizo **Service Name** en lugar de **Database**, porque `CelebraHub_Centro_de_eventos` es el nombre del PDB y del servicio de conexión.

**Evidencia (imágenes):**

![alt text](./evidencia/image-74.png)

![alt text](./evidencia/image-75.png)

 ![alt text](./evidencia/image-76.png)
**Registro:**

Consulté la dirección IP del equipo mediante `ip a` y utilicé `172.18.50.255` como Host. Después comprobé que el contenedor `oracle-server` estuviera activo, que el puerto `1521` estuviera publicado y que UFW permitiera ese puerto. Finalmente configuré en DBeaver una conexión Oracle con el Service Name `CelebraHub_Centro_de_eventos`, el usuario `einer_coa` y la contraseña `einer123`. La prueba fue exitosa y DBeaver mostró el mensaje `Conectado`; también verifiqué los detalles de la conexión y observé la conexión `CelebraHub_Centro_de_eventos Oracle` en el panel de conexiones.

### 4.8 Backup de una base de datos

Realizo un backup del PDB `CelebraHub_Centro_de_eventos` utilizando Oracle Data Pump y el usuario administrador `SYSTEM`:

```bash
sudo docker exec oracle-server expdp system/einer123@CelebraHub_Centro_de_eventos directory=DATA_PUMP_DIR dumpfile=backup_CelebraHub_Centro_de_eventos.dmp logfile=backup_CelebraHub_Centro_de_eventos.log
```

El comando genera el archivo `backup_CelebraHub_Centro_de_eventos.dmp` y el registro `backup_CelebraHub_Centro_de_eventos.log` dentro del directorio de Data Pump configurado por Oracle.

**Evidencia del primer intento (imagen):**

![Evidencia del error al realizar el backup de Oracle](./evidencia/image-77.png)

**Registro:**

Realicé el primer intento de backup del PDB `CelebraHub_Centro_de_eventos` utilizando el usuario `SYSTEM`, la contraseña `einer123` y el directorio `DATA_PUMP_DIR`. El comando no terminó correctamente: Oracle mostró `ORA-12154: TNS:could not resolve the connect identifier specified`, porque `@CelebraHub_Centro_de_eventos` se interpretó como un alias TNS que no estaba configurado. Por esta razón, en este intento no se confirmó la creación del archivo `backup_CelebraHub_Centro_de_eventos.dmp`.

Para repetir el backup utilizando la conexión EZCONNECT, ejecuto:

```bash
sudo docker exec oracle-server expdp 'system/einer123@//localhost:1521/CelebraHub_Centro_de_eventos' directory=DATA_PUMP_DIR dumpfile=backup_CelebraHub_Centro_de_eventos.dmp logfile=backup_CelebraHub_Centro_de_eventos.log
```

**Evidencia del segundo intento (imagen):**

![Evidencia del backup exitoso de CelebraHub_Centro_de_eventos en Oracle XE](./evidencia/image-79.png)

**Registro del segundo intento:**

Después de corregir la cadena de conexión y utilizar EZCONNECT, ejecuté nuevamente el backup de `CelebraHub_Centro_de_eventos`. Oracle confirmó la conexión con la base de datos, procesó los objetos del esquema y generó el archivo `backup_CelebraHub_Centro_de_eventos.dmp` dentro de `DATA_PUMP_DIR`. El proceso terminó correctamente con el mensaje `Job ... completed` y finalmente ejecuté `date` para registrar la fecha de la actividad.

### 4.9 Variables clave del `.env`

Estas son las variables principales que configuré para Oracle XE:

| Variable | Descripción | Valor |
| --- | --- | --- |
| `TZ` | Zona horaria | `America/Bogota` |
| `ORACLE_PASSWORD` | Contraseña de `SYSTEM` y `SYS` | `einer123` |
| `ORACLE_DATABASE` | PDB inicial / Service Name | `CelebraHub_Centro_de_eventos` |

Para consultar nuevamente estas variables, ejecuto:

```bash
cat ~/ia-lab/services/motores-bd/oracle/.env
```

**Evidencia (imagen):**

![Evidencia de las variables del archivo .env de Oracle XE](./evidencia/image-78.png)

**Registro:**

Consulté el contenido del archivo `.env` con el comando `cat` y confirmé que contiene `TZ=America/Bogota`, `ORACLE_PASSWORD=einer123` y `ORACLE_DATABASE=CelebraHub_Centro_de_eventos`. También ejecuté `date` y registré la fecha de la actividad.

# Conclusión

Durante el desarrollo de este taller instalé, configuré y verifiqué satisfactoriamente los cuatro motores de bases de datos solicitados: MySQL, PostgreSQL, MS SQL Server y Oracle XE. En cada motor creé la base de datos `CelebraHub_Centro_de_eventos`, configuré los contenedores Docker, revisé los registros de ejecución, habilité los puertos correspondientes y comprobé las conexiones locales y remotas mediante DBeaver.

El proceso me permitió resolver diferentes situaciones técnicas, como errores de indentación en archivos YAML, permisos de las carpetas de datos de Oracle, configuración de usuarios y diferencias entre los comandos de cada motor. También creé y verifiqué las tablas del proyecto **CelebraHub - Centro de eventos**, manteniendo sus relaciones principales en los cuatro motores. Finalmente confirmé que los servicios quedaron activos y disponibles para trabajar con la información del proyecto.

Este informe de trazabilidad reúne los comandos utilizados, las configuraciones aplicadas, los problemas encontrados, las soluciones implementadas y las evidencias obtenidas durante el proceso.


Estudiante de Ingeniería de Sistemas
alvaro javier montero fonseca  
Facultad de Ingeniería  
Universidad de La Guajira     
Fecha del informe de trazabilidad: 9 de septiembre de 2026



