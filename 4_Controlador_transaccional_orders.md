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
## 3. 
