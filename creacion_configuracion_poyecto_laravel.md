# Creación y Configuración inicial de proyecto laravel 12

## Introducción
Laravel 12 es la última versión del popular framework de desarrollo web en PHP, diseñado para facilitar la creación de aplicaciones web robustas y eficientes. Con una sintaxis elegante y herramientas integradas, Laravel simplifica tareas comunes como la autenticación, el enrutamiento y la gestión de bases de datos, permitiendo a los desarrolladores centrarse en la funcionalidad específica de sus aplicaciones.
## Requisitos para crear proyecto en laravel 12
### 1.- Versión de PHP >= 8.2

<img width="673" height="141" alt="image" src="https://github.com/user-attachments/assets/0142d494-4a84-4a02-b699-b8b61fafb331" />

### 2.- instalar composer 

- Descarga composer si no lo tienes instalado de https://getcomposer.org/ sigue los pasos de instalación y luego verifica la versión instalada (es posible que se necesite reiniciar el sistema)
  
<img width="611" height="103" alt="image" src="https://github.com/user-attachments/assets/93648cc7-ac07-438a-ae06-a831f31b3622" />

### 3.- nodeJS
<img width="242" height="63" alt="image" src="https://github.com/user-attachments/assets/617fe260-ee6f-4341-8093-3466eafbcc63" />

## Crear nuevo proyecto laravel
Existen dos formas de crear un proyecto en laravel, usando composer que para la práctica usaremos esta opción y la otra es usando el instalador de Laravel para opciones mas personalizadas del proyecto 
### 1.- Crear proyecto con composer
Creamos el proyecto con el gestor de dependencias de Laravel
```bash
composer create-project laravel/laravel:^12.0 shop-app
```

### 2.- Usando el instalador de Laravel
Este método es ideal si deseas una instalación rápida y personalizada.
Instalación del instalador de Laravel:

Primero, asegúrate de tener Composer instalado en tu sistema. Luego, ejecuta:
```
composer global require laravel/installer
```
Este comando instala el instalador de Laravel de forma global.

Creación de un nuevo proyecto:

Una vez instalado, puedes crear un nuevo proyecto con:
```bash
laravel new nombre-del-proyecto
```
Con este comando se crea una nueva aplicación con el instalador global de laravel
## Estructura e inicio del proyecto
Cuando se crea el proyecto, Laravel 12 ejecuta las migraciones que trae por defecto para sqlite, como se muestra en la siguiente imágen:
<img width="1079" height="690" alt="image" src="https://github.com/user-attachments/assets/67589e94-2058-49ea-a25f-b370cf7f4a78" />
### abrir proyecto con Visual Studio Code
<img width="282" height="69" alt="image" src="https://github.com/user-attachments/assets/54f59923-ab2f-4526-9b80-dc7a17d0866d" />

A continuación se muestra la estructura del proyecto

<img width="1919" height="1032" alt="image" src="https://github.com/user-attachments/assets/5618c904-fb9b-41f0-be05-d82257fa3cda" />


### iniciar proyecto con el servidor de desarrollo
#### 1.- iniciar servidor de desarrollo
<img width="1267" height="156" alt="image" src="https://github.com/user-attachments/assets/58d2a0a3-a9cd-41f1-bdec-156a33049ad0" />


#### 2.- Acceder a la aplicación desde el navegador
<img width="1909" height="768" alt="image" src="https://github.com/user-attachments/assets/a7d2102b-1572-47dc-93bd-1f7da6601f3f" />

## Configuración de conexión
Editar el archivo .env y establecer parámetros de conexión para la base de datos en MySQL

```bash
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=shop_db
DB_USERNAME=root
DB_PASSWORD=
```

## Especificar el motor de almacenamiento que se utilizará para las tablas de la base de datos MySQL.
Editar el archivo config/database.php

```bash
'mysql' => [
            'driver' => 'mysql',
            'url' => env('DB_URL'),
            'host' => env('DB_HOST', '127.0.0.1'),
            'port' => env('DB_PORT', '3306'),
            'database' => env('DB_DATABASE', 'laravel'),
            'username' => env('DB_USERNAME', 'root'),
            'password' => env('DB_PASSWORD', ''),
            'unix_socket' => env('DB_SOCKET', ''),
            'charset' => env('DB_CHARSET', 'utf8mb4'),
            'collation' => env('DB_COLLATION', 'utf8mb4_unicode_ci'),
            'prefix' => '',
            'prefix_indexes' => true,
            'strict' => true,
            'engine' => 'InnoDB',
            'engine' => null,
            'options' => extension_loaded('pdo_mysql') ? array_filter([
                PDO::MYSQL_ATTR_SSL_CA => env('MYSQL_ATTR_SSL_CA'),
            ]) : [],
        ],
```
## Ejecutar la migraciones
```bash
php artisan migrate
```
Puede ver el estado de las migraciones con el siguiente comando
```bash
php artisan migrate:status
```
Revisar tablas en la base de datos, con phpMyadmin

## Crear migraciones para la base de datos shop_db, que se utilizarán en el proyecto de clase
### 1.- Generar archivo de migracion para tabla de marcas
```bash
php artisan make:migration create_marcas_table
```
### 2.- Generar archivo de migracion para tabla de categorias
```bash
php artisan make:migration create_categorias_table
```
### 3.- Generar archivo de migracion para tabla de productos
```bash
php artisan make:migration create_productos_table
```
### 4.- Generar archivo de migracion para tabla de orders
```bash
php artisan make:migration create_orders_table
```
### 5.- Generar archivo de migracion para tabla de order_items
```bash
php artisan make:migration create_order_items_table
```
### 6.- Generar archivo de migracion para tabla de pagos
```bash
php artisan make:migration create_pagos_table
```
## Definir estructura de cada migración (método up)

