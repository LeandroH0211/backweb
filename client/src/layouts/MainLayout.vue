<template>
  <div class="d-flex flex-column min-vh-100">
    <!-- Barra de Navegación -->
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark shadow-sm">
      <div class="container">
        <a class="navbar-brand fw-bold" href="#">Panel administrativo</a>
        <span class="navbar-text text-light small">
        </span>
      </div>
    </nav>

    <!-- Contenido Principal -->
    <main class="flex-grow-1 py-4 container">
      <slot></slot> 
    </main>

    <!-- Pie de Página -->
    <footer class="footer py-3 border-top text-center">
      <div class="container text-muted small">
        Universidad Santiago de Cali &copy; 2026
      </div>
    </footer>
  </div>
</template>

<script>
export default {
  name: 'MainLayout'
}
</script>