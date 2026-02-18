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

## 4.3 Programar método store para guardar un registro
```php
public function store(Request $request)
    {
        try{
            //validaciones a nivel de Request
            $request->validate(
                [
                    'nombre' => 'required|string|min:2|max:80|unique:marcas,nombre'
                ],
                [
                    'nombre.unique' => 'Ya existe una marca con este nombre en la base de datos'
                ]
            );

            $marca = Marca::create([
                'nombre' => $request->nombre
            ]);
            return response()->json([
                'message' => 'Marca registrada correctamente',
                'marca' =>  $marca
            ],201);
        } catch (ValidationException $e) {
            return response()->json([
                'message' => 'Error de validación.',
                'errores' => $e->errors()
            ], 422);

        } catch(\Exception $e){
              return response()->json([
                'message' => 'Error interno del servidor',
                'error' => $e->getMessage()
            ],500);
        }
    }
```
## 4.4 Programar lógica del método show() para obtener un registro u objeto por su id

```php
public function show(string $id)
    {
        try{
            $marca = Marca::findOrFail($id);
            return response()->json($marca);
        }catch (ModelNotFoundException $e) {
            return response()->json([
                'message' => 'Marca no encontrada, con ID = ' . $id
            ], 404);
        }
        catch(\Exception $e){
            return response()->json([
                'message' => 'Marca no encontrada',
                'error' => $e->getMessage()
            ],500);
        }
    }
```
## 4.5 Implementar método update para actualizar un registro existente
```php
public function update(Request $request, string $id)
    {
        try{
             //primero obtenemos el registro de la bd
            $marca = Marca::findOrFail($id);
            //aplicamos validaciones a nivel de request
            $request->validate(
                [
                    'nombre' => [
                        'required',
                        'string',
                        'min:2',
                        'max:80',
                        Rule::unique('marcas', 'nombre')->ignore($id)
                    ]
                ],
                [
                    'nombre.unique' => 'Ya existe una marca con este nombre en la base de datos'
                ]
            );

            //mandamos a actualizar el registro
            $marca->update([
                'nombre' =>$request->nombre
            ]);
            return response()->json([
                'message' => 'Marca actualizada correctamente',
                'marca' => $marca
            ],202);    
        }catch(\Exception $e){
             return response()->json([
                'message' => 'Marca no encontrada',
                'error' => $e->getMessage()
            ],500);
        }
    }
```
## 4.6 Programar el método destroy para eliminar un registro
```php
public function destroy(string $id)
    {
         try {
            $marca = Marca::with('productos')->findOrFail($id);

            if ($marca->productos()->exists()) {
                return response()->json([
                    'message' => 'No se puede eliminar la marca porque tiene productos asociados.'
                ], 409);
            }

            $marca->delete();

            return response()->json([
                'message' => 'Marca eliminada correctamente.'
            ], 200);

        } catch (ModelNotFoundException $e) {
            return response()->json([
                'message' => 'Marca no encontrada, con el ID = ' .$id
            ], 404);
        }
    }
```
## 4.7 El código del controlador completo de MarcaController, queda así:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Validation\Rule;
use Illuminate\Validation\ValidationException;
use Illuminate\Database\Eloquent\ModelNotFoundException;
use App\Models\Marca; // 1- importamos el o los modelos


class MarcaController extends Controller
{
    /**
     * Display a listing of the resource.
     */
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

    /**
     * Store a newly created resource in storage.
     */
    public function store(Request $request)
    {
        try{
            //validaciones a nivel de Request
            $request->validate(
                [
                    'nombre' => 'required|string|min:2|max:80|unique:marcas,nombre'
                ],
                [
                    'nombre.unique' => 'Ya existe una marca con este nombre en la base de datos'
                ]
            );

            $marca = Marca::create([
                'nombre' => $request->nombre
            ]);
            return response()->json([
                'message' => 'Marca registrada correctamente',
                'marca' =>  $marca
            ],201);
        } catch (ValidationException $e) {
            return response()->json([
                'message' => 'Error de validación.',
                'errores' => $e->errors()
            ], 422);

        } catch(\Exception $e){
              return response()->json([
                'message' => 'Error interno del servidor',
                'error' => $e->getMessage()
            ],500);
        }
    }

    /**
     * Display the specified resource.
     */
    public function show(string $id)
    {
        try{
            $marca = Marca::findOrFail($id);
            return response()->json($marca);
        }catch (ModelNotFoundException $e) {
            return response()->json([
                'message' => 'Marca no encontrada, con ID = ' . $id
            ], 404);
        }
        catch(\Exception $e){
            return response()->json([
                'message' => 'Marca no encontrada',
                'error' => $e->getMessage()
            ],500);
        }
    }

