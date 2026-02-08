# Ejemplo API con PHP nativo y react
En el siguiente documento propongo una API REST de productos en PHP 8 nativo, organizada en Modelo – DAO – Controller, usando PDO, respuestas JSON, y preparada para CORS.

## 1. Crear base de datos products_db en MySQL y tabla de productos
Inicia **WampServer**, accede a la aplicación **phpMyAdmin**, haz click en la pestaña SQL y pega el siguiente código para crear base de datos y tabla
```sql
CREATE DATABASE products_db
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;

USE products_db;

CREATE TABLE productos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(150) NOT NULL,
    descripcion TEXT,
    precio DECIMAL(10,2) NOT NULL,
    stock INT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB;
```
tal como se muestra en la siguiente imágen:
<img width="1364" height="664" alt="image" src="https://github.com/user-attachments/assets/341d3872-4287-43a6-876f-d7928fb515e3" />

La estructura de la tabla de productos debe quedar como se muestra en la imagen siguiente:
<img width="1230" height="620" alt="image" src="https://github.com/user-attachments/assets/6fcb61e2-b2ae-45ac-bfa3-642055812756" />


##
##
