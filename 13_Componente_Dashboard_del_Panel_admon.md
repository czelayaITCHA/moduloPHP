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
Es importante verificar la tabla de rutas de los endpoints que se crean, así como hacer las respectivas pruebas en Postman para garantizar que esten bien programadas las funciones en el backend
```bash
php artisan r:l
```
<img width="1313" height="789" alt="image" src="https://github.com/user-attachments/assets/c18f1445-af5d-4a97-ac44-14765b2be92a" />
En la tabla de rutas anterior, se puede apreciar las rutas que se generan para consumir los endpoints para mostrar los datos en el Dashboard. A continuación se muestran resultados de algunas pruebas en Postman
<img width="1084" height="793" alt="image" src="https://github.com/user-attachments/assets/5caf3ad8-2f46-4eb5-99fb-612ab0adc8f1" />

<img width="1086" height="800" alt="image" src="https://github.com/user-attachments/assets/1c723d6f-4f5f-4593-90e3-7fb1e7feb342" />

<img width="1086" height="795" alt="image" src="https://github.com/user-attachments/assets/9af5563d-5829-4ba2-af6d-ae512c0a4d5d" />

## 13.3 instalar libreria para generar gráficos
Chart.js es una biblioteca JavaScript de código abierto utilizada para crear gráficos interactivos y responsivos en aplicaciones web. Está diseñada para ser ligera, fácil de usar y altamente configurable, permitiendo representar datos de forma visual mediante gráficos dinámicos.

Fue desarrollada para funcionar sobre el elemento <canvas> de HTML5, lo que permite dibujar gráficos directamente en el navegador sin depender de plugins externos.

* Instalación de la librería
```bash
npm install chart.js
```

## 13.4 Crear el service dashboardService.js en el proyecto frontend, con las funciones para consumir los endpoint

```js
import api from "./api";

export default {
  getResumen() {
    return api.get("/admin/dashboard/resumen");
  },

  ventasMes() {
    return api.get("/admin/dashboard/ventas-mes");
  },

  ventasAnio() {
    return api.get("/admin/dashboard/ventas-anio");
  },

  topProductos() {
    return api.get("/admin/dashboard/top-productos");
  },
  ultimasOrdenes() {
    return api.get("/admin/dashboard/ultimas-ordenes");
  },
  
};

```

## 13.5 Actualizar el código de Dashboard.vue, para que muestre registros de la base de datos

````vue
<template>
  <div class="space-y-8">
    <!-- Encabezado del Dashboard -->
    <div class="flex flex-col md:flex-row md:items-center md:justify-between gap-3">
      <h2 class="text-2xl font-bold text-gray-800">Dashboard Administrativo</h2>

      <span class="text-sm font-bold text-gray-500"> Año {{ anioActual }} </span>
    </div>

    <!-- Grid de tailwind para pintar las card con los datos de las ordenes por estado -->
    <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-5 gap-6">
      <div class="card">
        <div class="icon bg-blue-100 text-blue-600">
          <i class="pi pi-chart-line"></i>
        </div>

        <div>
          <p class="label">Ventas Totales</p>
          <h3 class="value">
            {{ formatoMoneda(resumen.ventas_totales) }}
          </h3>
        </div>
      </div>

      <div class="card">
        <div class="icon bg-yellow-100 text-yellow-600">
          <i class="pi pi-clock"></i>
        </div>

        <div>
          <p class="label">Órdenes Pendientes</p>
          <h3 class="value">{{ resumen.ordenes_pendientes }}</h3>
        </div>
      </div>

      <div class="card">
        <div class="icon bg-green-100 text-green-600">
          <i class="pi pi-check-circle"></i>
        </div>

        <div>
          <p class="label">Órdenes Pagadas</p>
          <h3 class="value">{{ resumen.ordenes_pagadas }}</h3>
        </div>
      </div>

      <div class="card">
        <div class="icon bg-red-100 text-red-600">
          <i class="pi pi-times-circle"></i>
        </div>

        <div>
          <p class="label">Órdenes Canceladas</p>
          <h3 class="value">{{ resumen.ordenes_canceladas }}</h3>
        </div>
      </div>

      <div class="card">
        <div class="icon bg-purple-100 text-purple-600">
          <i class="pi pi-refresh"></i>
        </div>

        <div>
          <p class="label">Órdenes Reembolsadas</p>
          <h3 class="value">{{ resumen.ordenes_reembolsadas }}</h3>
        </div>
      </div>
    </div>

    <!-- GRAFICOS -->
    <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
      <!-- Ventas Mensuales -->
      <div class="panel">
        <h3 class="panel-title">Ventas por Mes</h3>

        <div class="h-[320px]">
          <canvas ref="ventasChart"></canvas>
        </div>
      </div>
      <!-- Ventas por ano -->
      <div class="panel">
        <h3 class="panel-title">Ventas por Año</h3>

        <div class="h-[320px]">
          <canvas ref="ventasAnioChart"></canvas>
        </div>
      </div>

      <!-- Top productos -->
      <div class="panel">
        <h3 class="panel-title">Top Productos Vendidos</h3>

        <ul class="space-y-3">
          <li v-for="p in topProductos" :key="p.nombre" class="flex justify-between border-b pb-2">
            <span class="text-gray-700">
              {{ p.nombre }}
            </span>

            <span class="font-semibold">
              {{ p.vendidos }}
            </span>
          </li>
        </ul>
      </div>
    </div>

    <!-- Mostrando últimas 10 órdenes -->
    <div class="panel">
      <h3 class="panel-title mb-4">Últimas Órdenes</h3>

      <DataTable :value="ultimasOrdenes" responsiveLayout="scroll" class="p-datatable-sm">
        <Column field="correlativo" header="Orden" />

        <Column header="Cliente">
          <template #body="slotProps">
            {{ slotProps.data.user?.name }}
          </template>
        </Column>

        <Column field="total" header="Total">
          <template #body="slotProps">
            {{ formatoMoneda(slotProps.data.total) }}
          </template>
        </Column>

        <Column field="estado" header="Estado" />

        <Column field="fecha" header="Fecha" />
      </DataTable>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import Chart from "chart.js/auto";
