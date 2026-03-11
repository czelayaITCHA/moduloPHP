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
## 11.1.2 Programar el componentes para gestionar productos
* Ya se tiene crea el archivo views/admin/Productos.vue, agregar el siguiente código
````vue
<template>
  <div class="card">
    <Toast />
    <ConfirmDialog />

    <ProductoTable
      :productos="productos"
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

import ProductoTable from "@/components/admin/productos/ProductoTable.vue";
import ProductoForm from "@/components/admin/productos/ProductoForm.vue";

const toast = useToast();
const confirm = useConfirm();

const productos = ref([]);
const productoDialog = ref(false);
const producto = ref({});

const loadProductos = async () => {
  const res = await productoService.getAll();
  productos.value = res.data;
};

onMounted(loadProductos);

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
    message: `¿Eliminar ${p.nombre}?`,
    header: "Confirmar",
    icon: "pi pi-info-circle",

    accept: async () => {
      const res = await productoService.delete(p.id);

      toast.add({
        severity: "success",
        summary: "Eliminado",
        detail: res.data.message,
        life: 3000,
      });

      loadProductos();
    },
  });
};
</script>
````

* Crear el componente components/admin/productos/ProductoTable.vue
  ````vue
  <template>

<DataTable :value="productos" paginator :rows="10" responsiveLayout="scroll">

    <template #header>
        <div class="table-header">
            <h3>Productos</h3>

            <Button
            label="Nuevo"
            icon="pi pi-plus"
            severity="success"
            @click="$emit('nuevo')"
        />

        </div>
    </template>

    <Column field="nombre" header="Nombre" />

    <Column field="descripcion" header="Descripción" />

    <Column field="modelo" header="Modelo" />

    <!-- Marca -->
    <Column header="Marca">
    <template #body="slotProps">
        {{ slotProps.data.marca?.nombre }}
    </template>
    </Column>

    <!-- Precio -->
    <Column header="Precio">
    <template #body="slotProps">
        {{ formatCurrency(slotProps.data.precio) }}
    </template>
    </Column>

    <Column field="stock" header="Stock" />

    <!-- Acciones -->
    <Column header="Acciones" style="width:150px">

    <template #body="slotProps">

        <div class="acciones">

            <Button
            icon="pi pi-pencil"
            severity="warning"
            class="btn-editar"
            @click="$emit('editar',slotProps.data)"
            />

            <Button
            icon="pi pi-trash"
            severity="danger"
            class="btn-eliminar"
            @click="$emit('eliminar',slotProps.data)"
            />

        </div>

    </template>

    </Column>

</DataTable>

</template>

<script setup>

defineProps({
    productos: Array
})

const formatCurrency = (value) => {
    return new Intl.NumberFormat('es-SV', {
        style: 'currency',
        currency: 'USD'
    }).format(value)

}

</script>

<style scoped>

.table-header{
display:flex;
justify-content:space-between;
align-items:center;
margin-bottom:10px;
}

.acciones{
display:flex;
gap:10px;
}

.btn-editar{
margin-right:6px;
}

</style>
  ```` 
* Crear el componente components/admin/productos/ProductoForm.vue
````vue
<template>
  <Dialog
    v-model:visible="dialogVisible"
    header="Producto"
    :style="{ width: '600px' }"
  >
    <div class="p-fluid grid">
      <div class="col-12">
        <InputText v-model="producto.nombre" placeholder="Nombre" />
      </div>

      <div class="col-12">
        <Textarea
          v-model="producto.descripcion"
          rows="3"
          placeholder="Descripción"
        />
      </div>

      <div class="col-6">
        <InputText v-model="producto.modelo" placeholder="Modelo" />
      </div>

      <div class="col-6">
        <InputNumber v-model="producto.precio" placeholder="Precio" />
      </div>

      <div class="col-6">
        <InputNumber v-model="producto.stock" placeholder="Stock" />
      </div>

      <div class="col-6">
        <Dropdown
          v-model="producto.activo"
          :options="estados"
          optionLabel="label"
          optionValue="value"
          placeholder="Estado"
        />
      </div>

      <div class="col-6">
        <Dropdown
          v-model="producto.marca"
          :options="marcas"
          optionLabel="nombre"
          placeholder="Marca"
        />
      </div>

      <div class="col-6">
        <Dropdown
          v-model="producto.categoria"
          :options="categorias"
          optionLabel="nombre"
          placeholder="Categoría"
        />
      </div>
    </div>

    <h4 class="mt-4">Imágenes</h4>

    <FileUpload
      multiple
      customUpload
      mode="basic"
      accept="image/*"
      @select="onSelect"
    />

    <div class="flex gap-3 mt-3">
      <img
        v-for="img in preview"
        :key="img.url"
        :src="img.url"
        style="width: 80px; height: 80px; object-fit: cover"
      />
    </div>

    <template #footer>
      <Button
        label="Cancelar"
        icon="pi pi-times"
        severity="secondary"
        @click="close"
      />

      <Button label="Guardar" icon="pi pi-check" @click="submit" />
    </template>
  </Dialog>
</template>

<script setup>
import { ref, watch, onMounted, computed } from "vue";

import categoriaService from "@/services/categoriaService";
import marcaService from "@/services/marcaService";


const props = defineProps({
  visible: Boolean,
  producto: Object,
});

const emit = defineEmits(["update:visible", "guardar"]);

const dialogVisible = computed({
  get: () => props.visible,
  set: (val) => emit("update:visible", val),
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
    preview.value = []
    if (producto?.imagenes) {
        producto.imagenes.forEach(img => {
            preview.value.push({
                id: img.id,
                url: img.url,
                existente: true
            })

        })

    }

}


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
      stock: v.stock != null ? Number(v.stock) : null
    };

    loadPreviewImages(v);
  },
  { immediate: true }
);

const loadData = async () => {
  const c = await categoriaService.getAll();
  categorias.value = c.data;

  const m = await marcaService.getAll();
  marcas.value = m.data;
};

onMounted(loadData);

const onSelect = (e) => {

    e.files.forEach(file => {

        imagenes.value.push(file)

        preview.value.push({
            url: URL.createObjectURL(file),
            existente: false
        })

    })

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
````
