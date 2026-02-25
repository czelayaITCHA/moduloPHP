# Registro y autenticación con JWT en Laravel 12

## 1. Instalación del paquete mas usado para este propósito
```bash
composer require tymon/jwt-auth
```
## 2. Publicar configuración
```bash
php artisan vendor:publish --provider="Tymon\JWTAuth\Providers\LaravelServiceProvider"
```
## 3. Generar clave secreta
```bash
php artisan jwt:secret
```
Esto generará una llave secreta que se mostrará en la consola y creará una variable de entorno en el archivo **.env**, como se muestra a continuación:

<img width="763" height="460" alt="image" src="https://github.com/user-attachments/assets/38668577-e375-4a99-b906-41e72b2d41f6" />

## 4. Configurar el archivo config/auth.php para usar JWT
* configuración de la sección **defaults**
  ```php
    'defaults' => [
        'guard' => 'api',
        'passwords' => 'users',
    ],
  ```
* configuración de la sección **guards**
  ```php
    'guards' => [
        'api' => [
            'driver' => 'jwt',
            'provider' => 'users',
        ],
    ],

  ```




