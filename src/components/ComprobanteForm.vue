<template>
  <div class="container my-5">
    <div class="card shadow">
      <div class="card-header text-center">
        <h3>SPLANA</h3>
      </div>
      <div class="card-body">
        <form @submit.prevent="generarPDF">
          <div class="row mb-3">
            <label class="col-sm-4 col-form-label">Tipo de comprobante:</label>
            <div class="col-sm-8">
              <select v-model="form.tipoComprobante" class="form-control input-red" required @change="ajustarSerie">
                <option value="01">Factura Electrónica</option>
                <option value="03">Boleta de Venta Electrónica</option>
                <option value="07">Nota de Crédito Electrónica</option>
                <option value="08">Nota de Débito Electrónica</option>
              </select>
            </div>
          </div>
          <div class="row mb-3">
            <label class="col-sm-4 col-form-label">Serie:</label>
            <div class="col-sm-8">
              <input
                v-model="form.serie"
                type="text"
                class="form-control input-red"
                required
                maxlength="10"
                @blur="ajustarSerie"
              />
              <small class="form-text text-muted">Ej: 104 → se convierte en F104, B104, NC104 o ND104 según el tipo</small>
            </div>
          </div>
          <div class="row mb-3">
            <label class="col-sm-4 col-form-label">Número:</label>
            <div class="col-sm-8">
              <input
                v-model="form.numero"
                type="number"
                class="form-control input-red"
                required
                min="1"
                @blur="validarNumero"
              />
              <small v-if="errores.numero" class="text-danger">{{ errores.numero }}</small>
            </div>
          </div>
          <div class="row mb-3 align-items-center">
            <label class="col-sm-4 col-form-label">Fecha de emisión:</label>
            <div class="col-sm-8">
              <div class="input_span input-group">
                <input
                  ref="fechaInput"
                  v-model="form.fecha"
                  type="date"
                  class="form-control input-red"
                  required
                />
                <span class="input-group-text" @click="abrirCalendario">
                  <i class="fas fa-calendar-alt"></i>
                </span>
              </div>
            </div>
          </div>
          <div class="row mb-3">
            <label class="col-sm-4 col-form-label">RUC / DNI:</label>
            <div class="col-sm-8">
              <input
                v-model="form.ruc"
                type="text"
                class="form-control input-red"
                maxlength="11"
                required
                @blur="validarRuc"
              />
              <small v-if="errores.ruc" class="text-danger">{{ errores.ruc }}</small>
            </div>
          </div>
          <div class="row mb-3">
            <label class="col-sm-4 col-form-label">Tipo de moneda:</label>
            <div class="col-sm-8">
              <select v-model="form.moneda" class="form-control select-red" required>
                <option value="PEN">PEN</option>
                <option value="USD">USD</option>
              </select>
            </div>
          </div>
          <div class="row mb-4">
            <label class="col-sm-4 col-form-label">Importe total:</label>
            <div class="col-sm-8">
              <input
                v-model="form.importe"
                type="number"
                class="form-control input-red"
                required
                step="any"
                min="0"
                @keypress="bloquearNegativos"
                @blur="validarImporte"
              />
              <small v-if="errores.importe" class="text-danger">{{ errores.importe }}</small>
            </div>
          </div>
          <button type="submit" class="btn btn-danger w-100">Generar PDF</button>
        </form>
      </div>
      <div class="card-footer text-muted text-center ">
        ¿Dudas? Escríbenos a <a href="mailto:soporte@galaxiasoftware.com">soporte@galaxiasoftware.com</a>
      </div>
    </div>
  </div>
</template>

<script setup>
import { reactive, ref } from 'vue'

const form = reactive({
  tipoComprobante: '01',
  serie: '',
  numero: '',
  fecha: '',
  ruc: '',
  moneda: 'MN',
  importe: ''
})

const errores = reactive({
  numero: '',
  ruc: '',
  importe: ''
})

const validarNumero = () => {
  if (form.numero === '' || form.numero <= 0) {
    errores.numero = 'El número del comprobante debe ser mayor a cero.'
  } else {
    errores.numero = ''
  }
}

const validarRuc = () => {
  const rucRegex = /^(?:\d{8}|\d{11})$/ // 8 o 11 dígitos
  if (!rucRegex.test(form.ruc)) {
    errores.ruc = 'Debe ingresar un DNI (8 dígitos) o un RUC válido (11 dígitos).'
  } else {
    errores.ruc = ''
  }
}

const validarImporte = () => {
  if (!form.importe || form.importe <= 0) {
    errores.importe = 'El importe total debe ser mayor a 0.'
  } else {
    errores.importe = ''
    // Formatear a 2 decimales
    form.importe = parseFloat(form.importe).toFixed(2)
  }
}

const bloquearNegativos = (event) => {
  if (event.key === '-' || event.key === 'e') {
    event.preventDefault()
  }
}

const obtenerPrefijo = () => {
  switch (form.tipoComprobante) {
    case '01': return 'F'
    case '03': return 'B'
    case '07': return 'NC'
    case '08': return 'ND'
    default: return ''
  }
}

const ajustarSerie = () => {
  const numero = form.serie.replace(/^(F|B|NC|ND)/i, '').trim()
  const prefijo = obtenerPrefijo()
  form.serie = prefijo + numero
}

const fechaInput = ref(null)

const abrirCalendario = () => {
  if (fechaInput.value) {
    fechaInput.value.showPicker?.() // método nativo para abrir calendario
    fechaInput.value.focus()
  }
}

const generarPDF = async () => {
  validarNumero()
  validarRuc()
  validarImporte()

  if (errores.numero || errores.ruc || errores.importe) {
    alert('Corrige los errores antes de continuar.')
    return
  }

  const url = `http://163.123.180.94:3000/api/comprobantes/${form.tipoComprobante}/${form.serie}/${form.numero}?fecha=${form.fecha}&ruc=${form.ruc}&moneda=${form.moneda}&importe=${form.importe}`

  try {
    const res = await fetch(url)

    if (!res.ok) throw new Error('Error al generar comprobante')

    const blob = await res.blob()
    const pdfUrl = window.URL.createObjectURL(blob)
    window.open(pdfUrl, '_blank')
  } catch (error) {
    alert('Ocurrió un error: ' + error.message)
  }
}

</script>

<style scoped>
/* Chrome, Safari, Edge, Opera */
input[type="number"]::-webkit-outer-spin-button,
input[type="number"]::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

/* Quitar flechas en inputs tipo number */
input[type="number"] {
  appearance: textfield;        /* Propiedad estándar */
  -moz-appearance: textfield;   /* Firefox */
  -webkit-appearance: none;     /* Chrome, Safari, Edge */
}

input[type="number"]::-webkit-outer-spin-button,
input[type="number"]::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

/* Asegurar que el botón del calendario sea visible */

/* Ocultar ícono nativo del input date */
input[type="date"]::-webkit-calendar-picker-indicator {
  opacity: 0;
  display: none;
  pointer-events: none;
}

.input_span span{
  cursor: pointer;
}

.input-red:focus {
  border-color: red !important;
  box-shadow: 0 0 0 0.2rem rgba(255, 0, 0, 0.25);
}

.select-red {
  border: 1px solid red !important;
}

.select-red:focus {
  border-color: red !important;
  box-shadow: 0 0 0 0.2rem rgba(255, 0, 0, 0.25) !important;
}

.select-red:hover {
  border-color: red !important;
}

.card-footer a{
  color: red;
}

</style>
