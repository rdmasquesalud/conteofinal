
<template>
 
  <div id="app">
    <div class="container">
      <div v-if="paso === 1">
        <h2>Producto</h2>
        <div class="form-group">
          <input v-model="codigo" id="codigo" type="text" placeholder="Código del Producto" maxlength="10" @keyup.enter="siguientePaso" ref="codigo">
        </div>
        <div class="form-group">
          
          <input v-model="descripcion" id="descripcion" type="text" placeholder="Descripción del Producto" @keyup.enter="siguientePaso" ref="descripcion">
        </div>
        <button @click="siguientePaso">Siguiente</button>
      </div>

      <div v-if="paso === 2">
        <h2>Detalles del Producto</h2>
        <div class="form-group">
          <label for="cantidadPiso1">C/U Piso 1:</label>
          <input v-model.number="cantidadPiso1" id="cantidadPiso1" type="number" placeholder="C/U Piso 1" maxlength="4" @keyup.enter="focusNext('cantidadPiso2')" ref="cantidadPiso1" @input="updateCantidadPiso1FromInput">
        </div>
        <div class="form-group">
          <label for="L1">L1:</label>
          <input v-model.number="L1" id="L1" type="number" placeholder="L1" maxlength="4" @keyup.enter="focusNext('L2')" ref="L1" @input="updateCantidadPiso1">
          <span>×</span>
          <label for="L2">L2:</label>
          <input v-model.number="L2" id="L2" type="number" placeholder="L2" maxlength="4" @keyup.enter="focusNext('cantidadPiso2')" ref="L2" @input="updateCantidadPiso1">
        </div>
        <div class="form-group">
          <label for="cantidadPiso2">C/U Piso 2:</label>
          <input v-model.number="cantidadPiso2" id="cantidadPiso2" type="number" placeholder="C/U Piso 2" maxlength="4" @keyup.enter="focusNext('cantidadPisos')" ref="cantidadPiso2" @input="updateCantidadPiso2FromInput">
        </div>
        <div class="form-group">
          <label for="L3">L1:</label>
          <input v-model.number="L3" id="L3" type="number" placeholder="L1" maxlength="4" @keyup.enter="focusNext('L4')" ref="L3" @input="updateCantidadPiso2">
          <span>×</span>
          <label for="L4">L2:</label>
          <input v-model.number="L4" id="L4" type="number" placeholder="L2" maxlength="4" @keyup.enter="focusNext('cantidadPisos')" ref="L4" @input="updateCantidadPiso2">
        </div>
        <div class="form-group">
          <label for="cantidadPisos">N° Pisos:</label>
          <input v-model.number="cantidadPisos" id="cantidadPisos" type="number" placeholder="N° Pisos" maxlength="4" @keyup.enter="focusNext('pisosCompletos')" ref="cantidadPisos">
        </div>
        <div class="form-group">
          <label for="pisosCompletos">Pisos Completos:</label>
          <input v-model.number="pisosCompletos" id="pisosCompletos" type="number" placeholder="Pisos Completos" maxlength="4" @keyup.enter="focusNext('cantidadExistente')" ref="pisosCompletos">
        </div>

        <div v-for="(piso, index) in pisosIncompletos" :key="index" class="form-group">
          <h3>Piso {{ piso }}</h3>
          <div class="form-group">
            <label for="cantidadExistente">Cantidad Existente:</label>
            <input v-model.number="cantidadExistente[piso]" @input="handleExistenteInput(piso)" id="cantidadExistente" type="number" placeholder="Cantidad Existente" maxlength="4" @keyup.enter="focusNext(`cantidadFaltante${piso}`)" :ref="'cantidadExistente' + piso">
          </div>
          <div class="form-group">
            <label for="cantidadFaltante">Cantidad Faltante:</label>
            <input v-model.number="cantidadFaltante[piso]" @input="handleFaltanteInput(piso)" id="cantidadFaltante" type="number" placeholder="Cantidad Faltante" maxlength="4" @keyup.enter="focusNext(`cantidadExistente${pisosIncompletos[index + 1]}`)" :ref="'cantidadFaltante' + piso">
          </div>
          <p>Total: {{ totalPorPiso[piso] }}</p>
        </div>

        <button @click="calcularTotalUnidades">Calcular</button>
      </div>

      <div v-if="paso === 3">
        <h2>Resultado</h2>
        <p>Producto: {{ codigo }}</p>
        <p>Descripción: {{ descripcion }}</p>
        <p>Cantidad total de unidades: {{ cantidadTotalUnidades }}</p>
        <button @click="volverPasoAnterior">Volver</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      paso: 1,
      codigo: '',
      descripcion: '',
      cantidadPiso1: 0,
      cantidadPiso2: 0,
      L1: 0,
      L2: 0,
      L3: 0,
      L4: 0,
      cantidadPisos: 0,
      pisosCompletos: 0,
      cantidadExistente: {},
      cantidadFaltante: {},
      totalPorPiso: {},
      cantidadTotalUnidades: 0
    }
  },
  computed: {
    pisosIncompletos() {
      const pisosTotales = Array.from({ length: this.cantidadPisos }, (_, i) => i + 1);
      return pisosTotales.slice(this.pisosCompletos);
    }
  },
  methods: {
    focusNext(field) {
      this.$nextTick(() => {
        if (this.$refs[field] && this.$refs[field][0]) {
          this.$refs[field][0].focus();
        } else if (this.$refs[field]) {
          this.$refs[field].focus();
        }
      });
    },
    siguientePaso() {
      if (this.codigo || this.descripcion) {
        this.paso = 2;
        this.$nextTick(() => this.focusNext('cantidadPiso1'));
      } else {
        alert('Por favor, complete el código del producto o la descripción.');
      }
    },
    volverPasoAnterior() {
      this.paso = 2;
    },
    handleExistenteInput(piso) {
      if (this.cantidadExistente[piso] !== undefined) {
        this.cantidadFaltante[piso] = undefined;
        this.calcularTotalPiso(piso, 'existente');
      }
    },
    handleFaltanteInput(piso) {
      if (this.cantidadFaltante[piso] !== undefined) {
        this.cantidadExistente[piso] = undefined;
        this.calcularTotalPiso(piso, 'faltante');
      }
    },
    calcularTotalPiso(piso, tipo) {
      const cantidadPorPiso = piso % 2 === 0 ? this.cantidadPiso2 : this.cantidadPiso1;
      if (tipo === 'existente') {
        this.totalPorPiso[piso] = this.cantidadExistente[piso];
      } else {
        this.totalPorPiso[piso] = cantidadPorPiso - this.cantidadFaltante[piso];
      }
    },
    updateCantidadPiso1() {
      if (this.L1 && this.L2) {
        this.cantidadPiso1 = this.L1 * this.L2;
      }
    },
    updateCantidadPiso1FromInput() {
      if (this.cantidadPiso1) {
        this.L1 = 0;
        this.L2 = 0;
      }
    },
    updateCantidadPiso2() {
      if (this.L3 && this.L4) {
        this.cantidadPiso2 = this.L3 * this.L4;
      }
    },
    updateCantidadPiso2FromInput() {
      if (this.cantidadPiso2) {
        this.L3 = 0;
        this.L4 = 0;
      }
    },
    calcularTotalUnidades() {
      let totalCompletos = 0;
      for (let i = 1; i <= this.pisosCompletos; i++) {
        totalCompletos += i % 2 === 0 ? this.cantidadPiso2 : this.cantidadPiso1;
      }
      
      const totalIncompletos = Object.values(this.totalPorPiso).reduce((a, b) => a + b, 0);
      this.cantidadTotalUnidades = totalCompletos + totalIncompletos;
      this.paso = 3;
    }
  },
  mounted() {
    this.$refs.codigo.focus();
  }
}
</script>

<style>
#app {
  font-family: 'Roboto', sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #0e0d0d;
  background-color: #4CAF50;
  padding: 20px;
}

.container {
  max-width: 600px;
  margin: 0 auto;
  text-align: center;
  background-color: #ffffff;
  border-radius: 10px;
  padding: 20px;
  box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1);
}

h2 {
  color: #4CAF50;
}

.form-group {
  margin-bottom: 20px;
  text-align: center;
}

input[type="text"], input[type="number"] {
  padding: 10px;
  box-sizing: border-box;
  border: 1px solid #ccc;
  border-radius: 5px;
  margin-top: 5px;
}

input[type="number"] {
  width: 70px;
}

button {
  padding: 10px 20px;
  background-color: #4CAF50;
  color: #fff;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

button:hover {
  background-color: #45a049;
}

span {
  margin: 0 10px;
}
</style>
