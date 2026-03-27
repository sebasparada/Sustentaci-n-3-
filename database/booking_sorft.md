# Base de Datos - Booking_Sorft

Este documento describe la estructura de la base de datos del sistema, organizada por módulos funcionales.

---

## 1. Seguridad y Usuarios

Gestiona el acceso, roles y auditoría del sistema.

### Tablas

- **rol**: Define los roles del sistema.
- **usuario**: Información de usuarios del sistema.
- **usuario_rol**: Relación entre usuarios y roles.
- **auditoria**: Registro de acciones realizadas.

```sql
CREATE TABLE rol (
    id          INT AUTO_INCREMENT PRIMARY KEY,
    nombre      VARCHAR(50) NOT NULL UNIQUE,
    descripcion TEXT
);

CREATE TABLE usuario (
    id                  INT AUTO_INCREMENT PRIMARY KEY,
    nombre              VARCHAR(100) NOT NULL,
    apellido            VARCHAR(100) NOT NULL,
    cedula              VARCHAR(20)  NOT NULL UNIQUE,
    email               VARCHAR(150) NOT NULL UNIQUE,
    password            VARCHAR(255) NOT NULL,
    activo              BOOLEAN   DEFAULT TRUE,
    ultimo_login        TIMESTAMP NULL,
    fecha_creacion      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    fecha_actualizacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

CREATE TABLE usuario_rol (
    usuario_id INT NOT NULL,
    rol_id     INT NOT NULL,
    PRIMARY KEY (usuario_id, rol_id),
    CONSTRAINT fk_usu_rol_usuario FOREIGN KEY (usuario_id) REFERENCES usuario(id) ON DELETE CASCADE,
    CONSTRAINT fk_usu_rol_rol     FOREIGN KEY (rol_id) REFERENCES rol(id) ON DELETE CASCADE
);

CREATE TABLE auditoria (
    id          INT AUTO_INCREMENT PRIMARY KEY,
    usuario_id  INT,
    accion      VARCHAR(50)  NOT NULL,
    entidad     VARCHAR(100) NOT NULL,
    entidad_id  INT,
    detalle     TEXT,
    ip_address  VARCHAR(45),
    fecha       TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT fk_auditoria_usuario FOREIGN KEY (usuario_id) REFERENCES usuario(id) ON DELETE SET NULL
);
```
## 2. Clientes
Administra la información de los huéspedes.
### Tablas

tipo_documento
cliente
historial_cliente

```
SQLCREATE TABLE tipo_documento (
    id     INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL UNIQUE
);

CREATE TABLE cliente (
    id                INT AUTO_INCREMENT PRIMARY KEY,
    nombre            VARCHAR(100) NOT NULL,
    apellido          VARCHAR(100) NOT NULL,
    tipo_documento_id INT,
    cedula            VARCHAR(30)  NOT NULL UNIQUE,
    telefono          VARCHAR(20),
    email             VARCHAR(150),
    nacionalidad      VARCHAR(100),
    fecha_nacimiento  DATE,
    fecha_registro    TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT fk_cliente_tipo_doc FOREIGN KEY (tipo_documento_id) REFERENCES tipo_documento(id)
);

CREATE TABLE historial_cliente (
    id          INT AUTO_INCREMENT PRIMARY KEY,
    cliente_id  INT NOT NULL,
    usuario_id  INT,
    observacion TEXT NOT NULL,
    fecha       TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT fk_hist_cliente FOREIGN KEY (cliente_id) REFERENCES cliente(id),
    CONSTRAINT fk_hist_usuario FOREIGN KEY (usuario_id) REFERENCES usuario(id)
);
```
## 3. Gestión de Unidades
Control de habitaciones o unidades del hotel.
### Tablas

