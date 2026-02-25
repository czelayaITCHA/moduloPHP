# Gestión de Roles y Permisos con Laravel Permission 
## 1. Introducción
Laravel-Permission (desarrollado por Spatie) es el paquete estándar de la industria para gestionar roles y permisos de usuarios en aplicaciones Laravel. Permite asignar permisos específicos (ej. "editar artículos") a roles (ej. "editor") o directamente a usuarios, facilitando la autorización, protección de rutas mediante middleware y control de visibilidad en vistas Blade.

* **Spatie Laravel Permission**

Permite agregar:

- Roles (ADMIN, CLIENTE, VENDEDOR, etc.)

- Permisos (registrar producto, registrar reserva, gestionar estado, etc.)

- Relación many-to-many entre usuarios y roles

- Middleware automático role y permission
  

* **Cuando instalas el paquete, se crean las siguientes tablas:**

roles

permissions

model_has_roles

model_has_permissions

role_has_permissions
## 2. Instalar el paquete

```bash
composer require spatie/laravel-permission
```
## 3. Publicar migraciones
```bash
php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider"
```
## 4. Ejecutar migraciones de Laravel Permission
```bash
php artisan migrate
```
Verificar que en su base datos, se hayan creado las tablas marcadas en la siguiente imágen:

<img width="930" height="437" alt="image" src="https://github.com/user-attachments/assets/68037c4f-6159-40e6-9cc1-4a99b7be3b8e" />

## 5. Configurar el modelo User
* importación necesaria:
```bash
use Spatie\Permission\Traits\HasRoles;
```
* agregar **hasRoles**, la clase completa quedaría así:
  
```php
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Tymon\JWTAuth\Contracts\JWTSubject;
use Spatie\Permission\Traits\HasRoles;

class User extends Authenticatable implements JWTSubject
{
    /** @use HasFactory<\Database\Factories\UserFactory> */
    use HasFactory, Notifiable, HasRoles;

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
## 6. Crear Seeder para agregar Roles
Un Seeder es una clase de PHP diseñada para automatizar la carga de datos iniciales en las tablas de tu base de datos. Es la herramienta ideal para evitar insertar registros manualmente cada vez que reinicias tu entorno de desarrollo o pruebas.

### 6.1 Crear seeder RoleSeeder
```bash
php artisan make:seeder RoleSeeder
```
El comando anterior crea el seeder en la carpeta database\seeders, como se muestra en la imágen

<img width="1314" height="554" alt="image" src="https://github.com/user-attachments/assets/e703f62e-b8c6-4271-b0f4-cd249eddc23a" />

### 6.2 Importación necesaria
```php
use Spatie\Permission\Models\Role;
```
### 6.3 Crear los roles en el método **run**
```php
Role::create(['name' => 'ADMIN', 'guard_name' => 'api']);
Role::create(['name' => 'CLIENTE', 'guard_name' => 'api']);
Role::create(['name' => 'VENDEDOR', 'guard_name' => 'api']);
```
## 7.