### 1.- Estructura de migracion para marcas
```php
  public function up(): void
    {
        Schema::create('marcas', function (Blueprint $table) {
            $table->id();
            $table->string('nombre',80)->nullable(false)->unique();
            $table->timestamps();
        });
    }
```
### 2.- Estructura de migracion para categorias
```php
 public function up(): void
    {
        Schema::create('categorias', function (Blueprint $table) {
            $table->id();
            $table->string('nombre',80)->nullable(false)->unique();
            $table->timestamps();
        });
    }
```
### 3.- Estructura de migracion para productos
```php
public function up(): void
    {
        Schema::create('productos', function (Blueprint $table) {
            $table->id();
            $table->string('nombre',80)->nullable(false);
            $table->string('descripcion',200);
            $table->decimal('precio',10,2);
            $table->decimal('stock',10,2);
            $table->string('modelo',50)->nullable(true);
            $table->boolean('activo')->default(true);
            //creando las llaves foráneas
            $table->unsignedBigInteger('marca_id');
            $table->foreign('marca_id')->references('id')->on('marcas');
            $table->unsignedBigInteger('categoria_id');
            $table->foreign('categoria_id')->references('id')->on('categorias');
            $table->timestamps();
        });
    }
```
### 4.- Estructura de migracion para imagenes
```php
 public function up(): void
    {
        Schema::create('imagenes', function (Blueprint $table) {
            $table->id();
            $table->string('nombre',150);
            $table->unsignedBigInteger('producto_id');
            $table->foreign('producto_id')->references('id')->on('productos');
            $table->timestamps();
        });
    }
```
### 5.- Estructura de migracion para orders
```php
public function up(): void
    {
        Schema::create('orders', function (Blueprint $table) {
            $table->id();
            $table->string('correlativo',10)->unique();
            $table->date('fecha');
            $table->date('fecha_despacho')->nullable(true);
            $table->string('estado',1)->default('R');
            $table->decimal('subtotal', 10, 2);
            $table->decimal('impuesto', 10, 2)->default(0);
            $table->decimal('total', 10, 2);
            $table->enum('estado', ['PENDIENTE','PAGADA','CANCELADA','REEMBOLSADA'])->default('PENDIENTE');
            $table->unsignedBigInteger('user_id');
            $table->foreign('user_id')->references('id')->on('users');
            $table->timestamps();
        });
    }
```
### 6.- Estructura de migracion para order_items
```php
 public function up(): void
    {
        Schema::create('order_items', function (Blueprint $table) {
            $table->id();
            $table->integer('cantidad');
            $table->decimal('precio_unitario', 10, 2);
            $table->decimal('subtotal', 10, 2);
            $table->unsignedBigInteger('producto_id');
            $table->foreign('producto_id')->references('id')->on('productos');
            $table->unsignedBigInteger('orden_id');
            $table->foreign('orden_id')->references('id')->on('ordenes');
            $table->timestamps();
        });
    }
```
### 7.- Estructura de migracion para pagos
```php
 public function up(): void
    {
        Schema::create('order_items', function (Blueprint $table) {
            $table->id();
            $table->enum('metodo', ['STRIPE','PAYPAL','TRANSFERENCIA']);
            $table->string('referencia', 50);
            $table->decimal('monto', 10, 2);
            $table->enum('estado', ['PENDIENTE', 'APROBADO','RECHAZADO'])->default('PENDIENTE');
            $table->json('respuesta_pasarela')->nullable(); 
            $table->unsignedBigInteger('pago_id');
            $table->foreign('pago_id')->references('id')->on('pagos');
            $table->timestamps();
        });
    }
```

Cambiar el nombre de la tabla en el método down
```php
 public function down(): void
    {
        Schema::dropIfExists('order_items');
    }
```
## Ejecutar migraciones pendientes
```bash
php artisan migrate
```
Verificar estado de las migraciones
```bash
php artisan migrate:status
```
En caso de no existir errores verificar que se hayan creado las tablas en la base de datos ordersDB

## ¿Qué es artisan?
Artisan es la interfaz de línea de comandos (CLI) que viene incluida con Laravel. Es una herramienta muy útil que te permite realizar diversas tareas comunes en el desarrollo de aplicaciones Laravel de manera rápida y sencilla.
Para usar Artisan, simplemente abre la terminal en la raíz de tu proyecto Laravel y ejecuta el comando php artisan. Esto te mostrará una lista de todos los comandos disponibles. Luego, puedes ejecutar un comando específico escribiendo php artisan nombre_del_comando.

Algunos ejemplos de comandos Artisan comunes son:

* php artisan make:model NombreDelModelo: Crea un nuevo modelo.
* php artisan migrate: Ejecuta las migraciones pendientes.
* php artisan serve: Inicia el servidor de desarrollo.
* php artisan tinker: Abre una consola interactiva para probar código Laravel.
  
En resumen, Artisan es una herramienta esencial para cualquier desarrollador de Laravel. Te ayuda a agilizar el desarrollo, automatizar tareas y mantener tu código organizado y consistente
