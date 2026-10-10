<template>
  <MainLayout>
    <div class="row justify-content-center">
      <div class="col-lg-10">
        <h1 class="mb-4 text-center">Gestión de Usuarios</h1>

        <!-- Formulario para Crear / Editar -->
        <div class="card shadow-sm mb-4">
          <div class="card-body">
            <h3 class="card-title mb-3">
              {{ isEditing ? 'Actualizar Usuario' : 'Registrar Nuevo Usuario' }}
            </h3>
            
            <form @submit.prevent="handleSubmit">
              <div class="mb-3">
                <label for="name" class="form-label">Nombre</label>
                <input 
                  type="text" 
                  class="form-control" 
                  id="name" 
                  v-model="form.name" 
                  required 
                />
              </div>

              <div class="mb-3">
                <label for="email" class="form-label">Correo Electrónico</label>
                <input 
                  type="email" 
                  class="form-control" 
                  id="email" 
                  v-model="form.email" 
                  required 
                />
              </div>

              <div class="mb-3" v-if="!isEditing">
                <label for="password" class="form-label">Contraseña</label>
                <input 
                  type="password" 
                  class="form-control" 
                  id="password" 
                  v-model="form.password" 
                  required 
                />
              </div>

              <button type="submit" class="btn btn-primary me-2">
                {{ isEditing ? 'Actualizar' : 'Guardar Usuario' }}
              </button>
              
              <button 
                type="button" 
                class="btn btn-secondary" 
                v-if="isEditing" 
                @click="resetForm"
              >
                Cancelar
              </button>
            </form>
          </div>
        </div>

        <!-- Tabla de Usuarios -->
        <div class="card shadow-sm">
          <div class="card-body">
            <h3 class="card-title mb-3">Lista de Usuarios</h3>
            
            <div v-if="errorMessage" class="alert alert-danger" role="alert">
              {{ errorMessage }}
            </div>

            <div class="table-responsive">
              <table class="table table-striped align-middle">
                <thead>
                  <tr>
                    <th>ID</th>
                    <th>Nombre</th>
                    <th>Email</th>
                    <th>Creado</th>
                    <th>Acciones</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="user in users" :key="user.id">
                    <td>{{ user.id }}</td>
                    <td>{{ user.name }}</td>
                    <td>{{ user.email }}</td>
                    <td>{{ new Date(user.created_at).toLocaleDateString() }}</td>
                    <td>
                      <button 
                        class="btn btn-sm btn-warning me-2" 
                        @click="startEdit(user)"
                      >
                        Editar
                      </button>
                      <button 
                        class="btn btn-sm btn-danger" 
                        @click="deleteUser(user.id)"
                      >
                        Eliminar
                      </button>
                    </td>
                  </tr>
                  <tr v-if="users.length === 0">
                    <td colspan="5" class="text-center text-muted">No hay usuarios registrados.</td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
        </div>
      </div>
    </div>
  </MainLayout>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import MainLayout from './layouts/MainLayout.vue';

const API_URL = 'http://localhost:3000/api/users';

const users = ref([]);
const isEditing = ref(false);
const editId = ref(null);
const errorMessage = ref('');

const form = ref({
  name: '',
  email: '',
  password: ''
});

const fetchUsers = async () => {
  try {
    errorMessage.value = '';
    const response = await fetch(API_URL);
    const result = await response.json();
    if (result.success) {
      users.value = result.data;
    } else {
      errorMessage.value = result.message || 'Error al cargar usuarios.';
    }
  } catch (error) {
    console.error('Error de conexión:', error);
    errorMessage.value = 'No se pudo conectar con el servidor backend.';
  }
};

const handleSubmit = async () => {
  try {
    errorMessage.value = '';
    let url = API_URL;
    let method = 'POST';
    let bodyData = { name: form.value.name, email: form.value.email, password: form.value.password };

    if (isEditing.value) {
      url = `${API_URL}/${editId.value}`;
      method = 'PUT';
      bodyData = { name: form.value.name, email: form.value.email };
    }

    const response = await fetch(url, {
      method: method,
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(bodyData)
    });

    const result = await response.json();

    if (response.ok && result.success) {
      resetForm();
      fetchUsers();
    } else {
      errorMessage.value = result.message || 'Ocurrió un error al procesar la solicitud.';
    }
  } catch (error) {
    console.error('Error en la petición:', error);
    errorMessage.value = 'Error al comunicarse con la API.';
  }
};

const startEdit = (user) => {
  isEditing.value = true;
  editId.value = user.id;
  form.value.name = user.name;
  form.value.email = user.email;
  form.value.password = '';
};

const deleteUser = async (id) => {
  if (confirm('¿Estás seguro de que deseas eliminar este usuario?')) {
    try {
      errorMessage.value = '';
      const response = await fetch(`${API_URL}/${id}`, { method: 'DELETE' });
      const result = await response.json();
      
      if (response.ok && result.success) {
        fetchUsers();
      } else {
        errorMessage.value = result.message || 'No se pudo eliminar el usuario.';
      }
    } catch (error) {
      console.error('Error al eliminar:', error);
      errorMessage.value = 'Error de red al intentar eliminar.';
    }
  }
};

const resetForm = () => {
  isEditing.value = false;
  editId.value = null;
  form.value = { name: '', email: '', password: '' };
};

onMounted(() => {
  fetchUsers();
});
</script>