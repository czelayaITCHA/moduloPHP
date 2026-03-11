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
    },
    //método agregado para activar/desactivar
    toggleActivo(id){
        return api.patch(`/productos/${id}/toggle-activo`)
    },

}
```
## 11.1.2 Programar el componentes para gestionar productos
* Ya se tiene crea el archivo views/admin/Productos.vue, agregar el siguiente código - actualizado

````vue
<template>
  <div class="card">
    <Toast />
    <ConfirmDialog />

    <ProductoTable
      :productos="productos"
      :marcas="marcas"
      @toggle="toggleActivo"
      @nuevo="openNew"
      @editar="editProducto"
      @eliminar="confirmDelete"
    />

    <ProductoForm
      v-model:visible="productoDialog"
      :producto="producto"
      @guardar="guardarProducto"
    />
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import { useToast } from "primevue/usetoast";
import { useConfirm } from "primevue/useconfirm";

import productoService from "@/services/productoService";
import marcaService from "@/services/marcaService";

import ProductoTable from "@/components/admin/productos/ProductoTable.vue";
import ProductoForm from "@/components/admin/productos/ProductoForm.vue";

const toast = useToast();
const confirm = useConfirm();

const productos = ref([]);
const productoDialog = ref(false);
const producto = ref({});
const marcas = ref([]);

const loadProductos = async () => {
  const res = await productoService.getAll();
  productos.value = res.data;
};

const loadMarcas = async () => {
  const response = await marcaService.getAll();
  marcas.value = response.data;
};

const loadData = () => {
  loadMarcas();
  loadProductos();
};
onMounted(loadData);

const openNew = () => {
  producto.value = {};
  productoDialog.value = true;
};

const editProducto = (p) => {
  producto.value = { ...p };
  productoDialog.value = true;
};

const guardarProducto = async (formData, id) => {
  confirm.require({
    message: "¿Desea guardar los cambios?",
    header: "Confirmación",
    icon: "pi pi-exclamation-triangle",

    accept: async () => {
      try {
        let res;

        if (id) {
          res = await productoService.update(id, formData);
        } else {
          res = await productoService.store(formData);
        }

        toast.add({
          severity: "success",
          summary: "Correcto",
          detail: res.data.message,
          life: 3000,
        });

        productoDialog.value = false;
        loadProductos();
      } catch (error) {
        if (error.response?.status === 422) {
          const errores = error.response.data.errors;

          Object.values(errores).forEach((e) => {
            toast.add({
              severity: "error",
              summary: "Error",
              detail: e[0],
              life: 4000,
            });
          });
        } else {
          toast.add({
            severity: "error",
            summary: "Error",
            detail: error.response?.data?.message || "Error",
            life: 4000,
          });
        }
      }
    },
  });
};

const confirmDelete = (p) => {
  confirm.require({
    message: `¿Eliminar el producto ${p.nombre}?`,
    header: "Confirmar",
    icon: "pi pi-info-circle",

    accept: async () => {
      try {
        const res = await productoService.delete(p.id);
        toast.add({
          severity: "success",
          summary: "Eliminado",
          detail: res.data.message,
          life: 3000,
        });
        loadProductos();
      } catch (error) {
        if (error.response) {
          const data = error.response.data;
          // capturando errores de validación
          if (data.errors) {
            Object.values(data.errors).forEach((err) => {
              toast.add({
                severity: "warn",
                summary: "Validación",
                detail: err[0],
                life: 4000,
              });
            });
          } else {
            toast.add({
              severity: "warn",
              summary: "Aviso",
              detail: data.message,
              life: 4000,
            });
          }
        } else {
          toast.add({
            severity: "error",
            summary: "Error",
            detail: "No se pudo conectar con el servidor",
            life: 4000,
          });
        }
      }
    },
  });
};

//función para activar/desactivar
const toggleActivo = async (producto) => {
  try {
    const res = await productoService.toggleActivo(producto.id);

    toast.add({
      severity: "success",
      summary: "Estado actualizado",
      detail: res.data.message,
      life: 3000,
    });
    loadProductos();
  } catch (error) {
    toast.add({
      severity: "error",
      summary: "Error",
      detail: error.response?.data?.message || "Error del servidor",
      life: 4000,
    });
  }
};
</script>

````

* Crear el componente components/admin/productos/ProductoTable.vue
````vue
<template>
  <DataTable
    :value="productosFiltrados"
    paginator
    :rows="10"
    :rowsPerPageOptions="[5, 10, 20, 50]"
    dataKey="id"
    :filters="filters"
    :globalFilterFields="['nombre', 'descripcion', 'modelo', 'marca.nombre']"
    responsiveLayout="scroll"
  >
    <!-- HEADER -->

    <template #header>
      <div class="table-header">
        <h3 class="text-2xl">Gestión de Productos</h3>

        <div class="header-filters">
          <span class="p-input-icon-left">
            <i class="pi pi-search" />
            <InputText v-model="filters['global'].value" placeholder="Buscar producto" />
          </span>

          <Dropdown
            v-model="marcaSeleccionada"
            :options="marcas"
            optionLabel="nombre"
            placeholder="Filtrar marca"
            class="marca-dropdown"
            showClear
          />

          <Button label="Nuevo" icon="pi pi-plus" severity="success" @click="$emit('nuevo')" />
        </div>
      </div>
    </template>

    <!-- COLUMNAS -->

    <Column field="nombre" header="Nombre" sortable />

    <Column field="descripcion" header="Descripción" />

    <Column field="modelo" header="Modelo" sortable />

    <!-- Marca -->

    <Column header="Marca">
      <template #body="slotProps">
        {{ slotProps.data.marca?.nombre }}
      </template>
    </Column>

    <!-- Precio -->

    <Column header="Precio" sortable>
      <template #body="slotProps">
        {{ formatCurrency(slotProps.data.precio) }}
      </template>
    </Column>

    <Column field="stock" header="Stock" sortable />

    <Column header="Activo">
      <template #body="slotProps">
        <InputSwitch :modelValue="slotProps.data.activo" @change="emit('toggle', slotProps.data)" />
      </template>
    </Column>

    <!-- ACCIONES -->

    <Column header="Acciones" style="width: 140px">
      <template #body="slotProps">
        <div class="acciones">
          <Button icon="pi pi-pencil" severity="warning" @click="$emit('editar', slotProps.data)" />

          <Button icon="pi pi-trash" severity="danger" @click="$emit('eliminar', slotProps.data)" />
        </div>
      </template>
    </Column>
  </DataTable>
</template>

<script setup>
import { ref, computed } from "vue";
import { FilterMatchMode } from "primevue/api";

const emit = defineEmits(["edit", "toggle", "delete"]);

const props = defineProps({
  productos: Array,
  marcas: Array,
});

const filters = ref({
  global: { value: null, matchMode: FilterMatchMode.CONTAINS },
});

const marcaSeleccionada = ref(null);

const productosFiltrados = computed(() => {
  if (!marcaSeleccionada.value) {
    return props.productos;
  }

  return props.productos.filter((p) => p.marca?.id === marcaSeleccionada.value.id);
});

const formatCurrency = (value) => {
  return new Intl.NumberFormat("es-SV", {
    style: "currency",
    currency: "USD",
  }).format(value);
};
</script>

<style scoped>
.table-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 10px;
  margin-bottom: 10px;
}

.header-filters {
  display: flex;
  gap: 10px;
  align-items: center;
  flex-wrap: wrap;
}

.marca-dropdown {
  min-width: 160px;
}

.acciones {
  display: flex;
  gap: 8px;
}

/* RESPONSIVO */

@media (max-width: 768px) {
  .table-header {
    flex-direction: column;
    align-items: flex-start;
  }

  .header-filters {
    width: 100%;
    flex-direction: column;
    align-items: stretch;
  }

  .marca-dropdown {
    width: 100%;
  }
}
</style>

```` 
* Crear el componente components/admin/productos/ProductoForm.vue

````vue
<template>
  <Dialog v-model:visible="dialogVisible" :header="titleDialog" :style="{ width: '600px' }">
    <div class="p-fluid grid">
      <div class="col-12">
        <label class="form-label">Nombre</label>
        <InputText v-model="producto.nombre" placeholder="Nombre" />
      </div>

      <div class="col-12 mt-2">
        <label class="form-label">Descripción</label>
        <Textarea v-model="producto.descripcion" rows="3" placeholder="Descripción" />
      </div>

      <div class="col-6">
        <label class="form-label">Modelo</label>
        <InputText v-model="producto.modelo" placeholder="Modelo" />
      </div>

      <div class="col-6 mt-2">
        <label class="form-label">Precio</label>
        <InputNumber v-model="producto.precio" placeholder="Precio" />
      </div>

      <div class="col-6 mt-2">
        <label class="form-label">Stock</label>
        <InputNumber v-model="producto.stock" placeholder="Stock" />
      </div>

      <div class="col-6 mt-2">
        <label class="form-label">Estado</label>
        <Dropdown
          v-model="producto.activo"
          :options="estados"
          optionLabel="label"
          optionValue="value"
          placeholder="Estado"
        />
      </div>

      <div class="col-6 mt-2">
        <label class="form-label">Marca</label>
        <Dropdown
          v-model="producto.marca"
          :options="marcas"
          optionLabel="nombre"
          placeholder="Marca"
        />
      </div>

      <div class="col-6 mt-2">
        <label class="form-label">Categoría</label>
        <Dropdown
          v-model="producto.categoria"
          :options="categorias"
          optionLabel="nombre"
          placeholder="Categoría"
        />
      </div>
    </div>

    <h4 class="mt-4">Imágenes</h4>

    <FileUpload multiple customUpload mode="basic" accept="image/*" @select="onSelect" />

    <div class="flex gap-3 mt-3">
      <img
        v-for="img in preview"
        :key="img.url"
        :src="img.url"
        style="width: 80px; height: 80px; object-fit: cover"
      />
    </div>

    <template #footer>
      <Button label="Cancelar" icon="pi pi-times" severity="secondary" @click="close" />

      <Button :label="labelButton" icon="pi pi-check" @click="submit" />
    </template>
  </Dialog>
</template>

<script setup>
import { ref, watch, onMounted, computed } from "vue";

import categoriaService from "@/services/categoriaService";
import marcaService from "@/services/marcaService";

const BASE_IMAGE_URL = "http://localhost:8000/images/productos/";

const props = defineProps({
  visible: Boolean,
  producto: Object,
});

const emit = defineEmits(["update:visible", "guardar"]);

//propiedades computables
const dialogVisible = computed({
  get: () => props.visible,
  set: (val) => emit("update:visible", val),
});

//funciones computables para determinar si esta agregando o etidando un registro
const titleDialog = computed(() => {
  return producto.value.id ? "Edición de Productos" : "Registro de Productos";
});

const labelButton = computed(() => {
  return producto.value.id ? "Actualizar" : "Guardar";
});

const producto = ref({});
const categorias = ref([]);
const marcas = ref([]);
const preview = ref([]);
const imagenes = ref([]);

const estados = [
  { label: "Activo", value: 1 },
  { label: "Inactivo", value: 0 },
];

const loadPreviewImages = (producto) => {
  preview.value = [];
  if (producto?.imagenes) {
    producto.imagenes.forEach((img) => {
      preview.value.push({
        id: img.id,
        url: BASE_IMAGE_URL + img.nombre,
        existente: true,
      });
    });
  }
};
watch(
  () => props.producto,
  (v) => {
    // Si no hay valor, reseteamos y salimos
    if (!v) {
      producto.value = {};
      return;
    }

    // Mapeo del objeto con conversión numérica
    producto.value = {
      ...v,
      precio: v.precio != null ? Number(v.precio) : null,
      stock: v.stock != null ? Number(v.stock) : null,
    };

    loadPreviewImages(v);
  },
  { immediate: true },
);

const loadData = async () => {
  const c = await categoriaService.getAll();
  categorias.value = c.data;

  const m = await marcaService.getAll();
  marcas.value = m.data;
};

onMounted(loadData);

const onSelect = (e) => {
  e.files.forEach((file) => {
    imagenes.value.push(file);

    preview.value.push({
      url: URL.createObjectURL(file),
      existente: false,
    });
  });
};

const close = () => emit("update:visible", false);

const submit = () => {
  const formData = new FormData();

  formData.append("producto", JSON.stringify(producto.value));

  imagenes.value.forEach((i) => {
    formData.append("imagenes[]", i);
  });

  emit("guardar", formData, producto.value.id);
};
</script>

<style scoped>
.form-label {
  display: block;
  font-weight: 600;
  margin-bottom: 4px;
  font-size: 0.9rem;
  color: #444;
}

.imagenes-preview {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin-top: 10px;
}

.preview-img {
  width: 80px;
  height: 80px;
  object-fit: cover;
  border-radius: 4px;
  border: 1px solid #ddd;
}
</style>
````

* Importante revisar y crear los métodos de ProductoController

método **destroy** se dejó como tarea
```php
public function destroy(string $id)
    {
         try {

            DB::beginTransaction();

            // Buscar producto
            $producto = Producto::with('imagenes','orderItems')->findOrFail($id);

            // Validar si tiene órdenes asociadas
            if ($producto->orderItems()->exists()) {
                return response()->json([
                    'message' => 'No se puede eliminar porque tiene ordenes registradas'
                ], 409);
            }

            // Eliminar imágenes físicas y registros
            foreach ($producto->imagenes as $imagen) {

                $ruta = public_path('images/productos/'.$imagen->nombre);

                if (File::exists($ruta)) {
                    File::delete($ruta);
                }

                $imagen->delete();
            }

            // Eliminar producto
            $producto->delete();

            DB::commit();

            return response()->json([
                'message' => 'Producto eliminado correctamente'
            ], 200);

        } catch (ModelNotFoundException $e) {

            DB::rollBack();

            return response()->json([
                'message' => 'Producto no encontrado con el ID = '.$id
            ], 404);

        } catch (\Exception $e) {

            DB::rollBack();

            return response()->json([
                'message' => 'Error interno del servidor'
            ], 500);
        }

    }
```
método toogleActivo para activar/desactivar producto
```php
    public function toggleActivo($id)
    {

        $producto = Producto::findOrFail($id);

        $producto->activo = !$producto->activo;

        $producto->save();

        return response()->json([
            'message' => $producto->activo
                ? 'Producto activado'
                : 'Producto desactivado'
        ]);

    }
```
* Crear ruta en api.php para activar/desactivar producto
```php
Route::patch('productos/{id}/toggle-activo',[ProductoController::class,'toggleActivo']);
```
