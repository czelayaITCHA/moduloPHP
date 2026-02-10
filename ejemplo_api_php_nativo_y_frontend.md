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

## 3. Crear clase en la carpeta config para definir conexión con la base de datos, llamar al archvivo **Database.php**
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
## 4. En la carpeta models, crear la clase Producto para representar los datos, el archivo debe tener el nombre Producto.php

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
## 5. En la carpeta **dao**, crear la clase **ProductoDAO**, para realizar las operaciones CRUD en la tabla productos, llamar al archivo ProductoDAO.php
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
## 6. En la carpeta **controllers**, crear el archivo **ProductoController.php**, para programar los endpoints de la API
```php
<?php
header("Access-Control-Allow-Origin: *");
header("Access-Control-Allow-Headers: Content-Type");
header("Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS");
header("Content-Type: application/json");

if ($_SERVER["REQUEST_METHOD"] === "OPTIONS") {
    http_response_code(200);
    exit;
}

require_once __DIR__ . '/../config/Database.php';
require_once __DIR__ . '/../dao/ProductoDAO.php';
require_once __DIR__ . '/../models/Producto.php';

class ProductoController {
    private ProductoDAO $dao;

    public function __construct(ProductoDAO $dao) {
        $this->dao = $dao;
    }

    public function index() {
        echo json_encode($this->dao->findAll());
    }

    public function show(int $id) {
        $producto = $this->dao->findById($id);
        if (!$producto) {
            http_response_code(404);
            echo json_encode(["message" => "Producto no encontrado"]);
            return;
        }
        echo json_encode($producto);
    }

    public function store() {
        $data = json_decode(file_get_contents("php://input"), true);

        $producto = new Producto(
            null,
            $data["nombre"],
            $data["descripcion"],
            (float)$data["precio"],
            (int)$data["stock"]
        );

        if ($this->dao->create($producto)) {
            http_response_code(201);
            echo json_encode(["message" => "Producto creado"]);
        }
    }

    public function update(int $id) {
        $data = json_decode(file_get_contents("php://input"), true);

        $producto = new Producto(
            $id,
            $data["nombre"],
            $data["descripcion"],
            (float)$data["precio"],
            (int)$data["stock"]
        );

        if ($this->dao->update($id, $producto)) {
            echo json_encode(["message" => "Producto actualizado"]);
        }
    }

    public function destroy(int $id) {
        if ($this->dao->delete($id)) {
            echo json_encode(["message" => "Producto eliminado"]);
        }
    }
}

/* ==========================
   BOOTSTRAP + ROUTING
   ========================== */

$db = new Database();
$conn = $db->connect();
$dao = new ProductoDAO($conn);
$controller = new ProductoController($dao);

$method = $_SERVER["REQUEST_METHOD"];
$uri = explode("/", trim($_SERVER["REQUEST_URI"], "/"));

// Ajusta el índice según la carpeta donde esté el proyecto
$id = $uri[count($uri) - 1];
$id = is_numeric($id) ? (int)$id : null;

switch ($method) {
    case "GET":
        $id ? $controller->show($id) : $controller->index();
        break;

    case "POST":
        $controller->store();
        break;

    case "PUT":
        $controller->update($id);
        break;

    case "DELETE":
        $controller->destroy($id);
        break;
}

```
## 7. Crear el archivo .htaccess a nivel de carpeta principal
```xml
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^ index.php [QSA,L]
```
## 8. Probar API en Postman

## 9. Crear el proyecto frontend
### 9.1 Instalar la version 22 de node
Instalar **nvm** version 1.2.2
instalar la version 22.18.0 LTS de node 
```js
node install 22.18.0
```
### 9.2 Crear proyecto react

<img width="702" height="364" alt="image" src="https://github.com/user-attachments/assets/8f9a2276-dac6-434b-9fb5-8bd38e077ccf" />

Se crea una carpeta con el nombre del proyecto, en este caso **products-app** 
cambiar se a la carpeta products-app
instale dependencias
```js
npm install
```

### 9.2 integrar tailwindcss al proyecto react

