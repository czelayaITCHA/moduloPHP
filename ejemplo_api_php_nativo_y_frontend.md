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
* Instalar **nvm** version 1.2.2 <br>
* Instalar la version 22.18.0 LTS de node 
```js
nvm install 22.18.0
```
verifique que tiene instalado la versión 22 de node, con la siguiente linea de comandos:
<img width="333" height="40" alt="image" src="https://github.com/user-attachments/assets/af243f71-5ad0-4d27-b555-e1a4f039140f" />

### 9.2 Crear proyecto react
Siga los pasos de la documentación oficial de react https://tailwindcss.com/docs/installation/using-vite
- crear el proyecto en la linea de comandos
  ```js
  npm create vite@latest products-app -- --template react
  ```
<img width="702" height="364" alt="image" src="https://github.com/user-attachments/assets/8f9a2276-dac6-434b-9fb5-8bd38e077ccf" />

- Se crea una carpeta con el nombre del proyecto, en este caso **products-app** 
- Cambiarse a la carpeta products-app
- instale dependencias
```js
npm install
```
- Abra el proyecto con visual code
  Estando dentro de la carpeta del proyecto digite
  ```batch
  code .
  ```
### 9.2 integrar tailwindcss al proyecto react
- Instalar dependencias de tailwind
  ```js
  npm install tailwindcss @tailwindcss/vite
  ```
- Configurar el plugin de vite, para ello editar el archivo **vite.config.js**, debe quedar de la siguiente manera:
<img width="482" height="231" alt="image" src="https://github.com/user-attachments/assets/60e5a952-bedc-4824-b13b-5da04f66df74" />
  
- Importar tailwindcss al archivo **index.css** del proyecto react
  ```js
  @import "tailwindcss";
  ```
- 

### 9.3 Crear las carpetas **services** y **components** dentro de **src**
<img width="248" height="501" alt="image" src="https://github.com/user-attachments/assets/ab9a5de4-ddde-4210-a739-b7c440c9449b" />

### 9.4 Instalar dependencias de react-icons y SweatAlert2
```js
npm install react-icons sweetalert2
```
### 9.5 crear el servicio serviceProducto.js dentro de la carpeta service
```js
const URL_BASE = "http://localhost/products-api/controllers/ProductoController.php";

export const getProductos = async () => {
  const resp = await fetch(URL_BASE);
  if (!resp.ok) throw new Error("Error al obtener los productos");
  return resp.json();
};

export const getProductoById = async (id) => {
  const resp = await fetch(`${URL_BASE}/${id}`);
  if (!resp.ok) throw new Error("Producto no encontrado");
  return resp.json();
};

export const saveProduct = async (data) => {
  const resp = await fetch(URL_BASE, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(data),
  });
  if (!resp.ok) throw new Error("Error al guardar el producto");
  return resp.json();
};

export const updateProduct = async (id, data) => {
  const resp = await fetch(`${URL_BASE}/${id}`, {
    method: "PUT",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(data),
  });
  if (!resp.ok) throw new Error("Error al actualizar el producto");
  return resp.json();
};

export const deleteProduct = async (id) => {
  const resp = await fetch(`${URL_BASE}/${id}`, {
    method: "DELETE",
  });
  if (!resp.ok) throw new Error("Error al eliminar el producto");
  return resp.json();
};
```
### 9.6 Crear el componente ProductoModal.jsx, dentro de la carpeta components

```js
import { useEffect, useState } from "react";

export default function ProductoModal({ open, onClose, onSave, producto }) {
  const [form, setForm] = useState({
    nombre: "",
    descripcion: "",
    precio: "",
    stock: "",
  });

  useEffect(() => {
    producto
      ? setForm(producto)
      : setForm({ nombre: "", descripcion: "", precio: "", stock: "" });
  }, [producto]);

  if (!open) return null;

  const handleChange = (e) =>
    setForm({ ...form, [e.target.name]: e.target.value });

  const handleSubmit = (e) => {
    e.preventDefault();
    onSave(form);
  };

  return (
    <div className="fixed inset-0 bg-black/40 flex items-center justify-center z-50">
      <div className="bg-white rounded-xl w-full max-w-lg p-6 shadow-lg">
        <h3 className="text-xl font-semibold mb-4">
          {producto ? "Editar Producto" : "Nuevo Producto"}
        </h3>

        <form onSubmit={handleSubmit} className="space-y-4">
          <input
            name="nombre"
            placeholder="Nombre"
            value={form.nombre}
            onChange={handleChange}
            className="w-full border rounded px-3 py-2"
            required
          />

          <textarea
            name="descripcion"
            placeholder="Descripción"
            rows={3}
            value={form.descripcion}
            onChange={handleChange}
            className="w-full border rounded px-3 py-2"
            required
          />

          <div className="grid grid-cols-2 gap-4">
            <input
              name="precio"
              type="number"
              placeholder="Precio"
              value={form.precio}
              onChange={handleChange}
              className="border rounded px-3 py-2"
              required
            />
            <input
              name="stock"
              type="number"
              placeholder="Stock"
              value={form.stock}
              onChange={handleChange}
              className="border rounded px-3 py-2"
              required
            />
          </div>

          <div className="flex justify-end gap-3 pt-4">
            <button
              type="button"
              onClick={onClose}
              className="px-4 py-2 rounded bg-gray-200"
            >
              Cancelar
            </button>
            <button
              type="submit"
              className="px-4 py-2 rounded bg-blue-600 text-white"
            >
              Guardar
            </button>
          </div>
        </form>
      </div>
    </div>
  );
}

```
### 9.7 crear el componente ProductoTable.jsx, dentro de la carpeta components 

