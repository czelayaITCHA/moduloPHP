# 13. Programar funcionalidades para mostrar datos en el componente Dashboard.vue del panel administrativo
Un Dashboard (o panel de control) es una interfaz visual que muestra indicadores clave de información (KPIs) y métricas relevantes de un sistema o negocio en tiempo real o casi en tiempo real, con el objetivo de facilitar la toma de decisiones rápidas.

## 13.1 Crear y programar métodos del controlador para obtener datos a mostrar en el Dashboard

```bash
php artisan make:controller DashboardController
```
Programar los siguientes métodos

```php
//función para obtener totales
    public function getResumen(){
        //obtener las ventas totales
        $ventasTotales = Order::where('estado',['PAGADA','ENTREGADA'])->sum('total');
        //obtener las ventas de la fecha
        $ventasHoy = Order::where('estado','PAGADA')
        ->whereDate('fecha', Carbon::today())
        ->sum('total');
        //cantidad de ordenes pagadas
        $ordenesPagadas = Order::where('estado','PAGADA')->count();
        $ordenesPendientes = Order::where('estado','PENDIENTE')->count();
        $ordenesCanceladas = Order::where('estado','CANCELADA')->count();
        $ordenesReembolsadas = Order::where('estado','REEMBOLSADA')->count();
        return response()->json([
            'ventas_totales' => $ventasTotales,
            'ventas_hoy' => $ventasHoy,
            'ordenes_pagadas' =>$ordenesPagadas,
            'ordenes_pendientes' => $ordenesPendientes,
            'ordenes_canceladas' =>$ordenesCanceladas,
            'ordenes_reembolsadas' => $ordenesReembolsadas
        ]);
    }

    //función para obtener las ventas por mes
    public function ventasPorMes(){
        $anioActual = now()->year;
        $ventas = Order::select(
            DB::raw("MONTH(fecha) as numero_mes"),
            DB::raw("MONTHNAME(fecha) as mes"),
            DB::raw("SUM(total) as total")
        )
        ->whereIn('estado',['PAGADA','ENTREGADA'])
        ->whereYear('fecha', $anioActual)
        ->groupBy(DB::raw("MONTH(fecha)"), DB::raw("MONTHNAME(fecha)"))
        ->orderBy("mes")
        ->get();
        return response()->json($ventas);
    }

    //funcion para obtener las ventas por año
    public function ventasPorAnio()
    {

        $ventas = Order::select(
                DB::raw("YEAR(fecha) as anio"),
                DB::raw("SUM(total) as total")
            )
            ->whereIn('estado',['PAGADA', 'ENTREGADA'])
            ->groupBy(DB::raw("YEAR(fecha)"))
            ->orderBy("anio")
            ->get();

        return response()->json($ventas);
    }

    //Top 5 de productos mas vendidos
    public function topProductos()
    {

        $productos = DB::table('order_items')
            ->join('productos','order_items.producto_id','=','productos.id')
            ->select(
                'productos.nombre',
                DB::raw("SUM(order_items.cantidad) as vendidos")
            )
            ->groupBy('productos.nombre')
            ->orderByDesc('vendidos')
            ->limit(5)
            ->get();

        return response()->json($productos);
    }
    
    //funcion para obtener las últimas 10 ordenes
    public function ultimasOrdenes()
    {
        $ordenes = Order::with('user')
            ->orderBy('created_at','desc')
            ->limit(10)
            ->get(['id','correlativo','fecha','total','estado','user_id']);

        return response()->json($ordenes);

    }
```
**Nota:** Recuerde hacer las siguientes importaciones en el controlador DashboardController
```php
use Illuminate\Support\Facades\DB;
use Carbon\Carbon;
use App\Models\Order;
````
## 13.2 Crear las rutas en el archivo api.php
```php
Route::prefix('admin')->group(function(){
    Route::get('/dashboard/resumen', [DashboardController::class, 'getResumen']);
    Route::get('/dashboard/ventas-mes', [DashboardController::class, 'ventasPorMes']);
    Route::get('/dashboard/ventas-anio',[DashboardController::class,'ventasPorAnio']);
    Route::get('/dashboard/top-productos',[DashboardController::class,'topProductos']);
    Route::get('/dashboard/ultimas-ordenes',[DashboardController::class,'ultimasOrdenes']);
});
```
