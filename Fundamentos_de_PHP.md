# Fundamentos de PHP 8

## Introducción a PHP 8
PHP es un lenguaje de programación de propósito general enfocado en el desarrollo web del lado del servidor. En esta guía aprenderemos los fundamentos de PHP 8, incluyendo variables, constantes, tipos de datos, estructuras de control, programación orientada a objetos y peticiones asíncronas con `fetch` en JavaScript.

---

## 1. Variables y Constantes

### Variables
Las variables en PHP comienzan con `$` y su tipo de dato se asigna dinámicamente.
```php
$nombre = "Juan";
$edad = 25;
echo "Nombre: $nombre, Edad: $edad";
```

### Constantes
Se definen con `define` o `const`.
```php
define("PI", 3.1416);
const NOMBRE = "PHP";
echo PI;
```

**Ejercicio:** Crea un script que defina una variable con tu nombre y una constante con tu edad, e imprímelas.

---

## 2. Tipos de Datos y Tipado en PHP
PHP es un lenguaje de tipado dinámico y débilmente tipado, lo que significa que las variables pueden cambiar de tipo durante la ejecución del programa y no requieren declaración explícita del tipo.

```php
$valor = 10; // Entero
$valor = "Hola"; // Ahora es un string
```

Sin embargo, desde PHP 7 se pueden declarar tipos de datos estrictos.
```php
declare(strict_types=1);
function suma(int $a, int $b): int {
    return $a + $b;
}
echo suma(5, "10"); // Error si strict_types está activado
```

**Ejercicio:** Crea una función que acepte solo números enteros y devuelva su suma.

---

## 3. Arreglos

```php
$frutas = ["Manzana", "Banana", "Cereza"];
echo $frutas[1]; // Banana
```

**Ejercicio:** Crea un arreglo con 5 nombres y recórrelo con un `foreach`.

---

## 4. Estructuras de Control

### Condicionales
```php
$edad = 18;
if ($edad >= 18) {
    echo "Eres mayor de edad";
} else {
    echo "Eres menor de edad";
}
```

### Bucles
```php
for ($i = 0; $i < 5; $i++) {
    echo "$i ";
}
```

**Ejercicio:** Crea un programa que determine si un número es par o impar.

---

## 5. Programación Orientada a Objetos (POO) en PHP

PHP es un lenguaje que soporta POO con clases, objetos, herencia y encapsulación.

### Clases y Objetos
```php
class Persona {
    private $nombre;
    
    public function __construct($nombre) {
        $this->nombre = $nombre;
    }
    
    public function saludar() {
        return "Hola, soy " . $this->nombre;
    }
}

$persona = new Persona("Carlos");
echo $persona->saludar();
```

### Herencia
```php
class Empleado extends Persona {
    private $salario;
    
    public function __construct($nombre, $salario) {
        parent::__construct($nombre);
        $this->salario = $salario;
    }
    
    public function mostrarSalario() {
        return "Mi salario es " . $this->salario;
    }
}

$empleado = new Empleado("Ana", 3000);
echo $empleado->saludar();
echo $empleado->mostrarSalario();
```

**Ejercicio:** Crea una clase `Coche` con propiedades `marca` y `modelo`, y un método para mostrarlas.

---

## 6. Peticiones Asíncronas con Fetch en JavaScript

### Ejemplo de Fetch GET
```js
fetch('https://jsonplaceholder.typicode.com/posts')
    .then(response => response.json())
    .then(data => console.log(data))
    .catch(error => console.error('Error:', error));
```

### Ejemplo de Fetch POST con PHP
```js
fetch('procesar.php', {
    method: 'POST',
    headers: {'Content-Type': 'application/json'},
    body: JSON.stringify({nombre: "Juan", edad: 30})
})
.then(response => response.json())
.then(data => console.log(data));
```

### Procesando en PHP
```php
$data = json_decode(file_get_contents("php://input"), true);
echo json_encode(["mensaje" => "Hola, " . $data["nombre"]]);
```

**Ejercicio:** Crea un formulario HTML que envíe datos a un script PHP usando `fetch`.

---

## Conclusión
Esta guía cubre los fundamentos esenciales de PHP 8. ¡Sigue practicando y creando proyectos para mejorar tus habilidades!
