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

# 4️⃣ MarcaController Completo

```php
namespace App\Http\Controllers;

use App\Models\Marca;
use Illuminate\Http\Request;

class MarcaController extends Controller
{
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
                'estado' => true,
                'marcas' => $marcas
            ], 200);
        }catch(\Exception $e){
            return response()->json([
                'message' => 'Error al obtener las marcas',
                'error' => $e->getMessage()
            ],500);
        }
    }

    public function store(Request $request)
    {
        try {
            $request->validate([
                'nombre' => 'required|unique:marcas,nombre|max:80'
            ]);

            $marca = Marca::create($request->all());

            return response()->json([
                'message' => 'Marca creada correctamente',
                'data' => $marca
            ], 201);
        } catch (\Exception $e) {
            return response()->json([
                'message' => 'Error al crear marca',
                'error' => $e->getMessage()
            ], 500);
        }
    }

    public function show($id)
    {
        try {
            $marca = Marca::findOrFail($id);

            return response()->json($marca, 200);
        } catch (\Exception $e) {
            return response()->json([
                'message' => 'Marca no encontrada',
                'error' => $e->getMessage()
            ], 500);
        }
    }

    public function update(Request $request, $id)
    {
        try {
            $marca = Marca::findOrFail($id);

            $request->validate([
                'nombre' => 'required|unique:marcas,nombre,' . $id
            ]);

            $marca->update($request->all());

            return response()->json([
                'message' => 'Marca actualizada',
                'data' => $marca
            ], 202);
        } catch (\Exception $e) {
            return response()->json([
                'message' => 'Error al actualizar',
                'error' => $e->getMessage()
            ], 500);
        }
    }

    public function destroy($id)
    {
        try {
            $marca = Marca::findOrFail($id);
            $marca->delete();

            return response()->json([
                'message' => 'Marca eliminada'
            ], 200);
        } catch (\Exception $e) {
            return response()->json([
                'message' => 'Error al eliminar',
                'error' => $e->getMessage()
            ], 500);
        }
    }
}
```

---
