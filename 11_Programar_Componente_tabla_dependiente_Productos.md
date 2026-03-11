# 11. Programar componente de Productos.vue
Para este componente se desarrollará en partes, por lo tanto debes crear una carpeta productos, dentro de componentes/admin

## 11.1 Crear los siguientes archivos dentro de la carpeta services
* marcaService.js
```js
import api from "./api"

export default {

    getAll(){
        return api.get("/marcas")
    }

}
```
* categoriaService.js
```js
import api from "./api"

export default {

    getAll(){
        return api.get("/categorias")
    }

}
```
* productoService.js
```js
import api from "./api";

export default {

    getAll(){
        return api.get("/productos");
    },

    store(formData){
        return api.post("/productos", formData, {
            headers:{
                "Content-Type":"multipart/form-data"
            }
        });
    },

    update(id, formData){
        return api.post(`/productos/${id}?_method=PUT`, formData, {
            headers:{
                "Content-Type":"multipart/form-data"
            }
        });
    },

    delete(id){
        return api.delete(`/productos/${id}`);
    }

}
```
