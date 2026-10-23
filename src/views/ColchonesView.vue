<template>
  <main class="colchones">
    <header class="encabezado">
      <div>
        <h1>Colchones</h1>
        <p>Conoce nuestros modelos disponibles.</p>
      </div>
      <button type="button" :disabled="loading" @click="getAllColchones">
        {{ loading ? 'Cargando...' : 'Actualizar' }}
      </button>
    </header>

    <section class="panel-cliente">
      <label for="documento">Número de CC</label>
      <input
        id="documento"
        v-model.trim="documentoCliente"
        type="text"
        inputmode="numeric"
        autocomplete="off"
        placeholder="Ingresa tu número de cédula"
      />
      <p>Ingresa tu CC para comprar un colchón o consultar tus compras.</p>
    </section>

    <p v-if="error" class="mensaje error" role="alert">{{ error }}</p>
    <p v-if="compraError" class="mensaje error" role="alert">{{ compraError }}</p>
    <p v-if="compraMensaje" class="mensaje exito" role="status">{{ compraMensaje }}</p>
    <p v-if="loading" class="mensaje" role="status">Cargando colchones...</p>
    <p v-else-if="!colchones.length" class="mensaje">No hay colchones disponibles.</p>

    <section v-else class="catalogo" aria-label="Catálogo de colchones">
      <article v-for="colchon in colchones" :key="colchon.id" class="tarjeta">
        <img :src="colchon.imagen" :alt="colchon.nombre" class="imagen" />
        <div class="detalle">
          <h2>{{ colchon.nombre }}</h2>
          <p v-if="colchon.medidas">Medidas: {{ colchon.medidas }}</p>
          <p class="precio">{{ formatoPrecio(colchon.precio) }}</p>
          <button
            type="button"
            :disabled="!documentoValido || comprandoId !== null"
            @click="comprarColchon(colchon)"
          >
            {{ comprandoId === colchon.id ? 'Procesando...' : 'Comprar' }}
          </button>
        </div>
      </article>
    </section>

    <section class="compras">
      <div class="encabezado-compras">
        <div>
          <h2>Mis colchones comprados</h2>
          <p>Consulta las compras asociadas al número de CC ingresado.</p>
        </div>
        <button
          type="button"
          :disabled="!documentoValido || cargandoCompras"
          @click="getColchonesComprados"
        >
          {{ cargandoCompras ? 'Consultando...' : 'Consultar compras' }}
        </button>
      </div>

      <p v-if="comprasError" class="mensaje error" role="alert">{{ comprasError }}</p>
      <p v-else-if="cargandoCompras" class="mensaje" role="status">Consultando compras...</p>
      <p v-else-if="comprasConsultadas && !colchonesComprados.length" class="mensaje">
        No hay colchones comprados para esta CC.
      </p>

      <div v-else-if="colchonesComprados.length" class="catalogo">
        <article
          v-for="colchon in colchonesComprados"
          :key="colchon.id_compra"
          class="tarjeta"
        >
          <img :src="colchon.imagen" :alt="colchon.nombre" class="imagen" />
          <div class="detalle">
            <h3>{{ colchon.nombre }}</h3>
            <p v-if="colchon.medidas">Medidas: {{ colchon.medidas }}</p>
            <p class="precio">{{ formatoPrecio(colchon.precio) }}</p>
          </div>
        </article>
      </div>
    </section>
  </main>
</template>

<script>
const API_URL = (process.env.VUE_APP_API_URL || 'http://192.168.1.254:3000').replace(/\/$/, '')

