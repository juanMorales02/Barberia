<script setup>
import { ref } from 'vue'
import { useLocalStorage } from '@vueuse/core'
import Swal from 'sweetalert2'
import 'sweetalert2/dist/sweetalert2.min.css'

/* ---------- catálogos fijos ---------- */

const barberos = ['Don Ramiro', 'Kevin', 'Andrés']

const catalogoServicios = [
  { nombre: 'Corte Clásico', precio: 12000 },
  { nombre: 'Corte Moderno de Precisión', precio: 18000 },
  { nombre: 'Perfilado de Barba & Ritual', precio: 15000 },
  { nombre: 'Corte Clásico & Arreglo de Barba', precio: 25000 },
  { nombre: 'Cejas', precio: 6000 },
  { nombre: 'Tinte', precio: 30000 }
]

const metodosPago = ['Efectivo', 'Transferencia', 'Tarjeta / Débito']

const horaMinima = '09:00'
const horaMaxima = '20:00'

/* ---------- estilos de swal reutilizables ---------- */

const swalBase = {
  background: '#1a1a1a',
  color: '#ffffff',
  confirmButtonColor: '#3b82f6',
  cancelButtonColor: '#3c3c3c',
  customClass: {
    popup: 'swal-heritage'
  }
}

function alertaError(mensaje) {
  Swal.fire({ ...swalBase, icon: 'error', title: 'Falta información', text: mensaje })
}

function alertaExito(mensaje) {
  Swal.fire({
    ...swalBase,
    toast: true,
    position: 'top-end',
    icon: 'success',
    title: mensaje,
    showConfirmButton: false,
    timer: 2200,
    timerProgressBar: true
  })
}

/* ---------- datos persistentes ---------- */

const servicios = useLocalStorage('servicios-barberia-don-ramiro', [])

/* ---------- estado del modal ---------- */

const modalAbierto = ref(false)
const modoEdicion = ref(false)
const idEdicion = ref(null)

const fCliente = ref('')
const fServicio = ref('')
const fBarbero = ref('')
const fFecha = ref('')
const fHora = ref('')
const fPrecio = ref(0)
const fMetodoPago = ref('')
const fEstadoPago = ref('Pendiente')
const fMontoAbonado = ref('')

/* ---------- utilidades ---------- */

function formatearNumero(valor) {
  return Number(valor || 0).toLocaleString('es-CL')
}

function obtenerFechaHoy() {
  const hoy = new Date()
  const anio = hoy.getFullYear()
  const mes = String(hoy.getMonth() + 1).padStart(2, '0')
  const dia = String(hoy.getDate()).padStart(2, '0')
  return `${anio}-${mes}-${dia}`
}

function obtenerHoraActual() {
  const hoy = new Date()
  const horas = String(hoy.getHours()).padStart(2, '0')
  const minutos = String(hoy.getMinutes()).padStart(2, '0')
  return `${horas}:${minutos}`
}

function formatearFecha(fechaISO) {
  if (!fechaISO) return ''
  const meses = ['Ene', 'Feb', 'Mar', 'Abr', 'May', 'Jun', 'Jul', 'Ago', 'Sep', 'Oct', 'Nov', 'Dic']
  const partes = fechaISO.split('-')
  const anio = partes[0]
  const mes = meses[Number(partes[1]) - 1]
  const dia = Number(partes[2])
  return `${dia} ${mes}, ${anio}`
}

function textoCalificacion(numero) {
  if (numero === 5) return 'Excelente'
  if (numero === 4) return 'Muy bueno'
  if (numero === 3) return 'Bueno'
  if (numero === 2) return 'Regular'
  return 'Malo'
}

function calcularSaldo() {
  return Number(fPrecio.value || 0) - Number(fMontoAbonado.value || 0)
}

function abrirSelector(evento) {
  if (evento.target.showPicker) {
    evento.target.showPicker()
  }
}

function actualizarPrecioServicio() {
  fPrecio.value = 0
  for (let i = 0; i < catalogoServicios.length; i++) {
    if (catalogoServicios[i].nombre === fServicio.value) {
      fPrecio.value = catalogoServicios[i].precio
    }
  }
}