    /**
     * Update the specified resource in storage.
     */
    public function update(Request $request, string $id)
    {
        try{
             //primero obtenemos el registro de la bd
            $marca = Marca::findOrFail($id);
            //aplicamos validaciones a nivel de request
            $request->validate(
                [
                    'nombre' => [
                        'required',
                        'string',
                        'min:2',
                        'max:80',
                        Rule::unique('marcas', 'nombre')->ignore($id)
                    ]
                ],
                [
                    'nombre.unique' => 'Ya existe una marca con este nombre en la base de datos'
                ]
            );

            //mandamos a actualizar el registro
            $marca->update([
                'nombre' =>$request->nombre
            ]);
            return response()->json([
                'message' => 'Marca actualizada correctamente',
                'marca' => $marca
            ],202);    
        }catch(\Exception $e){
             return response()->json([
                'message' => 'Marca no encontrada',
                'error' => $e->getMessage()
            ],500);
        }
    }

    /**
     * Remove the specified resource from storage.
     */
    public function destroy(string $id)
    {
         try {
            $marca = Marca::with('productos')->findOrFail($id);

            if ($marca->productos()->exists()) {
                return response()->json([
                    'message' => 'No se puede eliminar la marca porque tiene productos asociados.'
                ], 409);
            }

            $marca->delete();

            return response()->json([
                'message' => 'Marca eliminada correctamente.'
            ], 200);

        } catch (ModelNotFoundException $e) {
            return response()->json([
                'message' => 'Marca no encontrada, con el ID = ' .$id
            ], 404);
        }
    }

}
```
## 5 Hacer pruebas en postman

El controlador anterior expone los diferentes endpoints para crear un CRUD de la tabla marcas y se pueden consumir por un proyecto frontend

## 6. Programar controlador ProductoController, con gestión de imágenes

Incluye:
* Validaciones robustas
* Subida de múltiples imágenes
* Eliminación física del storage
* Prevención de eliminación si existen órdenes
* Uso obligatorio de transacciones

### 6.1 Importar modelos y espacios de nombres necesarios
```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Models\Producto;
use App\Models\Imagen;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Storage;
use Illuminate\Validation\ValidationException;
use Illuminate\Database\Eloquent\ModelNotFoundException;
use Illuminate\Support\Facades\Validator;

class ProductoController extends Controller
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
### 6.2 Programar el método index() para obtener la colección de productos con sus relaciones
```php
public function index()
    {
        try {
            $productos = Producto::with(['marca','categoria','imagenes'])
                ->orderBy('id','desc')
                ->get();

            return response()->json($productos, 200);

        } catch (\Exception $e) {
            return response()->json([
                'message' => 'Error al obtener la lista de productos.'
            ], 500);
        }
    }
```
### 6.3 Programar el método **store()** para guardar un productos con imágenes asociadas
```php
 public function store(Request $request)
    {
        try{

            if (!$request->has('producto')) {
                return response()->json([
                    'message' => 'El objeto producto es requerido'
                ], 422);
            }
            // Decodificamos el JSON
            $productoData = json_decode($request->producto, true);

            if (!$productoData) {
                return response()->json([
                    'message' => 'El formato del JSON es inválido'
                ], 422);
            }
            // Normalizar estructura
            $data = [
                'nombre' => $productoData['nombre'] ?? null,
                'descripcion' => $productoData['descripcion'] ?? null,
                'precio' => $productoData['precio'] ?? null,
                'stock' => $productoData['stock'] ?? null,
                'modelo' => $productoData['modelo'] ?? null,
                'marca_id' => $productoData['marca']['id'] ?? null,
                'categoria_id' => $productoData['categoria']['id'] ?? null,
                'activo' => $productoData['activo'] ?? null,
            ];

            // Validaciones datos recibidos
            $validator = Validator::make($data, [
                'nombre' => 'required|string|max:80|unique:productos,nombre',
                'descripcion' => 'required|string|max:200',
                'precio' => 'required|numeric|min:0',
                'stock' => 'required|integer|min:0',
                'modelo' => 'required|string|max:50',
                'marca_id' => 'required|exists:marcas,id',
                'categoria_id' => 'required|exists:categorias,id',
                'activo' => 'required|boolean',
            ]);

            if ($validator->fails()) {
                return response()->json([
                    'message' => 'Errores de validación',
                    'errors' => $validator->errors()
                ], 422);
            }


            //iniciamos una transacción porque se debe guardar en ambas tablas
            DB::beginTransaction();
            
            //Guardar el producto
            $producto = Producto::create($data);
            //comprobamos si vienen imagenes en el request para guardarlas
            if($request->hasFile('imagenes')){
                //recorremos la coleccion de imagenes para gestionarlas
                foreach($request->file('imagenes') as $file){
                    //cambiamos el nombre de la imagen
                    $nombreImagen = time().'_'.$file->getClientOriginalName();
                    $rutaDestino = public_path('images/productos');
                    //si no existe la carpeta la creamos
                    if(!file_exists($rutaDestino)){
                        mkdir($rutaDestino, 0755, true);
                    }
                    //copiamos el archivo a la ruta destino
                    $file->move($rutaDestino, $nombreImagen);
                    //guardamos el nombre de la imagen en la tabla
                    //imagenes a través del modelo Imagen
                    Imagen::create([
                        'nombre' => $nombreImagen,
                        'producto_id' => $producto->id
                    ]);
                }
            }
            //confirmamos la transacción
            DB::commit();
            //obtenemos el objeto guardado completo
            $producto->load(['marca','categoria','imagenes']);
            return response()->json([
                'message' => 'Producto registrado correctamente',
                'producto' => $producto
            ],201);
        }catch(ValidationException $e){
            DB::rollBack();
            return response()->json([
                'message' => 'Error de validación',
                'errors' => $e->errors()
            ],422);
        }catch(\Exception $e){
             DB::rollBack();
            return response()->json([
                'message' => 'Error interno del servidor'
            ],500);
        }
    }

```
### 6.3 Programar el método **show()** para obtener un producto completo por medio de su id