tipo_unidad
estado_unidad
unidad
tipo_tarifa
tarifa
```

CREATE TABLE tipo_unidad (
    id     INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL UNIQUE
);

CREATE TABLE estado_unidad (
    id     INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL UNIQUE
);

CREATE TABLE unidad (
    id                     INT AUTO_INCREMENT PRIMARY KEY,
    numero                 VARCHAR(10) NOT NULL UNIQUE,
    piso                   INT         NOT NULL,
    tipo_unidad_id         INT,
    capacidad_adultos      INT         NOT NULL,
    capacidad_ninos        INT         NOT NULL,
    estado_unidad_id       INT,
    bloqueo_administrativo BOOLEAN DEFAULT FALSE,
    descripcion            TEXT,
    activa                 BOOLEAN DEFAULT TRUE,
    CONSTRAINT fk_unidad_tipo   FOREIGN KEY (tipo_unidad_id) REFERENCES tipo_unidad(id),
    CONSTRAINT fk_unidad_estado FOREIGN KEY (estado_unidad_id) REFERENCES estado_unidad(id)
);

CREATE TABLE tipo_tarifa (
    id     INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL UNIQUE
);

CREATE TABLE tarifa (
    id             INT AUTO_INCREMENT PRIMARY KEY,
    tipo_unidad_id INT,
    tipo_tarifa_id INT,
    nombre_tarifa  VARCHAR(100),
    precio         DECIMAL(10,2) NOT NULL,
    fecha_inicio   DATE,
    fecha_fin      DATE,
    CONSTRAINT fk_tarifa_unidad FOREIGN KEY (tipo_unidad_id) REFERENCES tipo_unidad(id),
    CONSTRAINT fk_tarifa_tipo   FOREIGN KEY (tipo_tarifa_id) REFERENCES tipo_tarifa(id)
);
```
## 4. Reservas y Estadías
Gestión completa del ciclo de reservas.
```
CREATE TABLE canal_reserva (
    id     INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL UNIQUE
);

CREATE TABLE estado_reserva (
    id     INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL UNIQUE
);

CREATE TABLE reserva (
    id                INT AUTO_INCREMENT PRIMARY KEY,
    cliente_id        INT,
    unidad_id         INT,
    tarifa_id         INT,
    canal_reserva_id  INT,
    fecha_inicio      DATE NOT NULL,
    fecha_fin         DATE NOT NULL,
    estado_reserva_id INT,
    cantidad_adultos  INT DEFAULT 1,
    cantidad_ninos    INT DEFAULT 0,
    observaciones     TEXT,
    fecha_creacion    TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT fk_reserva_cliente FOREIGN KEY (cliente_id) REFERENCES cliente(id),
    CONSTRAINT fk_reserva_unidad  FOREIGN KEY (unidad_id)  REFERENCES unidad(id),
    CONSTRAINT fk_reserva_tarifa  FOREIGN KEY (tarifa_id)  REFERENCES tarifa(id),
    CONSTRAINT fk_reserva_canal   FOREIGN KEY (canal_reserva_id) REFERENCES canal_reserva(id),
    CONSTRAINT fk_reserva_estado  FOREIGN KEY (estado_reserva_id) REFERENCES estado_reserva(id)
);

CREATE TABLE reserva_huesped (
    id         INT AUTO_INCREMENT PRIMARY KEY,
    reserva_id INT NOT NULL,
    cliente_id INT NOT NULL,
    es_titular BOOLEAN DEFAULT FALSE,
    CONSTRAINT fk_rh_reserva FOREIGN KEY (reserva_id) REFERENCES reserva(id),
    CONSTRAINT fk_rh_cliente FOREIGN KEY (cliente_id) REFERENCES cliente(id)
);

CREATE TABLE estadia (
    id                  INT AUTO_INCREMENT PRIMARY KEY,
    reserva_id          INT,
    fecha_checkin       TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    fecha_checkout      TIMESTAMP NULL DEFAULT NULL,
    usuario_checkin_id  INT,
    usuario_checkout_id INT,
    estado              VARCHAR(20) DEFAULT 'ACTIVO',
    CONSTRAINT fk_estadia_reserva  FOREIGN KEY (reserva_id) REFERENCES reserva(id),
    CONSTRAINT fk_estadia_u_in     FOREIGN KEY (usuario_checkin_id) REFERENCES usuario(id),
    CONSTRAINT fk_estadia_u_out    FOREIGN KEY (usuario_checkout_id) REFERENCES usuario(id)
);
```
## 5. Limpieza
Control del estado y mantenimiento de unidades.
```
CREATE TABLE tipo_limpieza (
    id     INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL UNIQUE
);

CREATE TABLE limpieza (
    id               INT AUTO_INCREMENT PRIMARY KEY,
    unidad_id        INT,
    usuario_id       INT,
    tipo_limpieza_id INT,
    fecha            TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    estado           VARCHAR(20) NOT NULL,
    duracion_minutos INT,
    observacion      TEXT,
    CONSTRAINT fk_limpieza_unidad FOREIGN KEY (unidad_id) REFERENCES unidad(id),
    CONSTRAINT fk_limpieza_usuario FOREIGN KEY (usuario_id) REFERENCES usuario(id),
    CONSTRAINT fk_limpieza_tipo FOREIGN KEY (tipo_limpieza_id) REFERENCES tipo_limpieza(id)
);
```
## 6. Facturación y Pagos
Módulo financiero del sistema.
```
CREATE TABLE estado_factura (
    id     INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL UNIQUE
);

CREATE TABLE factura (
    id                INT AUTO_INCREMENT PRIMARY KEY,
    cliente_id        INT,
    reserva_id        INT,
    estado_factura_id INT,
    subtotal          DECIMAL(10,2) NOT NULL DEFAULT 0,
    descuento         DECIMAL(10,2) DEFAULT 0,
    impuesto          DECIMAL(10,2) DEFAULT 0,
    total             DECIMAL(10,2) NOT NULL DEFAULT 0,
    fecha             TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT fk_factura_cliente FOREIGN KEY (cliente_id) REFERENCES cliente(id),
    CONSTRAINT fk_factura_reserva FOREIGN KEY (reserva_id) REFERENCES reserva(id),
    CONSTRAINT fk_factura_estado  FOREIGN KEY (estado_factura_id) REFERENCES estado_factura(id)
);

CREATE TABLE factura_detalle (
    id              INT AUTO_INCREMENT PRIMARY KEY,
    factura_id      INT NOT NULL,
    tipo_concepto   VARCHAR(50),
    descripcion     VARCHAR(255) NOT NULL,
    cantidad        DECIMAL(10,2) NOT NULL,
    precio_unitario DECIMAL(10,2) NOT NULL,
    subtotal        DECIMAL(10,2) AS (cantidad * precio_unitario) STORED,
    CONSTRAINT fk_detalle_factura FOREIGN KEY (factura_id) REFERENCES factura(id) ON DELETE CASCADE
);

CREATE TABLE metodo_pago (
    id     INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL UNIQUE
);

CREATE TABLE estado_pago (
    id     INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL UNIQUE
);

CREATE TABLE pago (
    id              INT AUTO_INCREMENT PRIMARY KEY,
    factura_id      INT,
    metodo_pago_id  INT,
    estado_pago_id  INT,
    monto           DECIMAL(10,2) NOT NULL,
    referencia_pago VARCHAR(100),
    usuario_id      INT,
    fecha           TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT fk_pago_factura FOREIGN KEY (factura_id) REFERENCES factura(id),
    CONSTRAINT fk_pago_metodo  FOREIGN KEY (metodo_pago_id) REFERENCES metodo_pago(id),
    CONSTRAINT fk_pago_estado  FOREIGN KEY (estado_pago_id) REFERENCES estado_pago(id),
    CONSTRAINT fk_pago_usuario FOREIGN KEY (usuario_id) REFERENCES usuario(id)
);
```
## 7. Servicios y Eventos
Servicios adicionales y gestión de eventos.
```
CREATE TABLE tipo_servicio (
    id     INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL UNIQUE
);

CREATE TABLE servicio (
    id               INT AUTO_INCREMENT PRIMARY KEY,
    nombre           VARCHAR(100) NOT NULL,
    tipo_servicio_id INT,
    precio_base      DECIMAL(10,2),
    activo           BOOLEAN DEFAULT TRUE,
    CONSTRAINT fk_servicio_tipo FOREIGN KEY (tipo_servicio_id) REFERENCES tipo_servicio(id)
);

CREATE TABLE consumo_servicio (
    id              INT AUTO_INCREMENT PRIMARY KEY,
    cliente_id      INT,
    servicio_id     INT,
    factura_id      INT,
    cantidad        DECIMAL(10,2) DEFAULT 1,
    precio_unitario DECIMAL(10,2) NOT NULL,
    observacion     TEXT,
    fecha           TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT fk_consumo_cliente FOREIGN KEY (cliente_id) REFERENCES cliente(id),
    CONSTRAINT fk_consumo_servicio FOREIGN KEY (servicio_id) REFERENCES servicio(id),
    CONSTRAINT fk_consumo_factura FOREIGN KEY (factura_id) REFERENCES factura(id)
);

CREATE TABLE estado_evento (
    id     INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL UNIQUE
);

CREATE TABLE evento (
    id                INT AUTO_INCREMENT PRIMARY KEY,
    nombre            VARCHAR(150) NOT NULL,
    cliente_id        INT,
    unidad_id         INT,
    estado_evento_id  INT,
    tipo_evento       VARCHAR(100),
    fecha             DATE NOT NULL,
    hora_inicio       TIME,
    hora_fin          TIME,
    cantidad_personas INT,
    precio            DECIMAL(10,2),
    observaciones     TEXT,
    CONSTRAINT fk_evento_cliente FOREIGN KEY (cliente_id) REFERENCES cliente(id),
    CONSTRAINT fk_evento_unidad  FOREIGN KEY (unidad_id) REFERENCES unidad(id),
    CONSTRAINT fk_evento_estado  FOREIGN KEY (estado_evento_id) REFERENCES estado_evento(id)
);
```
## 8. Configuración
Parámetros generales del sistema.
```
CREATE TABLE configuracion (
    id          INT AUTO_INCREMENT PRIMARY KEY,
    clave       VARCHAR(50) NOT NULL UNIQUE,
    valor       VARCHAR(255) NOT NULL,
    descripcion TEXT
);
```
## Conclusión
Esta base de datos está diseñada para cubrir todas las operaciones clave de un sistema hotelero moderno:

Gestión de usuarios y seguridad
Administración de clientes
Control de habitaciones
Reservas en tiempo real
Facturación integrada
Servicios y eventos
