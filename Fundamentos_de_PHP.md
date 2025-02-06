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

## 5.1. Introducción a la POO en PHP
La Programación Orientada a Objetos (POO) es un paradigma que organiza el código en torno a "objetos", que son instancias de "clases". PHP soporta completamente la POO, lo que permite escribir código modular, reutilizable y más mantenible.

## 5.2. Conceptos Claves de la POO
### a) Clases y Objetos
Las clases son plantillas para crear objetos. Los objetos son instancias de una clase.

**Ejemplo:**
```php
class Persona {
    private $nombre;
    private $edad;
    
    public function __construct($nombre, $edad) {
        $this->nombre = $nombre;
        $this->edad = $edad;
    }
    
    public function saludar() {
        return "Hola, mi nombre es $this->nombre y tengo $this->edad años.";
    }
}

$persona1 = new Persona("Juan", 30);
echo $persona1->saludar();
```

### b) Propiedades y Métodos
Las propiedades son variables dentro de una clase, y los métodos son funciones dentro de una clase.

### c) Encapsulamiento
El encapsulamiento protege los datos de una clase. Se define con:
- `public`: Accesible desde cualquier parte.
- `private`: Accesible solo dentro de la clase.
- `protected`: Accesible dentro de la clase y sus subclases.

**Ejemplo:**
```php
class CuentaBancaria {
    private $saldo;
    
    public function __construct($saldoInicial) {
        $this->saldo = $saldoInicial;
    }
    
    public function depositar($monto) {
        $this->saldo += $monto;
    }
    
    public function obtenerSaldo() {
        return $this->saldo;
    }
}

$cuenta = new CuentaBancaria(1000);
$cuenta->depositar(500);
echo "Saldo actual: " . $cuenta->obtenerSaldo();
```

### d) Herencia
Permite que una clase herede propiedades y métodos de otra.

**Ejemplo:**
```php
class Empleado extends Persona {
    private $puesto;
    
    public function __construct($nombre, $edad, $puesto) {
        parent::__construct($nombre, $edad);
        $this->puesto = $puesto;
    }
    
    public function obtenerPuesto() {
        return $this->puesto;
    }
}

$empleado = new Empleado("Ana", 25, "Desarrollador");
echo $empleado->saludar() . " y soy " . $empleado->obtenerPuesto();
```

### e) Polimorfismo
Permite redefinir métodos de una clase padre en una clase hija.

**Ejemplo:**
```php
class Animal {
    public function hacerSonido() {
        return "Sonido genérico";
    }
}

class Perro extends Animal {
    public function hacerSonido() {
        return "Guau Guau";
    }
}

$miPerro = new Perro();
echo $miPerro->hacerSonido();
```

### f) Interfaces
Las interfaces definen métodos que una clase debe implementar.

**Ejemplo:**
```php
interface Vehiculo {
    public function acelerar();
    public function frenar();
}

class Coche implements Vehiculo {
    public function acelerar() {
        return "El coche está acelerando";
    }
    
    public function frenar() {
        return "El coche está frenando";
    }
}

$miCoche = new Coche();
echo $miCoche->acelerar();
```

### g) Clases y Métodos Estáticos
Los métodos estáticos pueden llamarse sin crear una instancia de la clase.

**Ejemplo:**
```php
class Utilidades {
    public static function mensaje() {
        return "Este es un mensaje estático.";
    }
}

echo Utilidades::mensaje();
```
## 5.3 Ejemplo de cálculo de tabla de amortizaciones de un crédito
### 5.3.1. Crear un formulario para definir parámetros para generar tabla de amortizaciones
```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ejemplo con POO en PHP</title>
    <style>
         table{
            width: 100%;
            border-collapse: collapse;
            margin-top: 20px;
        }
        th,td{
            border: 1px solid gray;
            padding: 8px;
            text-align: left;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        th{
            background-color: #f4f4f4;
        }
    </style>
    
</head>
<body>
    <h2>Calculadora de créditos - POO en PHP</h2>
    <form id="formCredito">
        <table>
            <tr>
                <td>
                    <label for="">Monto solicitado: </label>
                    $<input type="number" id="monto" required>
                </td>
                <td>
                    <label for="">Iva (13%): </label>
                    $<input type="text" id="iva" readonly>
                </td>
                <td>
                    <label for="">Monto Total: </label>
                    $<input type="text" id="montoTotal" readonly>
                </td>
            </tr>
            <tr>
                <td>
                    <label for="">Plazo (Meses):</label>
                    <select id="plazo">
                        <option value="36">36 Meses</option>
                        <option value="60">60 Meses</option>
                        <option value="72">72 Meses</option>
                        <option value="96">96 Meses</option>
                        <option value="120">120 Meses</option>
                    </select>
                </td>
                <td colspan="2">
                    <label for="">Interes Anual (%):</label>
                    <input type="number" id="interesAnual" required>
                </td>
            </tr>
            <tr>
                <td colspan="3" style="text-align: center;">
                    <button type="submit">Calcular Crédito</button>
                </td>
            </tr>
        </table>
    </form>
    <h3>Tabla de Amortizaciones</h3>
    <table id="tablaAmortizacion">
        <thead>
            <tr>
                <th>Mes</th>
                <th>Cuota</th>
                <th>Interés</th>
                <th>Abono a Capital</th>
                <th>Saldo</th>
            </tr>
        </thead>
        <tbody></tbody>
    </table>
    <script>
        //obtenemos los id de los input
        const montoInput = document.getElementById("monto");
        const ivaInput = document.getElementById("iva");
        const montoTotalInput = document.getElementById("montoTotal");

        montoInput.addEventListener("input", ()=>{
            //calculo de iva y montoTotal
            let monto = parseFloat(montoInput.value) || 0;
            let iva = monto * 0.13;
            let total = monto + iva;
            //asignamos los valores a los input
            ivaInput.value = iva.toFixed(2);
            montoTotalInput.value = total.toFixed(2);
        });

        //manejando el evento submit para hacer la peticion
        document.getElementById("formCredito").addEventListener(
            "submit", async (event) =>{
                event.preventDefault();
                //obtenemos los valores de monto, plazo e interes
                let montoTotal = parseFloat(montoTotalInput.value);
                let meses = parseInt(document.getElementById("plazo").value);
                let interesAnual = parseFloat(document.getElementById("interesAnual").value);

                //hacemos la petición con fetch
                const response = await fetch(`calcular_credito.php`,{
                    method: 'POST',
                    headers:{"Content-Type" : "application/json"},
                    body: JSON.stringify({ montoTotal, meses, interesAnual })
                });
                const amortizaciones = await response.json();
                const tbody = document.querySelector("#tablaAmortizacion tbody");
                tbody.innerHTML = "";
                amortizaciones.forEach(item =>{
                    let fila = document.createElement("tr");
                    fila.innerHTML = `
                        <td>${item.mes}</td>
                        <td>$${item.cuota.toFixed(2)}</td>
                        <td>$${item.interes.toFixed(2)}</td>
                        <td>$${item.abonoCapital.toFixed(2)}</td>
                        <td>$${item.saldo.toFixed(2)}</td>
                    `;
                    tbody.appendChild(fila);
                });
            }
        )
    </script>
</body>
</html>
```

## 5.4. Conclusión
La POO en PHP permite escribir código estructurado, reutilizable y más fácil de mantener. Con estos conceptos básicos, puedes desarrollar aplicaciones más robustas y escalables.


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
