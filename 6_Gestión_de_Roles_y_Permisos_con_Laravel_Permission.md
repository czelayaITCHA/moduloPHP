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
