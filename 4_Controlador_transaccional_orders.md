# Controlador transaccional de Gestión de Ordenes
Ahora vamos a crear el controlador, que permitir gestionar las ordenes en la parte pública y administrativa
## 1. Crear controlador con apiResource
```bash
php artisan make:controller OrderController --api
```
## 2. Crear ruta en archivo api.php
Si aún no ha creado la ruta para acceder a las funcionalidades del controlador, se debe crear la las rutas en api.php, con la linea siguiente:
```bash
Route::apiResource('ordenes',OrderController::class);
```
recuerde importar el espacio de nombres del controlador en la parte superior del archivo 
```bash
use App\Http\Controllers\OrderController;
```
Al consultar la tabla de rutas debe aparcer como la siguiente imágen
<img width="1086" height="648" alt="image" src="https://github.com/user-attachments/assets/0de89346-c7c0-4f30-91e6-f729a2c2c8ca" />


## 3. Programar métodos o funciones del controlador
### 3.1 Programar lógica del método index()
Se implementa lógica para obtener todas las ordenes en general, filtradas por estado y por cliente
```php
public function index(Request $request)
    {   
        try{
            $query = Order::with(['user', 'items.producto', 'pagos']);
            //filtramos por estado de la orden
            if($request->estado){
                $query->where('estado', $request->estado);
            }
            //filtramos por cliente que realizó las ordenes
            if($request->user_id){
                $query->where('user_id', $request->user_id);
            }
            //definimos el orden en que se presentarán los datos
            $orders = $query->orderBy('fecha', 'desc')->get();
            return response()->json($orders);
        }catch(\Exception $e){
            return response()->json([
                'message' => 'Error al obtener el listado de órdenes',
                'error' => $e->getMessage()
            ],500);
        }
    }
```
Pruebas en Postman, a continuación se muestran ejemplos de pruebas de la funcionalidad del método anterior 
* Obtener todas las ordenes guardadas en la base de datos
  <img width="1012" height="760" alt="image" src="https://github.com/user-attachments/assets/6cf1e22a-a418-492a-aa8d-455e50783dde" />

* Obtener las ordes de acuerdo a un estado, devolverá una colección de ordenes filtradas por el valor del parámetro **estado**
  <img width="1019" height="763" alt="image" src="https://github.com/user-attachments/assets/ebe469f6-8807-4003-a268-1d8af09b7962" />

 ### 3.2 Programar el método show($id) para obtener una orden por medio de su id
 ```php
 public function show(string $id)
    {
        try{
            $order = Order::with(['user','items.producto','pagos'])->findOrFail($id);
            return response()->json($order);
        }catch(ModelNotFoundException $e){
            return response()->json([
                'message' => 'No se ha encontrado la orden con ID = ' . $id
            ],404);
        }
    }
```
 ### 3.3 Programar método store para registrar o crear una orden

 ```php
public function store(Request $request)
    {
        try{
            //validamos a nivel $request
            $data = $request->validate([
                'user_id' => 'required|exists:users,id',
                'fecha' =>'required|date',
                'subtotal' =>'required|numeric|min:0',
                'impuesto' =>'required|numeric|min:0',
                'total' =>'required|numeric|min:0',
                'items' => 'required|array|min:1',
                'items.*.producto_id' =>'required|exists:productos,id',
                'items.*.cantidad'  => 'required|numeric|min:1'
            ]);
            //iniciamos la transacción
            DB::beginTransaction();
            //creamos el registro en orders
            $order = Order::create([
                'correlativo' => $this->generarCorrelativo(),
                'fecha' => $data['fecha'],
                'subtotal' => $data['subtotal'],
                'impuesto' => $data['impuesto'],
                'total' => $data['total'],
                'estado' => 'PENDIENTE',
                'user_id' => $data['user_id']
            ]);
            //recorremos la colección de items para agregar en order_items
            foreach($data['items'] as $item){
                //obtenemos el producto de la tabla
                $producto = Producto::findOrFail($item['producto_id']);
                $subt = $producto->precio * $item['cantidad'];
                //creamos cada OrderItem
                OrderItem::create([
                    'cantidad' => $item['cantidad'],
                    'precio_unitario' => $producto->precio,
                    'subtotal' => $subt,
                    'producto_id' => $producto->id,
                    'order_id' => $order->id
                ]);
            }
            DB::commit();
            return response()->json([
                'message' => 'Orden creada correctamente',
                'order' => $order->load('items.producto')
            ],201);
        }catch(\Exception $e){
            DB::rollBack();
            return response()->json([
                'message' => 'Error al crear la orden',
                'error' => $e->getMessage()
            ],500);
        }
    }
```
* Crear método privado para generar correlativo de la orden
```php
private function generarCorrelativo(){
        $year = now()->format('Y');
        $month = now()->format('m');
        $ultimo = Order::whereYear('fecha', $year)
        ->whereMonth('fecha', $month)
        ->lockForUpdate()
        ->count();
        $numero = str_pad($ultimo + 1 ,4,'0', STR_PAD_LEFT);
        return $year . $month . $numero;
    }
```
### 3.4 Crear un método para gestionar los estados de las órdenes, aparte de los métodos que proporciona un controlador de tipo apiResource

* agregar una migración para actualizar el estado de la tabla orders para que el *enum* tenga también el valor de 'ENTREGADA'
 ```bash
php artisan make:migration update_estado_enum_in_table_orders
```
* Editar el archivo de migración para cambiar los valores de **enum** del campo **estado**, como se va ejecutar una consulta nativa, se requiere importar la clase DB, en la parte superior de la migración
  ```php
  use Illuminate\Support\Facades\DB;
  ```
  Agregamos la consulta para modificar la columna estado, en la función up() de la migración y down() debe quedar vacía, tal como se muestra a continuación:
  ```php
  public function up(): void
    {
        DB::statement("ALTER TABLE orders MODIFY estado ENUM(
        'PENDIENTE','PAGADA','CANCELADA', 'REEMBOLSADA',
        'ENTREGADA') DEFAULT 'PENDIENTE'");
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        
    }
  ```
* 
  
