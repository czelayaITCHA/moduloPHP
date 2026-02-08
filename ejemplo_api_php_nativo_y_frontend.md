# Ejemplo API con PHP nativo y React
En el siguiente documento propongo una API REST de productos en PHP 8 nativo, organizada en Modelo – DAO – Controller, usando PDO, respuestas JSON, y preparada para CORS.

## 1. Crear base de datos products_db y tabla productos en MySQL
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

## 2. Crear estructura de proyecto en visual studio code como se muestra en la siguiente imagen, debe crearse en la carpeta **WWW** de WampServer
<img width="313" height="167" alt="image" src="https://github.com/user-attachments/assets/82e34db4-3d0c-465a-8bdc-189519b9989e" />

## 3. Crear clase en la carpeta config para definir conexión con la base de datos 
```php
<?php
class Database {
    private string $host = "localhost";
    private string $db_name = "products_db";
    private string $username = "root";
    private string $password = "";
    public ?PDO $conn = null;

    public function connect(): ?PDO {
        try {
            $this->conn = new PDO(
                "mysql:host={$this->host};dbname={$this->db_name};charset=utf8",
                $this->username,
                $this->password
            );
            $this->conn->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
            return $this->conn;
        } catch (PDOException $e) {
            echo json_encode([
                "error" => "Error de conexión",
                "message" => $e->getMessage()
            ]);
            return null;
        }
    }
}

```
## 4. En la carpeta models, crear la clase Producto para representar los datos

```php
<?php
class Producto {
    public ?int $id;
    public string $nombre;
    public string $descripcion;
    public float $precio;
    public int $stock;

    public function __construct(
        ?int $id,
        string $nombre,
        string $descripcion,
        float $precio,
        int $stock
    ) {
        $this->id = $id;
        $this->nombre = $nombre;
        $this->descripcion = $descripcion;
        $this->precio = $precio;
        $this->stock = $stock;
    }
}

```
## 5. En la carpeta **dao**, crear la clase **ProductoDAO**, para realizar las operaciones CRUD en la tabla productos
```
<?php
require_once __DIR__ . '/../models/Producto.php';

class ProductoDAO {
    private PDO $conn;

    public function __construct(PDO $conn) {
        $this->conn = $conn;
    }

    public function findAll(): array {
        $stmt = $this->conn->query("SELECT * FROM productos");
        return $stmt->fetchAll(PDO::FETCH_ASSOC);
    }

    public function findById(int $id): ?array {
        $stmt = $this->conn->prepare("SELECT * FROM productos WHERE id = ?");
        $stmt->execute([$id]);
        $producto = $stmt->fetch(PDO::FETCH_ASSOC);
        return $producto ?: null;
    }

    public function create(Producto $producto): bool {
        $sql = "INSERT INTO productos (nombre, descripcion, precio, stock)
                VALUES (:nombre, :descripcion, :precio, :stock)";
        $stmt = $this->conn->prepare($sql);
        return $stmt->execute([
            ":nombre" => $producto->nombre,
            ":descripcion" => $producto->descripcion,
            ":precio" => $producto->precio,
            ":stock" => $producto->stock
        ]);
    }

    public function update(int $id, Producto $producto): bool {
        $sql = "UPDATE productos
                SET nombre = :nombre,
                    descripcion = :descripcion,
                    precio = :precio,
                    stock = :stock
                WHERE id = :id";
        $stmt = $this->conn->prepare($sql);
        return $stmt->execute([
            ":nombre" => $producto->nombre,
            ":descripcion" => $producto->descripcion,
            ":precio" => $producto->precio,
            ":stock" => $producto->stock,
            ":id" => $id
        ]);
    }

    public function delete(int $id): bool {
        $stmt = $this->conn->prepare("DELETE FROM productos WHERE id = ?");
        return $stmt->execute([$id]);
    }
}

```
