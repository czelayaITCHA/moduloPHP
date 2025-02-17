# Creación y Configuración inicial de proyecto laravel 11

## Introducción
Laravel 11 es la última versión del popular framework de desarrollo web en PHP, diseñado para facilitar la creación de aplicaciones web robustas y eficientes. Con una sintaxis elegante y herramientas integradas, Laravel simplifica tareas comunes como la autenticación, el enrutamiento y la gestión de bases de datos, permitiendo a los desarrolladores centrarse en la funcionalidad específica de sus aplicaciones.
## Requisitos para crear proyecto en laravel 11
### 1.- Versión de PHP >= 8.2
![image](https://github.com/user-attachments/assets/0885b363-dc36-4e7e-adbb-f66fe62fd80b)
### 2.- composer
![image](https://github.com/user-attachments/assets/0db204f0-a92a-4a78-8f76-541c22327a97)
### 1.- nodeJS
![image](https://github.com/user-attachments/assets/56c29391-7e65-499b-bbb7-1932f07a9b64)

## Crear nuevo proyecto laravel
Existen dos formas de crear un proyecto en laravel, usando composer que para la práctica usaremos esta opción y la otra es usando el instalador de Laravel para opciones mas personalizadas del proyecto 
### 1.- Crear proyecto con composer
Creamos el proyecto con el gestor de dependencias de Laravel
```bash
composer create-project laravel/laravel orders-app
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
Cuando se crea el proyecto, Laravel 11 ejecuta las migraciones que trae por defecto para sqlite, como se muestra en la siguiente imágen
![image](https://github.com/user-attachments/assets/c1b23301-029d-4c84-b95c-d68d375bc17e)
### abrir proyecto con Visual Studio Code
![image](https://github.com/user-attachments/assets/101ebbb0-2618-4f25-a8fc-9d1db075ea01)
A continuación se muestra la estructura del proyecto
![image](https://github.com/user-attachments/assets/7e9145ec-94ba-4b73-a6ac-966d7955a6c4)

### iniciar proyecto con el servidor de desarrollo
#### 1.- iniciar servidor de desarrollo
![image](https://github.com/user-attachments/assets/a39e1e26-72ea-4721-a876-39a024952206)

#### 2.- Acceder a la aplicación desde el navegador
![image](https://github.com/user-attachments/assets/e170181f-75b7-4aa4-bfaa-f32261234dc3)

## Configuración de conexión
Editar el archivo .env y establecer parámetros de conexión para la base de datos en MySQL

![image](https://github.com/user-attachments/assets/374331a9-ce6e-4393-95ce-8cf0df6c00c4)

## Especificar el motor de almacenamiento que se utilizará para las tablas de la base de datos MySQL.
Editar el archivo config/database.php

![image](https://github.com/user-attachments/assets/a055ef7d-68be-4531-afd7-5d231aab5bef)
## Ejecutar la migraciones
```bash
php artisan migrate
```
Puede ver el estado de las migraciones con el siguiente comando
```bash
php artisan migrate:status
```
Revisar tablas en la base de datos, con phpMyadmin

## Crear migraciones para la base de datos ordersDB, que se utilizarán en el proyecto de clase
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
### 4.- Generar archivo de migracion para tabla de imagenes
```bash
php artisan make:migration create_imagenes_table
```
### 5.- Generar archivo de migracion para tabla de ordenes
```bash
php artisan make:migration create_ordenes_table
```
### 6.- Generar archivo de migracion para tabla de detalle_ordenes
```bash
php artisan make:migration create_detalle_ordenes_table
```
