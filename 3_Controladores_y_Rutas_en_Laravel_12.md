# Controladores en Laravel 12 (API REST)

Esta guía explica TODO lo relacionado con controladores en Laravel 12 y creación de rutas:

- try {} catch {}
- Respuestas JSON estructuradas
- Status codes correctos (201, 202, 200, 500)
- Método show($id)
- Manejo de imágenes en Producto
- Transacciones en Order

---

# 1️ ¿Qué es un Controlador en Laravel?

Un controlador es una clase que maneja las peticiones HTTP.

Ubicación:

```
app/Http/Controllers
```

Responsabilidades:

- Recibir Request
- Validar datos
- Ejecutar lógica de negocio
- Consultar modelos
- Devolver respuestas JSON

---

# 2️ Crear los Controladores

Ejecutar desde la raíz del proyecto:

```bash
php artisan make:controller MarcaController --api
php artisan make:controller ProductoController --api
php artisan make:controller OrderController --api
```

Se crean en:

```
app/Http/Controllers/
```

---

# 3️ Rutas API
El archivo api.php no viene habilitado a partir de la versión 11, por lo que debe instalarse con:
```bash
php artisan install:api
```
Editar:

```
routes/api.php
```

```php
use App\Http\Controllers\MarcaController;
use App\Http\Controllers\ProductoController;
use App\Http\Controllers\OrderController;

Route::apiResource('marcas', MarcaController::class);
Route::apiResource('productos', ProductoController::class);
Route::apiResource('orders', OrderController::class);
```

---

# 4️ Programar controlador MarcaController Completo

Al crear el controlador con **php artisan make:controller MarcaController --api***, se crea la estructura básica de un controlador 
```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class MarcaController extends Controller
{
    /**
     * Display a listing of the resource.
     */
    public function index()
    {
        //
    }

    /**
     * Store a newly created resource in storage.
     */
    public function store(Request $request)
    {
        //
    }

    /**
     * Display the specified resource.
     */
    public function show(string $id)
    {
        //
    }

    /**
     * Update the specified resource in storage.
     */
    public function update(Request $request, string $id)
    {
        //
    }

    /**
     * Remove the specified resource from storage.
     */
    public function destroy(string $id)
    {
        // 
    }

}

```
## 4.1 importar espacios de nombres necesarios para validaciones y el modelo Marca
```php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Validation\Rule;
use Illuminate\Validation\ValidationException;
use Illuminate\Database\Eloquent\ModelNotFoundException;
use App\Models\Marca; 

class MarcaController extends Controller
{
    /**
     * Display a listing of the resource.
     */
    public function index()
    {
        //
    }

    /**
     * Store a newly created resource in storage.
     */
    public function store(Request $request)
    {
        //
    }

    /**
     * Display the specified resource.
     */
    public function show(string $id)
    {
        //
    }

    /**
     * Update the specified resource in storage.
     */
    public function update(Request $request, string $id)
    {
        //
    }

    /**
     * Remove the specified resource from storage.
     */
    public function destroy(string $id)
    {
        // 
    }

}
```
## 4.2 Programar la lógica del método index, para obtener la colección o lista de marcas registradas en la base de datos
```php
 public function index()
    {
        try{
            // Forma básica
            // $marcas = Marca::all();

            // Ordenadas
            $marcas = Marca::orderBy('id','desc')->get();

            // Paginadas
            // $marcas = Marca::orderBy('id','desc')->paginate(10);

            return response()->json([
                'marcas' => $marcas
            ], 200);
        }catch(\Exception $e){
            return response()->json([
                'message' => 'Error al obtener las marcas',
                'error' => $e->getMessage()
            ],500);
        }
    }
```
---