```php
  public function show(string $id)
    {
        try{
            $producto = Producto::with(['marca','categoria','imagenes'])->findOrFail($id);
            return response()->json($producto);
        }catch(ModelNotFoundException $e){
            return response()->json([
                'message' => 'No se ha encontrado el producto con ID = ' . $id
            ],404);
        }
    }

```
### 6.4 Programar el método **update()** para actualizar los datos e imágenes de un producto
```php
public function update(Request $request, string $id)
    {
        try{
            //obtenemos el producto de bd
            $producto = Producto::with('imagenes')->findOrFail($id);
            //veriflicanmos se sea un objeto el que se recibe
            if(!$request->has('producto')){
                return response()->json([
                    'message' => 'El objeto producto es requerido'
                ],422);
            }
            //decodificamos el objeto producto
            $productoData = json_decode($request->producto,true);
            //verificamos que tenga un formato válido
            if(json_last_error() !== JSON_ERROR_NONE){
                return response()->json([
                    'message' => 'El JSON enviado en producto no es válido',
                    'error' => json_last_error_msg()
                ],422);
            }
            //validamos estructura 
            $validator = Validator::make($productoData, [
                'nombre' => 'required|string|max:80|unique:productos,nombre,' .$id,
                'descripcion' => 'required|string|max:200',
                'precio' => 'required|numeric|min:0',
                'stock' => 'required|integer|min:0',
                'modelo' => 'required|string|max:50',
                'marca.id' => 'required|exists:marcas,id',
                'categoria.id' => 'required|exists:categorias,id',
                'activo' => 'required|boolean',
            ]);
            if($validator->fails()){
                return response()->json([
                    'message' => 'Existen errores de validación',
                    'error' => $validator->errors()
                ],500);
            }
            //iniciamos la transación
            DB::beginTransaction();
            //transformamos datos al formato de base de datos
             $data = [
                'nombre' => $productoData['nombre'],
                'descripcion' => $productoData['descripcion'],
                'precio' => $productoData['precio'],
                'stock' => $productoData['stock'],
                'modelo' => $productoData['modelo'],
                'marca_id' => $productoData['marca']['id'],
                'categoria_id' => $productoData['categoria']['id'],
                'activo' => $productoData['activo'],
            ];
            //guardamos en producto
            $producto->update($data);
            //gestionamos las imagenes
           if($request->hasFile('imagenes')){
                //eliminamos fisicamente las imagenes anteriores
                foreach($producto->imagenes as $img){
                    $ruta = public_path('images/productos/'. $img->nombre);
                    if(file_exists($ruta)){
                        unlink($ruta); //borramos el archivo físico
                    }
                    //borramos el registro de imagenes
                    $img->delete();
                } 
                //guardamos las nuevas imágenes
                 foreach($request->file('imagenes') as $file){
                    //cambiamos el nombre de la imagen
                    $nombreImagen = time().'_'.$file->getClientOriginalName();
                    $rutaDestino = public_path('images/productos');
                    //si no existe la carpeta la creamos
                    if(!file_exists($rutaDestino)){
                        mkdir($rutaDestino, 0755, true);
                    }
                    //copiamos el archivo a la ruta destino
                    $file->move($rutaDestino, $nombreImagen);
                    //guardamos el nombre de la imagen en la tabla
                    //imagenes a través del modelo Imagen
                    Imagen::create([
                        'nombre' => $nombreImagen,
                        'producto_id' => $producto->id
                    ]);
                }     
           }
           //confirmamos la transacción
           DB::commit();
           return response()->json([
                'message' => 'Producto actualizado correctamente',
                'producto' => $producto->load(['marca','categoria','imagenes'])
           ],202);
        }catch(ModelNotFoundException $e){
            return response()->json([
                'message' => 'No se encuentra el producto con ID = ' . $id
            ],404);
        }catch(\Exception $e){
            DB::rollBack();
            return response()->json([
                'message' => 'Error al actualizar el producto',
                'error' => $e->getMessage()
            ],500);
        }
    }
```
### 6.5 Programar el método **destroy** para eliminar un registro de la tabla productos
Este método deberá desarrollarlo, considerar lo siguiente:
* Considerar que debe existir el modelo con el $id pasado como parámetro, en caso contrario manejarlo con ModelNotFoundException
* Verificar que el producto a eliminar no tenga registros relacionados con order_items
* En caso de poderse eliminar, se deben eliminar los archicos de imagen de la carpeta **public/images/productos**, asociados con el producto, de igual manera eliminar los registros en la tabla **imagenes** asociadas al producto
---