/* ---------- modal: abrir / cerrar ---------- */

function limpiarFormulario() {
  fCliente.value = ''
  fServicio.value = ''
  fBarbero.value = ''
  fFecha.value = ''
  fHora.value = ''
  fPrecio.value = 0
  fMetodoPago.value = ''
  fEstadoPago.value = 'Pendiente'
  fMontoAbonado.value = ''
}

function abrirModalNuevo() {
  modoEdicion.value = false
  idEdicion.value = null
  limpiarFormulario()
  modalAbierto.value = true
}

function abrirModalEditar(servicio) {
  modoEdicion.value = true
  idEdicion.value = servicio.id
  fCliente.value = servicio.cliente
  fServicio.value = servicio.servicio
  fBarbero.value = servicio.barbero
  fFecha.value = servicio.fecha
  fHora.value = servicio.hora
  fPrecio.value = servicio.precio
  fMetodoPago.value = servicio.metodoPago
  fEstadoPago.value = servicio.estadoPago
  fMontoAbonado.value = servicio.montoAbonado || ''
  modalAbierto.value = true
}

function cerrarModal() {
  modalAbierto.value = false
}

/* ---------- guardar ---------- */

function guardarServicio() {
  if (fCliente.value.trim() === '') {
    alertaError('Escribe el nombre del cliente.')
    return
  }
  if (fServicio.value === '') {
    alertaError('Selecciona el servicio a realizar.')
    return
  }
  if (fBarbero.value === '') {
    alertaError('Selecciona quién atendió.')
    return
  }
  if (fFecha.value === '') {
    alertaError('Selecciona la fecha de la cita.')
    return
  }
  if (fFecha.value < obtenerFechaHoy()) {
    alertaError('La fecha de la cita no puede ser anterior al día de hoy.')
    return
  }
  if (fHora.value === '') {
    alertaError('Selecciona la hora programada.')
    return
  }
  if (fHora.value < horaMinima || fHora.value > horaMaxima) {
    alertaError(`El horario de atención es de ${horaMinima} a ${horaMaxima} hrs.`)
    return
  }
  if (fFecha.value === obtenerFechaHoy() && fHora.value < obtenerHoraActual()) {
    alertaError('Esa hora ya pasó, elige una hora posterior a la actual.')
    return
  }
  if (fMetodoPago.value === '') {
    alertaError('Selecciona el método de pago.')
    return
  }
  if (fEstadoPago.value === 'Abonado') {
    if (fMontoAbonado.value === '' || Number(fMontoAbonado.value) <= 0) {
      alertaError('Ingresa el monto abonado.')
      return
    }
    if (Number(fMontoAbonado.value) >= Number(fPrecio.value)) {
      alertaError('El monto abonado no puede ser mayor o igual al arancel. Marca el estado como Pagado.')
      return
    }
  }

  const montoAbonadoFinal = fEstadoPago.value === 'Abonado' ? Number(fMontoAbonado.value) : 0

  if (modoEdicion.value) {
    for (let i = 0; i < servicios.value.length; i++) {
      if (servicios.value[i].id === idEdicion.value) {
        servicios.value[i].cliente = fCliente.value.trim()
        servicios.value[i].servicio = fServicio.value
        servicios.value[i].barbero = fBarbero.value
        servicios.value[i].fecha = fFecha.value
        servicios.value[i].hora = fHora.value
        servicios.value[i].precio = Number(fPrecio.value)
        servicios.value[i].metodoPago = fMetodoPago.value
        servicios.value[i].estadoPago = fEstadoPago.value
        servicios.value[i].montoAbonado = montoAbonadoFinal
      }
    }
    alertaExito('Registro actualizado correctamente.')
  } else {
    servicios.value.push({
      id: Date.now(),
      cliente: fCliente.value.trim(),
      servicio: fServicio.value,
      barbero: fBarbero.value,
      fecha: fFecha.value,
      hora: fHora.value,
      precio: Number(fPrecio.value),
      metodoPago: fMetodoPago.value,
      estadoPago: fEstadoPago.value,
      montoAbonado: montoAbonadoFinal,
      etapa: 'pendiente',
      calificacion: 0,
      observaciones: ''
    })
    alertaExito('Servicio agendado correctamente.')
  }

  cerrarModal()
}

