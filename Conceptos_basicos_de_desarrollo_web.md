# Términos Clave en Desarrollo Web con PHP y Laravel

El desarrollo web con PHP y Laravel incluye diversos conceptos fundamentales para construir aplicaciones modernas y eficientes. A continuación, se explican los términos clave y su aplicación en este framework.

---

## 1. **API (Interfaz de Programación de Aplicaciones)**
- **Definición:** 
  Es un conjunto de reglas y protocolos que permiten que diferentes sistemas o aplicaciones se comuniquen entre sí.
- **Ejemplo de uso:**
  En Laravel, una API se puede crear utilizando controladores y rutas específicas para manejar solicitudes HTTP.

---

## 2. **API REST**
- **Definición:** 
  Representational State Transfer (REST) es un estilo arquitectónico que organiza APIs basándose en recursos y métodos HTTP.
- **Características en Laravel:**
  - Laravel facilita la creación de APIs REST con controladores de tipo `api` y middleware como `auth:sanctum` para la autenticación.

---

## 3. **API RESTful**
- **Definición:** 
  Una API que sigue estrictamente los principios REST, como utilizar URLs semánticas y métodos HTTP estándar.
- **Ventajas:** 
  En Laravel, puedes usar `Resource Controllers` para implementar fácilmente APIs RESTful.

---

## 4. **Endpoint**
- **Definición:** 
  Es una URL específica que expone un recurso o realiza una acción en una API.
- **Ejemplo en Laravel:**
  ```php
  Route::get('/api/usuarios/{id}', [UsuarioController::class, 'show']);