export default {
  name: 'ColchonesView',

  data() {
    return {
      colchones: [],
      loading: false,
      error: '',
      documentoCliente: '',
      comprandoId: null,
      compraError: '',
      compraMensaje: '',
      colchonesComprados: [],
      comprasConsultadas: false,
      cargandoCompras: false,
      comprasError: '',
    }
  },

  computed: {
    documentoValido() {
      return /^\d+$/.test(this.documentoCliente)
    },
  },

  watch: {
    documentoCliente() {
      this.colchonesComprados = []
      this.comprasConsultadas = false
      this.comprasError = ''
      this.compraError = ''
      this.compraMensaje = ''
    },
  },

  created() {
    this.getAllColchones()
  },

  methods: {
    async getAllColchones() {
      this.loading = true
      this.error = ''

      try {
        const { data } = await this.axios.get(`${API_URL}/getallcolchones`)
        this.colchones = data
      } catch (error) {
        this.error = error.response?.data?.error || 'No se pudieron cargar los colchones.'
      } finally {
        this.loading = false
      }
    },

    async comprarColchon(colchon) {
      this.compraError = ''
      this.compraMensaje = ''

      if (!this.documentoValido) {
        this.compraError = 'Ingresa un número de CC válido para realizar la compra.'
        return
      }

      this.comprandoId = colchon.id

      try {
        const { data } = await this.axios.post(`${API_URL}/buycolchon`, {
          id_colchon: colchon.id,
          documento_cliente: this.documentoCliente,
        })
        this.compraMensaje = data.message || 'Compra registrada correctamente.'
        await this.getColchonesComprados()
      } catch (error) {
        this.compraError = error.response?.data?.error || 'No se pudo registrar la compra.'
      } finally {
        this.comprandoId = null
      }
    },

    async getColchonesComprados() {
      this.comprasError = ''
      this.comprasConsultadas = false

      if (!this.documentoValido) {
        this.comprasError = 'Ingresa un número de CC válido para consultar tus compras.'
        return
      }

      this.cargandoCompras = true

      try {
        const { data } = await this.axios.get(`${API_URL}/getcolchonescomprados`, {
          params: { documento_cliente: this.documentoCliente },
        })
        this.colchonesComprados = data
        this.comprasConsultadas = true
      } catch (error) {
        this.comprasError = error.response?.data?.error || 'No se pudieron consultar tus compras.'
      } finally {
        this.cargandoCompras = false
      }
    },

    formatoPrecio(precio) {
      return new Intl.NumberFormat('es-CO', {
        style: 'currency',
        currency: 'COP',
        maximumFractionDigits: 0,
      }).format(precio)
    },
  },
}
</script>

<style scoped>
.colchones {
  max-width: 1100px;
  margin: 2rem auto;
  padding: 0 1rem;
  font-family: system-ui, sans-serif;
}

.encabezado {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  margin-bottom: 1.5rem;
}

.encabezado h1,
.encabezado p,
.detalle h2,
.detalle h3,
.detalle p {
  margin: 0;
}

.encabezado p {
  margin-top: 0.25rem;
  color: #555;
}

button {
  padding: 0.65rem 1rem;
  border: 0;
  border-radius: 4px;
  background: #2563eb;
  color: #fff;
  cursor: pointer;
}

button:disabled {
  cursor: wait;
  opacity: 0.6;
}

.catalogo {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 1.25rem;
}

.panel-cliente {
  display: grid;
  gap: 0.5rem;
  max-width: 420px;
  margin-bottom: 1.5rem;
}

.panel-cliente label {
  font-weight: 600;
}

.panel-cliente input {
  padding: 0.65rem;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.panel-cliente p,
.encabezado-compras p {
  margin: 0;
  color: #555;
}

.tarjeta {
  overflow: hidden;
  border: 1px solid #ddd;
  border-radius: 8px;
  background: #fff;
}

.imagen {
  display: block;
  width: 100%;
  height: 190px;
  object-fit: cover;
}

.detalle {
  display: grid;
  gap: 0.5rem;
  padding: 1rem;
}

.detalle h2 {
  font-size: 1.1rem;
}

.detalle h3 {
  font-size: 1rem;
}

.detalle p {
  color: #555;
}

.detalle .precio {
  color: #166534;
  font-weight: 700;
}

.mensaje {
  padding: 0.75rem 1rem;
  border-radius: 4px;
  background: #f3f4f6;
}

.mensaje.error {
  background: #fee2e2;
  color: #991b1b;
}

.mensaje.exito {
  background: #dcfce7;
  color: #166534;
}

.compras {
  margin-top: 2.5rem;
}

.encabezado-compras {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  margin-bottom: 1rem;
}

.encabezado-compras h2 {
  margin: 0 0 0.25rem;
}

@media (max-width: 560px) {
  .encabezado,
  .encabezado-compras {
    align-items: flex-start;
    flex-direction: column;
  }
}
</style>