/* ---------- eliminar (con confirmación SweetAlert2) ---------- */

function pedirConfirmacionEliminar(servicio) {
  Swal.fire({
    ...swalBase,
    icon: 'warning',
    title: '¿Cancelar y eliminar la cita?',
    text: `Se eliminará el registro de ${servicio.cliente}. Esta acción no se puede deshacer.`,
    showCancelButton: true,
    confirmButtonText: 'Sí, eliminar',
    cancelButtonText: 'Volver',
    confirmButtonColor: '#f87171'
  }).then((resultado) => {
    if (resultado.isConfirmed) {
      eliminarServicio(servicio.id)
      alertaExito('Cita eliminada.')
    }
  })
}

function eliminarServicio(id) {
  const nuevaLista = []
  for (let i = 0; i < servicios.value.length; i++) {
    if (servicios.value[i].id !== id) {
      nuevaLista.push(servicios.value[i])
    }
  }
  servicios.value = nuevaLista
}

/* ---------- etapas de la tarjeta ---------- */

function marcarCompletado(servicio) {
  servicio.etapa = 'calificando'
  servicio.calificacion = 5
}

function elegirEstrella(servicio, numero) {
  servicio.calificacion = numero
}

function guardarCalificacion(servicio) {
  servicio.etapa = 'completado'
  alertaExito('Calificación guardada.')
}

function volverSinCalificar(servicio) {
  servicio.etapa = 'pendiente'
}
</script>