import dashboardService from "@/services/dashboardService";

const anioActual = new Date().getFullYear();

const resumen = ref({});

const ventasChart = ref(null);
let chartInstance = null;

const ventasAnioChart = ref(null);
let chartVentasAnio = null;

const topProductos = ref([]);
const ultimasOrdenes = ref([]);

/* Función para dar formato de moneda */

const formatoMoneda = (valor) => {
  return new Intl.NumberFormat("es-SV", {
    style: "currency",
    currency: "USD",
  }).format(valor);
};

/* Función para obtener resumen de las ordenes  */
const loadResumen = async () => {
  const res = await dashboardService.getResumen();

  resumen.value = res.data;
};

/* Función para preperar el dataset (datos) para el gráfico de ventas por mes */
const loadVentasMes = async () => {
  const res = await dashboardService.ventasMes();

  const labels = res.data.map((v) => v.mes);
  const data = res.data.map((v) => v.total);

  if (chartInstance) {
    chartInstance.destroy();
  }

  chartInstance = new Chart(ventasChart.value, {
    type: "line",

    data: {
      labels,

      datasets: [
        {
          label: "Ventas",
          data,
          borderWidth: 3,
          tension: 0.4,
          fill: true,
        },
      ],
    },

    options: {
      responsive: true,
      maintainAspectRatio: false,
    },
  });
};

//funcion para obtener ventas por año
const loadVentasAnio = async () => {
  const res = await dashboardService.ventasAnio();

  const labels = res.data.map((v) => v.anio);
  const data = res.data.map((v) => v.total);

  if (chartVentasAnio) {
    chartVentasAnio.destroy();
  }

  chartVentasAnio = new Chart(ventasAnioChart.value, {
    type: "bar",

    data: {
      labels,
      datasets: [
        {
          label: "Ventas por Año",
          data,
          borderWidth: 1,
        },
      ],
    },

    options: {
      responsive: true,
      maintainAspectRatio: false,
    },
  });
};

const loadTopProductos = async () => {
  const res = await dashboardService.topProductos();

  topProductos.value = res.data;
};

const loadUltimasOrdenes = async () => {
  const res = await dashboardService.ultimasOrdenes();

  ultimasOrdenes.value = res.data;
};

onMounted(() => {
  loadResumen();
  loadVentasMes();
  loadVentasAnio();
  loadTopProductos();
  loadUltimasOrdenes();
});
</script>

<style scoped>
.card {
  @apply bg-white p-5 rounded-xl shadow-sm flex items-center gap-4 hover:shadow-md transition;
}

.icon {
  @apply flex items-center justify-center w-12 h-12 rounded-lg text-xl;
}

.label {
  @apply text-gray-500 text-sm;
}

.value {
  @apply font-bold text-xl text-gray-800;
}

.panel {
  @apply bg-white rounded-xl shadow-sm p-6;
}

.panel-title {
  @apply text-lg font-semibold text-gray-700 mb-3;
}
</style>

````
**Resultado Final: **, debe mostrarse un Dashboard con información proveniende de los registros dinámicos de la base de datos, similar a la siguiente imágen:
<img width="1914" height="951" alt="image" src="https://github.com/user-attachments/assets/9293d24a-0391-484a-9bfd-71bba0d44dce" />
