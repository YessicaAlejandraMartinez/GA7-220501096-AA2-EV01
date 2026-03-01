# Sistema de Agendamiento — Sergio Calderón Atelier de Modas

![Java](https://img.shields.io/badge/Java-25-orange?style=flat-square&logo=java)
![Maven](https://img.shields.io/badge/Maven-3.9.12-red?style=flat-square&logo=apache-maven)
![Tomcat](https://img.shields.io/badge/Tomcat-10.1.52-yellow?style=flat-square&logo=apache-tomcat)
![MySQL](https://img.shields.io/badge/MySQL-8.0-blue?style=flat-square&logo=mysql)
![Bootstrap](https://img.shields.io/badge/Bootstrap-5.3-purple?style=flat-square&logo=bootstrap)

> **Evidencia:** GA7-220501096-AA2-EV01 / GA7-220501096-AA2-EV02  
> **Actividad:** GA7-220501096-AA2 — Aplicar estándares de codificación  
> **Programa:** Análisis y Desarrollo de Software — SENA

---

## Descripción del Proyecto

Sistema web de agendamiento de citas para el **Atelier de Modas Sergio Calderón**, desarrollado con tecnologías Java EE. Permite a los clientes explorar el catálogo de vestidos, registrarse, iniciar sesión y gestionar sus citas. El administrador puede configurar horarios disponibles, gestionar citas y administrar usuarios.

### Funcionalidades principales

**Módulo Cliente:**
- Registro e inicio de sesión
- Visualización del catálogo de vestidos con filtros por categoría
- Agendar, modificar y cancelar citas
- Historial de citas propias

**Módulo Administrador:**
- Dashboard con estadísticas
- Gestión completa de horarios disponibles
- Gestión de citas de todos los clientes
- Administración de usuarios


## Tecnologías Utilizadas

- Java JDK  25 (Eclipse Adoptium) - Lenguaje principal 
- Apache Maven 3.9.12 - Gestión de dependencias 
- Apache Tomcat  10.1.52 - Servidor de aplicaciones 
- MySQL  - Base de datos relacional 
- JDBC  -  Conexión a base de datos 
- Servlets - Jakarta EE 5.0 - Controladores web 
- JSP Jakarta 3.0 - Vistas del sistema 
- JSTL 2.0 - Librería de etiquetas JSP 
- Bootstrap 5.3 - Framework CSS 
- Font Awesome 6.4 - Iconografía 


## Requisitos Previos

Antes de ejecutar el proyecto asegúrate de tener instalado:

- **JDK 17 o superior** — [Eclipse Adoptium](https://adoptium.net/)
- **Apache Maven 3.9+** — [maven.apache.org](https://maven.apache.org/)
- **Apache Tomcat 10.1+** — [tomcat.apache.org](https://tomcat.apache.org/)
- **MySQL 8.0+** — [mysql.com](https://www.mysql.com/)
- **MySQL Workbench** (opcional, recomendado)

---



### 1. Clonar el repositorio

```bash
git clone https://github.com/YessicaAlejandraMartinez/GA7-220501096-AA2-EV01.git
```

### 2. Configurar la base de datos

Abrir **MySQL Workbench** y ejecuta el script para la base de datos:


DROP DATABASE IF EXISTS sergiocalderon_db;
CREATE DATABASE sergiocalderon_db
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_unicode_ci;

USE sergiocalderon_db;

CREATE TABLE usuario (
    id_usuario      INT AUTO_INCREMENT PRIMARY KEY,
    nombre          VARCHAR(100) NOT NULL,
    apellido        VARCHAR(100) NOT NULL,
    email           VARCHAR(150) NOT NULL UNIQUE,
    contrasena      VARCHAR(255) NOT NULL,
    telefono        VARCHAR(20),
    tipo_usuario    ENUM('ADMIN', 'CLIENTE') NOT NULL DEFAULT 'CLIENTE',
    fecha_registro  TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE cliente (
    id_cliente      INT PRIMARY KEY,
    direccion       VARCHAR(200) DEFAULT '',
    ciudad          VARCHAR(100) DEFAULT '',
    numero_citas    INT DEFAULT 0,
    CONSTRAINT fk_cliente_usuario
        FOREIGN KEY (id_cliente)
        REFERENCES usuario(id_usuario)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);

CREATE TABLE personal_administrativo (
    id_administrador    INT PRIMARY KEY,
    cargo               VARCHAR(100) NOT NULL,
    permisos            JSON,
    CONSTRAINT fk_admin_usuario
        FOREIGN KEY (id_administrador)
        REFERENCES usuario(id_usuario)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);

CREATE TABLE configuracion_horario (
    id_configuracion    INT AUTO_INCREMENT PRIMARY KEY,
    dia_semana          ENUM('LUNES','MARTES','MIERCOLES',
                             'JUEVES','VIERNES','SABADO','DOMINGO')
                        NOT NULL,
    hora_inicio         TIME NOT NULL,
    hora_fin            TIME NOT NULL,
    intervalo_minutos   INT NOT NULL DEFAULT 60,
    activo              TINYINT(1) DEFAULT 1,
    id_administrador    INT NOT NULL,
    CONSTRAINT fk_confhorario_admin
        FOREIGN KEY (id_administrador)
        REFERENCES personal_administrativo(id_administrador)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);

CREATE TABLE disponibilidad (
    id_disponibilidad   INT AUTO_INCREMENT PRIMARY KEY,
    fecha               DATE NOT NULL,
    hora_inicio         TIME NOT NULL,
    hora_fin            TIME NOT NULL,
    disponible          TINYINT(1) DEFAULT 1,
    cupos_totales       INT NOT NULL DEFAULT 1,
    cupos_ocupados      INT NOT NULL DEFAULT 0,
    fecha_creacion      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    fecha_modificacion  TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                        ON UPDATE CURRENT_TIMESTAMP,
    id_administrador    INT NOT NULL,
    CONSTRAINT fk_disp_admin
        FOREIGN KEY (id_administrador)
        REFERENCES personal_administrativo(id_administrador)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);

CREATE TABLE bloqueo_calendario (
    id_bloqueo          INT AUTO_INCREMENT PRIMARY KEY,
    fecha_inicio        DATE NOT NULL,
    fecha_fin           DATE NOT NULL,
    hora_inicio         TIME,
    hora_fin            TIME,
    motivo              VARCHAR(255),
    tipo_bloqueo        ENUM('VACACIONES','FESTIVO',
                             'MANTENIMIENTO','OTRO')
                        DEFAULT 'OTRO',
    todo_el_dia         TINYINT(1) DEFAULT 0,
    activo              TINYINT(1) DEFAULT 1,
    id_administrador    INT NOT NULL,
    CONSTRAINT fk_bloqueo_admin
        FOREIGN KEY (id_administrador)
        REFERENCES personal_administrativo(id_administrador)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);

CREATE TABLE cita (
    id_cita                 INT AUTO_INCREMENT PRIMARY KEY,
    fecha_cita              DATE NOT NULL,
    hora_inicio             TIME NOT NULL,
    hora_fin                TIME NOT NULL,
    duracion_minutos        INT DEFAULT 60,
    tipo_evento             ENUM('BODA','FIESTA','QUINCE','OTRO')
                            NOT NULL DEFAULT 'OTRO',
    motivo_cita             TEXT,
    estado                  ENUM('PENDIENTE','CONFIRMADA',
                                 'COMPLETADA','CANCELADA')
                            NOT NULL DEFAULT 'PENDIENTE',
    fecha_creacion          TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    fecha_modificacion      TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                            ON UPDATE CURRENT_TIMESTAMP,
    fecha_cancelacion       DATETIME,
    motivo_cancelacion      TEXT,
    id_cliente              INT NOT NULL,
    id_disponibilidad       INT,
    id_administrador_asigno INT,
    CONSTRAINT fk_cita_cliente
        FOREIGN KEY (id_cliente)
        REFERENCES cliente(id_cliente)
        ON DELETE CASCADE
        ON UPDATE CASCADE,
    CONSTRAINT fk_cita_disponibilidad
        FOREIGN KEY (id_disponibilidad)
        REFERENCES disponibilidad(id_disponibilidad)
        ON DELETE SET NULL
        ON UPDATE CASCADE,
    CONSTRAINT fk_cita_admin
        FOREIGN KEY (id_administrador_asigno)
        REFERENCES personal_administrativo(id_administrador)
        ON DELETE SET NULL
        ON UPDATE CASCADE
);

CREATE TABLE historial_cita (
    id_historial            INT AUTO_INCREMENT PRIMARY KEY,
    accion                  VARCHAR(100) NOT NULL,
    estado_anterior         VARCHAR(50),
    estado_nuevo            VARCHAR(50),
    campos_modificados      JSON,
    fecha_accion            TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    descripcion             TEXT,
    id_cita                 INT NOT NULL,
    id_usuario_responsable  INT NOT NULL,
    CONSTRAINT fk_historial_cita
        FOREIGN KEY (id_cita)
        REFERENCES cita(id_cita)
        ON DELETE CASCADE
        ON UPDATE CASCADE,
    CONSTRAINT fk_historial_usuario
        FOREIGN KEY (id_usuario_responsable)
        REFERENCES usuario(id_usuario)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);

CREATE TABLE notificacion (
    id_notificacion     INT AUTO_INCREMENT PRIMARY KEY,
    tipo_notificacion   ENUM('CONFIRMACION','RECORDATORIO',
                             'CANCELACION','MODIFICACION')
                        NOT NULL,
    titulo              VARCHAR(200) NOT NULL,
    mensaje             TEXT NOT NULL,
    canal_envio         ENUM('EMAIL','SMS','WHATSAPP')
                        DEFAULT 'EMAIL',
    destinatario        VARCHAR(150) NOT NULL,
    fecha_programada    DATETIME,
    fecha_envio         DATETIME,
    estado_envio        ENUM('PENDIENTE','ENVIADO','FALLIDO')
                        DEFAULT 'PENDIENTE',
    id_cita             INT,
    CONSTRAINT fk_notif_cita
        FOREIGN KEY (id_cita)
        REFERENCES cita(id_cita)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);

-- DATOS INICIALES


-- Usuario administrador
INSERT INTO usuario (
    id_usuario, nombre, apellido, email,
    contrasena, telefono, tipo_usuario
) VALUES (
    1, 'Sergio', 'Calderon',
    'admin@sergiocalderon.com',
    'admin123', '3001234567', 'ADMIN'
);

-- Registro en personal_administrativo
INSERT INTO personal_administrativo (
    id_administrador, cargo, permisos
) VALUES (
    1,
    'Administrador Principal',
    '{"citas": true, "usuarios": true, "horarios": true, "catalogo": true}'
);

SELECT 'BASE DE DATOS CREADA EXITOSAMENTE' AS resultado;
SHOW TABLES;

### 3. Configurar la conexión a la base de datos

Abrir el archivo:

```
src/main/java/com/sergiocalderon/dao/ConexionDB.java
```

Actualizar las credenciales según el entorno de mysql:

```java
private static final String URL =
    "jdbc:mysql://localhost:3306/sergiocalderon_db" +
    "?useSSL=false&serverTimezone=America/Bogota" +
    "&allowPublicKeyRetrieval=true";
private static final String USUARIO = "root";
private static final String CONTRASENA = "tu_contraseña";
```

### 4. Compilar y empaquetar

```bash
mvn clean package
```

### 5. Desplegar en Tomcat

```bash
copy /Y target\agendamiento.war C:\apache-tomcat-10.1.52\webapps\
o desde visual
Copy-Item "target\agendamiento.war" "C:\apache-tomcat-10.1.52\webapps\" -Force
```

### 6. Iniciar Tomcat

```bash
set CATALINA_HOME=C:\apache-tomcat-10.1.52
set JAVA_HOME=C:\Program Files\Eclipse Adoptium\jdk-25.0.2.10-hotspot
C:\apache-tomcat-10.1.52\bin\startup.bat

o desde visual

$env:CATALINA_HOME = "C:\apache-tomcat-10.1.52"
$env:JAVA_HOME = "C:\Program Files\Eclipse Adoptium\jdk-25.0.2.10-hotspot"
& "C:\apache-tomcat-10.1.52\bin\startup.bat"
```

### 7. Acceder al sistema

- `http://localhost:8080/agendamiento/` Página principal 
- `http://localhost:8080/agendamiento/login` Inicio de sesión 
- `http://localhost:8080/agendamiento/registro`  Registro de cliente 
- `http://localhost:8080/agendamiento/admin/horarios`  Panel admin 



## Aprendices

- Diego Armando Higuita Cortés.
- Gean Carlos Coplas Romero. 
- Luis Eduardo Zabaleta Mora.
- Yessica Alejandra Martínez Rincón.

**Institución:** Servicio Nacional de Aprendizaje — SENA  
**Programa:** Análisis y Desarrollo de Software  
**Ficha:** 3070324  
**Año:** 2026

---

> **Credenciales de prueba:**  
> Admin: `admin@sergiocalderon.com` / `admin123`  
> Cliente: Registrar nueva cuenta en `/registro`