<template>
  <div>
    <div class="encabezado">
      <div class="encabezado-texto">
        <p class="eyebrow">● Cuaderno operativo &amp; control de cabina · Edición Don Ramiro</p>
        <h1>Registro de Atención &amp; Citas</h1>
        <p class="subtitulo">
          Supervisión técnica de intervenciones programadas, flujos arancelarios de caja, estados de liquidación y
          rúbrica de satisfacción de caballeros.
        </p>
      </div>
      <button class="boton-nuevo" @click="abrirModalNuevo">
        <span class="material-symbols-outlined icono">add_circle</span> + Nuevo servicio
      </button>
    </div>

    <p v-if="servicios.length === 0" class="sin-resultados">Todavía no hay servicios registrados.</p>

    <div class="lista-tarjetas">
      <div v-for="servicio in servicios" :key="servicio.id" class="tarjeta-servicio">
        <div class="tarjeta-encabezado">
          <h3>{{ servicio.cliente }}</h3>
          <span
            class="estado"
            :class="{
              pendiente: servicio.estadoPago === 'Pendiente',
              abonado: servicio.estadoPago === 'Abonado',
              pagado: servicio.estadoPago === 'Pagado'
            }"
            >● {{ servicio.estadoPago }}</span
          >
        </div>

        <div class="divisor-tarjeta"></div>

        <div class="seccion-tarjeta datos-tarjeta">
          <div>
            <p class="etiqueta-dato">Servicio</p>
            <p class="valor-dato">{{ servicio.servicio }}</p>
          </div>
          <div>
            <p class="etiqueta-dato">Especialista</p>
            <p class="valor-dato">{{ servicio.barbero }}</p>
          </div>
          <div>
            <p class="etiqueta-dato">Fecha y hora</p>
            <p class="valor-dato">{{ formatearFecha(servicio.fecha) }} · {{ servicio.hora }} hrs</p>
          </div>
        </div>

        <div class="divisor-tarjeta"></div>

        <div v-if="servicio.estadoPago === 'Abonado'" class="seccion-tarjeta">
          <div class="datos-tarjeta">
            <div>
              <p class="etiqueta-dato">Arancel total</p>
              <p class="precio-dato">${{ formatearNumero(servicio.precio) }} <span>CLP</span></p>
            </div>
            <div class="alineado-derecha">
              <p class="etiqueta-dato">Saldo restante</p>
              <p class="precio-dato dorado">${{ formatearNumero(servicio.precio - servicio.montoAbonado) }} <span>CLP</span></p>
            </div>
          </div>
          <div class="datos-tarjeta datos-tarjeta-sin-margen">
            <div>
              <p class="etiqueta-dato">Abonado</p>
              <p class="valor-dato azul">${{ formatearNumero(servicio.montoAbonado) }} CLP</p>
            </div>
          </div>
        </div>

        <div v-else class="seccion-tarjeta datos-tarjeta">
          <div>
            <p class="etiqueta-dato" v-if="servicio.etapa === 'completado'">Total liquidado</p>
            <p class="etiqueta-dato" v-else>Arancel fijado</p>
            <p class="precio-dato">${{ formatearNumero(servicio.precio) }} <span>CLP</span></p>
          </div>
          <div class="alineado-derecha">
            <p class="etiqueta-dato">Método de pago</p>
            <p class="chip-metodo">● {{ servicio.metodoPago }}</p>
          </div>
        </div>

        <div class="divisor-tarjeta"></div>

        <div class="seccion-tarjeta" v-if="servicio.etapa === 'pendiente'">
          <div class="aviso-evaluacion">
            <p class="titulo-aviso"><span class="material-symbols-outlined icono">info</span> Evaluación post-servicio</p>
            <p>La calificación con estrellas y las anotaciones técnicas de corte se habilitan al completar la cita.</p>
            <button class="boton-completar" @click="marcarCompletado(servicio)">
              <span class="material-symbols-outlined icono">check</span> Marcar como completado &amp; calificar
            </button>
          </div>
          <div class="fila-acciones-tarjeta">
            <button class="enlace-reprogramar" @click="abrirModalEditar(servicio)">Reprogramar</button>
            <button class="boton-cancelar-cita" @click="pedirConfirmacionEliminar(servicio)">Cancelar cita</button>
          </div>
        </div>

        <div class="seccion-tarjeta" v-else-if="servicio.etapa === 'calificando'">
          <div class="bloque-calificacion">
            <div class="fila-calificacion">
              <p class="titulo-aviso"><span class="material-symbols-outlined icono">star</span> Calificación del servicio</p>
              <div class="selector-estrellas">
                <button
                  type="button"
                  v-for="n in 5"
                  :key="n"
                  :class="{ activa: n <= servicio.calificacion }"
                  @click="elegirEstrella(servicio, n)"
                >★</button>
                <span class="texto-estrellas">{{ servicio.calificacion }}.0 · {{ textoCalificacion(servicio.calificacion) }}</span>
              </div>
            </div>
            <label class="etiqueta-dato">Observaciones técnicas de cabina</label>
            <textarea v-model="servicio.observaciones" placeholder="Ej: texturizado con navaja, tratamiento capilar, etc."></textarea>
            <div class="fila-acciones-tarjeta">
              <button class="boton-completar" @click="guardarCalificacion(servicio)">
                <span class="material-symbols-outlined icono">check</span> Aceptar y guardar calificación
              </button>
              <button class="boton-volver" @click="volverSinCalificar(servicio)">Cancelar y volver</button>
            </div>
          </div>
        </div>

        <div class="seccion-tarjeta" v-else>
          <div class="bloque-calificacion bloque-calificacion-lectura">
            <div class="fila-calificacion">
              <p class="titulo-aviso-simple">Calificación de servicio</p>
              <div class="estrellas-fijas" :class="{ baja: servicio.calificacion <= 2 }">
                <span v-for="n in 5" :key="n" :class="{ llena: n <= servicio.calificacion }">★</span>
                <span class="texto-estrellas">{{ servicio.calificacion }}.0 / 5.0</span>
              </div>
            </div>
            <p v-if="servicio.calificacion <= 2" class="texto-calificacion-baja">Cliente insatisfecho, revisar con el barbero</p>
            <p class="etiqueta-dato-dorado">Observaciones técnicas</p>
            <p class="observaciones">"{{ servicio.observaciones || 'Sin observaciones registradas.' }}"</p>
          </div>
          <div class="divisor-tarjeta"></div>
          <div class="fila-acciones-tarjeta">
            <button class="boton-modificar" @click="abrirModalEditar(servicio)">Modificar registro</button>
            <button class="enlace-eliminar" @click="pedirConfirmacionEliminar(servicio)">Eliminar</button>
          </div>
        </div>
      </div>
    </div>

    <div v-if="modalAbierto" class="fondo-modal">
      <div class="caja-modal">
        <button class="boton-cerrar" @click="cerrarModal">✕</button>
        <p class="eyebrow-modal">● Don Ramiro · Registro</p>
        <h2 v-if="modoEdicion">Modificar servicio</h2>
        <h2 v-else>Agendar nuevo servicio</h2>
        <p class="descripcion-modal">Ingreso de cita inicial. Calificación y notas técnicas se habilitan al finalizar el servicio.</p>

        <form @submit.prevent="guardarServicio">
          <div class="campo-formulario">
            <label>Nombre del cliente</label>
            <input type="text" v-model="fCliente" placeholder="Ej: Carlos Pérez">
          </div>

          <div class="fila-formulario">
            <div class="campo-formulario">
              <label>Servicio a realizar</label>
              <select v-model="fServicio" @change="actualizarPrecioServicio">
                <option value="">Selecciona...</option>
                <option v-for="s in catalogoServicios" :key="s.nombre" :value="s.nombre">{{ s.nombre }}</option>
              </select>
            </div>
            <div class="campo-formulario">
              <label>Especialista / Barbero</label>
              <select v-model="fBarbero">
                <option value="">Selecciona...</option>
                <option v-for="barbero in barberos" :key="barbero" :value="barbero">{{ barbero }}</option>
              </select>
            </div>
          </div>

          <div class="fila-formulario">
            <div class="campo-formulario">
              <label>Fecha de cita</label>
              <input
                type="date"
                v-model="fFecha"
                :min="obtenerFechaHoy()"
                @click="abrirSelector"
              >
            </div>
            <div class="campo-formulario">
              <label>Hora programada</label>
              <input
                type="time"
                v-model="fHora"
                :min="horaMinima"
                :max="horaMaxima"
                @click="abrirSelector"
              >
            </div>
          </div>

          <div class="fila-formulario">
            <div class="campo-formulario">
              <label>Arancel fijado (CLP)</label>
              <input type="number" v-model="fPrecio" readonly placeholder="Selecciona un servicio">
            </div>
            <div class="campo-formulario">
              <label>Método de pago acordado</label>
              <select v-model="fMetodoPago">
                <option value="">Selecciona...</option>
                <option v-for="metodo in metodosPago" :key="metodo" :value="metodo">{{ metodo }}</option>
              </select>
            </div>
          </div>

          <div class="campo-formulario">
            <label>Estado de pago</label>
            <select v-model="fEstadoPago">
              <option value="Pendiente">Pendiente</option>
              <option value="Abonado">Abonado (Parcial)</option>
              <option value="Pagado">Pagado (Total)</option>
            </select>
          </div>

          <div class="campo-formulario" v-if="fEstadoPago === 'Abonado'">
            <label>Monto abonado (CLP)</label>
            <input type="number" v-model="fMontoAbonado" placeholder="Ej: 50000">
          </div>

          <div class="aviso-saldo" v-if="fEstadoPago === 'Abonado'">
            ● Saldo pendiente de cobro: <span>${{ formatearNumero(calcularSaldo()) }} CLP</span>
          </div>

          <div class="botones-modal">
            <button type="button" class="boton-cancelar" @click="cerrarModal">Cancelar</button>
            <button type="submit" class="boton-guardar" v-if="modoEdicion">Guardar cambios</button>
            <button type="submit" class="boton-guardar" v-else>✓ Confirmar y agendar</button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>
