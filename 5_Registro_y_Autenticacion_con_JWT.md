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
## 5. Configurar el modelo User para implementar JWT
importamos el espacio de nombre para la interface **JWTSubject**, antes de la definición de la clase del modelo User
```php
use Tymon\JWTAuth\Contracts\JWTSubject;
```
Implementación de la interface **JWTSubject** al modelo user y sus métodos
```php
class User extends Authenticatable implements JWTSubject
{
    /** @use HasFactory<\Database\Factories\UserFactory> */
    use HasFactory, Notifiable;

    //sobreescribimos la variable $guard_name
    protected $guard_name = 'api';

    //implementación de los métodos de JWT
    public function getJWTIdentifier(){
        return $this->getKey();
    }
    public function getJWTCustomClaims(){
        return [];
    }

    /**
     * The attributes that are mass assignable.
     *
     * @var list<string>
     */
    protected $fillable = [
        'name',
        'email',
        'password',
    ];

    /**
     * The attributes that should be hidden for serialization.
     *
     * @var list<string>
     */
    protected $hidden = [
        'password',
        'remember_token',
    ];

    /**
     * Get the attributes that should be cast.
     *
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            'email_verified_at' => 'datetime',
            'password' => 'hashed',
        ];
    }
}
```
## 6. Crear el controlador AuthController
* Crear el archivo del controlador dentro de una subcarpeta Auth
  ```bash
  php artisan make:controller Auth\AuthController
  ```
* Programar los métodos para autenticación y registro de usuarios
  
* 
## 7. Crear rutas en el archivo api.php
```php
Route::prefix('auth')->group(function(){
    Route::post('register', [AuthController::class, 'register']);
    Route::post('login', [AuthController::class, 'login']);

    Route::middleware('auth:api')->group(function(){
        Route::get('me',[AuthController::class, 'me']);
        Route::post('logout',[AuthController::class, 'logout']);
        Route::post('refresh',[AuthController::class, 'refresh']);
    });
});
```
Recuerde que debe importar el controlador AuthController

```php
use App\Http\Controllers\Auth\AuthController;
```


