# Creación de Modelos y Relaciones en Laravel 12

## Introducción

En esta guía aprenderás a:

- Crear modelos en Laravel 12.
- Comprender las propiedades principales de una clase Modelo.
- Definir relaciones entre modelos (Eloquent ORM).
- Implementar las relaciones según las migraciones proporcionadas.

En el documento anterior se creó el proyecto Laravel, se realizaron algunas configuraciones y se crearon migraciones para las tablas en la base de datos 
**shop_db**, el siguiente paso es crear los modelos que representarán los datos para las tablas:

- Marcas
- Categorías
- Productos
- Imágenes
- Orders
- OrderItems
- Pagos
- Users (ya incluido en Laravel)

---

# 1️ Crear los Modelos

Desde la terminal ejecuta:

```bash
php artisan make:model Marca
php artisan make:model Categoria
php artisan make:model Producto
php artisan make:model Imagen
php artisan make:model Order
php artisan make:model OrderItem
php artisan make:model Pago
```

Si deseas crear modelo + migración + factory + controlador:

```bash
php artisan make:model Producto -mfsc
```

---

# 2️ Propiedades Importantes de un Modelo en Laravel

Todos los modelos extienden de:

```php
use Illuminate\Database\Eloquent\Model;
```

Ejemplo base:

```php
class Producto extends Model
{
    protected $table = 'productos';

    protected $fillable = [];

    protected $casts = [];

    protected $hidden = [];
}
```

## 🔹 Propiedades Principales

### 1. `$table`
Define el nombre de la tabla si no sigue la convención plural.

```php
protected $table = 'productos';
```

---

### 2. `$fillable`
Permite asignación masiva segura.

```php
protected $fillable = [
    'nombre',
    'descripcion',
    'precio',
    'stock',
    'modelo',
    'activo',
    'marca_id',
    'categoria_id'
];
```

---

### 3. `$guarded`
Alternativa a fillable.

```php
protected $guarded = ['id'];
```

---

### 4. `$casts`
Convierte tipos automáticamente.

```php
protected $casts = [
    'precio' => 'decimal:2',
    'stock' => 'decimal:2',
    'activo' => 'boolean'
];
```

---

### 5. `$hidden`
Oculta atributos en respuestas JSON.

```php
protected $hidden = ['respuesta_pasarela'];
```

---

# 3️ Tipos de Relaciones en Eloquent

## 🔹 1. One To Many (Uno a Muchos)

- Marca → muchos Productos
- Categoría → muchos Productos
- Producto → muchas Imágenes
- Order → muchos OrderItems
- Order → muchos Pagos
- User → muchas Orders

Métodos utilizados:

```php
hasMany()
belongsTo()
```

---

## 🔹 2. Many To One (Muchos a Uno)

Es la relación inversa de One to Many.

---

## 🔹 3. One To One

No aplica en este caso.

---

## 🔹 4. Many To Many

No aplica en este diseño actual.

---

# 4️ Implementación de Modelos con Relaciones

---

# Marca Model

```php
class Marca extends Model
{
    protected $fillable = ['nombre'];

    public function productos()
    {
        return $this->hasMany(Producto::class);
    }
}
```

---

# Categoria Model

```php
class Categoria extends Model
{
    protected $fillable = ['nombre'];

    public function productos()
    {
        return $this->hasMany(Producto::class);
    }
}
```

---

# Producto Model

```php
class Producto extends Model
{
    protected $fillable = [
        'nombre',
        'descripcion',
        'precio',
        'stock',
        'modelo',
        'activo',
        'marca_id',
        'categoria_id'
    ];

    protected $casts = [
        'precio' => 'decimal:2',
        'stock' => 'decimal:2',
        'activo' => 'boolean'
    ];

    public function marca()
    {
        return $this->belongsTo(Marca::class);
    }

    public function categoria()
    {
        return $this->belongsTo(Categoria::class);
    }

    public function imagenes()
    {
        return $this->hasMany(Imagen::class);
    }

    public function orderItems()
    {
        return $this->hasMany(OrderItem::class);
    }
}
```

---

# Imagen Model

```php
class Imagen extends Model
{
    protected $table = "imagenes";
    protected $fillable = ['nombre', 'producto_id'];

    public function producto()
    {
        return $this->belongsTo(Producto::class);
    }
}
```

---

# Order Model

```php
class Order extends Model
{
    protected $fillable = [
        'correlativo',
        'fecha',
        'fecha_despacho',
        'subtotal',
        'impuesto',
        'total',
        'estado',
        'user_id'
    ];

    protected $casts = [
        'fecha' => 'date',
        'fecha_despacho' => 'date',
        'subtotal' => 'decimal:2',
        'impuesto' => 'decimal:2',
        'total' => 'decimal:2'
    ];

    public function user()
    {
        return $this->belongsTo(User::class);
    }

    public function items()
    {
        return $this->hasMany(OrderItem::class);
    }

    public function pagos()
    {
        return $this->hasMany(Pago::class);
    }
}
```

---

# OrderItem Model

```php
class OrderItem extends Model
{
    protected $table = "order_items";

    protected $fillable = [
        'cantidad',
        'precio_unitario',
        'subtotal',
        'producto_id',
        'order_id'
    ];

    protected $casts = [
        'precio_unitario' => 'decimal:2',
        'subtotal' => 'decimal:2'
    ];

    public function producto()
    {
        return $this->belongsTo(Producto::class);
    }

    public function order()
    {
        return $this->belongsTo(Order::class);
    }
}
```

---

# Pago Model

```php
class Pago extends Model
{
    protected $fillable = [
        'metodo',
        'referencia',
        'monto',
        'estado',
        'respuesta_pasarela',
        'order_id'
    ];

    protected $casts = [
        'monto' => 'decimal:2',
        'respuesta_pasarela' => 'array'
    ];

    public function order()
    {
        return $this->belongsTo(Order::class);
    }
}
```

---

# 5️ Ejemplos de Uso de Relaciones

## 🔹 Obtener productos con marca y categoría

```php
Producto::with(['marca', 'categoria'])->get();
```

## 🔹 Obtener una orden con detalles y pagos

```php
Order::with(['items.producto', 'pagos'])->find($id);
```

## 🔹 Crear una orden con items

```php
$order = Order::create([...]);

$order->items()->create([
    'producto_id' => 1,
    'cantidad' => 2,
    'precio_unitario' => 100,
    'subtotal' => 200
]);
```

---

# 6️ Diagrama Lógico de Relaciones

User (1) → (N) Orders  
Order (1) → (N) OrderItems  
Order (1) → (N) Pagos  
Producto (1) → (N) Imagenes  
Marca (1) → (N) Productos  
Categoria (1) → (N) Productos  
Producto (1) → (N) OrderItems  

---

# ✅ Conclusión

En esta guía aprendiste:

- Cómo crear modelos en Laravel 12.
- Propiedades fundamentales de un modelo.
- Tipos de relaciones en Eloquent.
- Implementación completa de los modelos para Shop-APP.
- Ejemplos prácticos de uso.

**Nota: Desarrollar guía en el proyecto de clase, subir evidencia**
---

--> Base sólida para continuar con controladores y probar la API