```js
import { FaEdit, FaTrash } from "react-icons/fa";

export default function ProductoTable({ productos, onEdit, onDelete }) {
  return (
    <div className="overflow-x-auto mt-4 border border-gray-300 rounded-lg">
      <table className="w-full border-collapse">
        <thead className="bg-gray-100 text-gray-700">
          <tr>
            <th className="p-3 border border-gray-300 text-left">Nombre</th>
            <th className="p-3 border border-gray-300 text-left">Descripción</th>
            <th className="p-3 border border-gray-300 text-right">Precio</th>
            <th className="p-3 border border-gray-300 text-right">Stock</th>
            <th className="p-3 border border-gray-300 text-center">Acciones</th>
          </tr>
        </thead>

        <tbody>
          {productos.length === 0 && (
            <tr>
              <td
                colSpan="6"
                className="p-4 border border-gray-300 text-center text-gray-500"
              >
                No hay productos
              </td>
            </tr>
          )}

          {productos.map((p) => (
            <tr key={p.id} className="hover:bg-gray-50 transition">
              <td className="p-3 border border-gray-300 font-medium">
                {p.nombre}
              </td>
              <td className="p-3 border border-gray-300 text-sm text-gray-600">
                {p.descripcion}
              </td>
              <td className="p-3 border border-gray-300 text-right">
                ${p.precio}
              </td>
              <td className="p-3 border border-gray-300 text-right">
                {p.stock}
              </td>
              <td className="p-3 border border-gray-300">
                <div className="flex justify-center gap-3">
                  <button
                    onClick={() => onEdit(p)}
                    className="text-yellow-600 hover:text-yellow-800"
                    title="Editar"
                  >
                    <FaEdit />
                  </button>
                  <button
                    onClick={() => onDelete(p.id)}
                    className="text-red-600 hover:text-red-800"
                    title="Eliminar"
                  >
                    <FaTrash />
                  </button>
                </div>
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

### 9.8 crear el archivo Productos.jsx en la carpeta components
 ```js
import { useEffect, useState } from "react";
import Swal from "sweetalert2";
import {
  getProductos,
  saveProduct,
  updateProduct,
  deleteProduct,
} from "../services/productoService";
import ProductoTable from "./ProductoTable";
import ProductoModal from "./ProductoModal";

export default function Productos() {
  const [productos, setProductos] = useState([]);
  const [search, setSearch] = useState("");
  const [openModal, setOpenModal] = useState(false);
  const [productoEdit, setProductoEdit] = useState(null);

  const loadProductos = async () => {
    const data = await getProductos();
    setProductos(data);
  };

  useEffect(() => {
    loadProductos();
  }, []);

  const handleSave = async (data) => {
    try {
      productoEdit
        ? await updateProduct(productoEdit.id, data)
        : await saveProduct(data);

      Swal.fire("Éxito", "Producto guardado correctamente", "success");
      setOpenModal(false);
      setProductoEdit(null);
      loadProductos();
    } catch (e) {
      Swal.fire("Error", e.message, "error");
    }
  };

  const handleDelete = async (id) => {
    const result = await Swal.fire({
      title: "¿Eliminar producto?",
      text: "Esta acción no se puede deshacer",
      icon: "warning",
      showCancelButton: true,
      confirmButtonText: "Sí, eliminar",
      cancelButtonText: "Cancelar",
    });

    if (result.isConfirmed) {
      await deleteProduct(id);
      Swal.fire("Eliminado", "Producto eliminado", "success");
      loadProductos();
    }
  };

  const productosFiltrados = productos.filter((p) =>
    p.nombre?.toLowerCase().includes(search.toLowerCase())
  );

  return (
    <div className="p-8 max-w-6xl mx-auto">
      <div className="flex justify-between items-center mb-6">
        <h1 className="text-2xl font-bold">Catálogo de Productos</h1>
        <button
          onClick={() => {
            setProductoEdit(null);
            setOpenModal(true)}}
          className="bg-blue-600 text-white px-4 py-2 rounded"
        >
          + Agregar
        </button>
      </div>

      <input
        placeholder="Buscar producto..."
        value={search}
        onChange={(e) => setSearch(e.target.value)}
        className="border p-2 w-full max-w-sm mb-4"
      />

      <ProductoTable
        productos={productosFiltrados}
        onEdit={(p) => {
          setProductoEdit(p);
          setOpenModal(true);
        }}
        onDelete={handleDelete}
      />

      <ProductoModal
        open={openModal}
        producto={productoEdit}
        onClose={() => {
          setOpenModal(false);
          setProductoEdit(null);
        }}
        onSave={handleSave}
      />
    </div>
  );
}

```
### 9.9 inyectar el componente Productos.jsx en el componente principal App.jsx
```js
import './App.css'
import Productos from './components/Productos'
function App() {
  return <Productos />;

}

export default App
``` 
## 10 Resultado 
<img width="959" height="340" alt="image" src="https://github.com/user-attachments/assets/092f2763-65d0-4cfd-af48-0a2d02d823ba" />
