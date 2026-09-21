<script setup>
import { ref, computed, watch } from 'vue'
import { useLocalStorage } from '@vueuse/core'
import Swal from 'sweetalert2'
import 'sweetalert2/dist/sweetalert2.min.css'

/* ---------- catálogos fijos ---------- */

const barberos = ['Don Ramiro', 'Kevin', 'Andrés']

const metodosPago = ['Efectivo', 'Transferencia', 'Tarjeta / Débito']

const horaMinima = '09:00'
const horaMaxima = '20:00'

/* ---------- catálogo de servicios (editable por don Ramiro) ---------- */

const catalogoServiciosDefault = [
  { id: 1, nombre: 'Corte Afro', tipo: 'principal', precio: 22000 },
  { id: 2, nombre: 'Corte Curly Shag', tipo: 'principal', precio: 26000 },
  { id: 3, nombre: 'Corte Crew Cut', tipo: 'principal', precio: 18000 },
  { id: 4, nombre: 'Corte Ivy League', tipo: 'principal', precio: 24000 },
  { id: 5, nombre: 'Corte Quiff', tipo: 'principal', precio: 28000 },
  { id: 6, nombre: 'Corte French Crop', tipo: 'principal', precio: 27000 },
  { id: 7, nombre: 'Corte Caesar', tipo: 'principal', precio: 20000 },
  { id: 8, nombre: 'Corte Slick Back', tipo: 'principal', precio: 30000 },
  { id: 9, nombre: 'Corte Flat Top', tipo: 'principal', precio: 23000 },
  { id: 10, nombre: 'Corte Mohawk', tipo: 'principal', precio: 32000 },
  { id: 11, nombre: 'Arreglo de Barba', tipo: 'adicional', precio: 12000 },
  { id: 12, nombre: 'Diseño de Cejas', tipo: 'adicional', precio: 8000 },
  { id: 13, nombre: 'Coloración', tipo: 'adicional', precio: 20000 },
  { id: 14, nombre: 'Hidratación Capilar', tipo: 'adicional', precio: 15000 },
  { id: 15, nombre: 'Limpieza Facial', tipo: 'adicional', precio: 14000 }
]

const catalogoServicios = useLocalStorage('catalogo-servicios-don-ramiro', catalogoServiciosDefault)

/* tipo: 'principal' (cortes) o 'adicional' (barba, cejas, etc.).
   Si el catálogo guardado es anterior y no trae tipo, se infiere por el nombre. */
for (let i = 0; i < catalogoServicios.value.length; i++) {
  const item = catalogoServicios.value[i]
  if (!item.tipo) item.tipo = item.nombre.toLowerCase().startsWith('corte') ? 'principal' : 'adicional'
}

/* ---------- comisiones por barbero (%) ---------- */

const comisiones = useLocalStorage('comisiones-barberia-don-ramiro', {
  'Don Ramiro': 50,
  Kevin: 40,
  Andrés: 40
})

/* ---------- estilos de swal reutilizables ---------- */

const swalBase = {
  background: '#ffffff',
  color: '#16181d',
  confirmButtonColor: '#f4923a',
  cancelButtonColor: '#6b7280',
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

let avisoCuotaMostrado = false
function alCuotaLlena(error) {
  console.error(error)
  if (avisoCuotaMostrado) return
  avisoCuotaMostrado = true
  Swal.fire({
    ...swalBase,
    icon: 'error',
    title: 'Almacenamiento lleno',
    text: 'No se pudieron guardar los últimos cambios. Quita fotos de algunos servicios o cierra caja y elimina registros antiguos.'
  }).then(() => {
    avisoCuotaMostrado = false
  })
}

const servicios = useLocalStorage('servicios-barberia-don-ramiro', [], { onError: alCuotaLlena })

/* Solo existen tres estados de pago: Pendiente, Abonado y Pagado.
   Si hay registros viejos guardados como "Fiado", se convierten a Pendiente. */
for (let i = 0; i < servicios.value.length; i++) {
  if (servicios.value[i].estadoPago === 'Fiado') servicios.value[i].estadoPago = 'Pendiente'
}

/* ---------- estado del modal de servicio ---------- */

const modalAbierto = ref(false)
const modoEdicion = ref(false)
const idEdicion = ref(null)

const fCliente = ref('')
const fServicio = ref('')
const fBarbero = ref('')
const fFecha = ref('')
const fHora = ref('')
const fPrecio = ref(0)
const fPropina = ref('')
const fMetodoPago = ref('')
const fEstadoPago = ref('Pendiente')
const fMontoAbonado = ref('')
const fServiciosExtra = ref([])
const fFotoAntes = ref('')
const fFotoDespues = ref('')

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

/* Un servicio cuenta para "el día" (estadística, comisiones y cierre de caja) si no está archivado y
   su fecha es hoy o anterior (días sin cerrar), o si ya fue atendido y calificado. */
function esDelDia(servicio) {
  if (servicio.archivado) return false
  return servicio.fecha <= obtenerFechaHoy() || servicio.etapa === 'completado'
}

/* turno del día según la hora del servicio */
function obtenerTurno(hora) {
  const h = Number(String(hora).split(':')[0])
  if (h < 12) return 'Mañana'
  if (h < 19) return 'Tarde'
  return 'Noche'
}

/* ---------- campos de dinero con formato en vivo ---------- */

function crearDisplayDinero(refValor) {
  return computed({
    get() {
      return refValor.value === '' || refValor.value === 0 ? '' : formatearNumero(refValor.value)
    },
    set(valor) {
      const soloDigitos = String(valor).replace(/\D/g, '')
      refValor.value = soloDigitos === '' ? '' : Number(soloDigitos)
    }
  })
}

const fMontoAbonadoDisplay = crearDisplayDinero(fMontoAbonado)
const fPropinaDisplay = crearDisplayDinero(fPropina)
const fPrecioDisplay = crearDisplayDinero(fPrecio)

function abrirSelector(evento) {
  if (evento.target.showPicker) {
    evento.target.showPicker()
  }
}

/* ---------- arancel: servicio principal + servicios extra seleccionados ---------- */

function precioDelServicio(nombre) {
  for (let i = 0; i < catalogoServicios.value.length; i++) {
    if (catalogoServicios.value[i].nombre === nombre) return catalogoServicios.value[i].precio
  }
  return 0
}

/* El servicio principal solo ofrece los de tipo "principal" (cortes).
   Si al editar un registro antiguo su servicio ya no está en el catálogo, se conserva como opción. */
const serviciosPrincipales = computed(() => {
  const lista = []
  let incluyeActual = false
  for (let i = 0; i < catalogoServicios.value.length; i++) {
    const item = catalogoServicios.value[i]
    if (item.tipo === 'principal') {
      lista.push(item)
      if (item.nombre === fServicio.value) incluyeActual = true
    }
  }
  if (fServicio.value !== '' && !incluyeActual) {
    lista.push({ id: 'actual', nombre: fServicio.value, tipo: 'principal', precio: 0 })
  }
  return lista
})

/* Los adicionales solo ofrecen los de tipo "adicional", y nunca el que ya es el principal. */
const serviciosExtraDisponibles = computed(() => {
  const lista = []
  for (let i = 0; i < catalogoServicios.value.length; i++) {
    const item = catalogoServicios.value[i]
    if (item.tipo === 'adicional' && item.nombre !== fServicio.value) lista.push(item)
  }
  return lista
})

function recalcularPrecio() {
  let total = precioDelServicio(fServicio.value)
  for (let i = 0; i < fServiciosExtra.value.length; i++) {
    total += precioDelServicio(fServiciosExtra.value[i])
  }
  fPrecio.value = total
}

function alCambiarServicioPrincipal() {
  const nuevaLista = []
  for (let i = 0; i < fServiciosExtra.value.length; i++) {
    if (fServiciosExtra.value[i] !== fServicio.value) nuevaLista.push(fServiciosExtra.value[i])
  }
  fServiciosExtra.value = nuevaLista
  recalcularPrecio()
}

/* ---------- fidelidad: aviso de cliente frecuente ---------- */

const alertaFidelidadMostrada = ref(false)

function contarServiciosCliente(nombre) {
  const nombreNormalizado = nombre.trim().toLowerCase()
  let contador = 0
  for (let i = 0; i < servicios.value.length; i++) {
    const s = servicios.value[i]
    if (s.cliente.toLowerCase() === nombreNormalizado && s.id !== idEdicion.value) contador++
  }
  return contador
}

const clienteEsFrecuente = computed(() => {
  if (fCliente.value.trim() === '') return false
  return contarServiciosCliente(fCliente.value) >= 5
})

watch(clienteEsFrecuente, (esFrecuente) => {
  if (esFrecuente && !alertaFidelidadMostrada.value) {
    alertaFidelidadMostrada.value = true
    Swal.fire({
      ...swalBase,
      icon: 'info',
      title: '¡Cliente frecuente!',
      text: '¡Cliente frecuente, aplica 10% de descuento!'
    })
  }
})

/* ---------- fotos antes / después (base64, comprimidas) ---------- */

function manejarFoto(evento, tipo) {
  const archivo = evento.target.files[0]
  if (!archivo) return

  if (!archivo.type.startsWith('image/')) {
    alertaError('Selecciona un archivo de imagen válido.')
    return
  }

  const lector = new FileReader()
  lector.onload = () => {
    const imagen = new Image()
    imagen.onload = () => {
      // Redimensionamos antes de guardar en base64 para no saturar el localStorage.
      const anchoMaximo = 480
      const escala = Math.min(1, anchoMaximo / imagen.width)
      const canvas = document.createElement('canvas')
      canvas.width = imagen.width * escala
      canvas.height = imagen.height * escala

      const contexto = canvas.getContext('2d')
      contexto.fillStyle = '#ffffff'
      contexto.fillRect(0, 0, canvas.width, canvas.height)
      contexto.drawImage(imagen, 0, 0, canvas.width, canvas.height)
      const base64Comprimido = canvas.toDataURL('image/jpeg', 0.7)

      if (tipo === 'antes') fFotoAntes.value = base64Comprimido
      else fFotoDespues.value = base64Comprimido
    }
    imagen.onerror = () => alertaError('No se pudo leer esa imagen, prueba con otra.')
    imagen.src = lector.result
  }
  lector.readAsDataURL(archivo)
  evento.target.value = ''
}

function quitarFoto(tipo) {
  if (tipo === 'antes') fFotoAntes.value = ''
  else fFotoDespues.value = ''
}

/* ---------- modal: abrir / cerrar ---------- */

function limpiarFormulario() {
  fCliente.value = ''
  fServicio.value = ''
  fBarbero.value = ''
  fFecha.value = ''
  fHora.value = ''
  fPrecio.value = 0
  fPropina.value = ''
  fMetodoPago.value = ''
  fEstadoPago.value = 'Pendiente'
  fMontoAbonado.value = ''
  fServiciosExtra.value = []
  fFotoAntes.value = ''
  fFotoDespues.value = ''
}

function abrirModalNuevo() {
  modoEdicion.value = false
  idEdicion.value = null
  alertaFidelidadMostrada.value = false
  limpiarFormulario()
  modalAbierto.value = true
}

function abrirModalEditar(servicio) {
  modoEdicion.value = true
  idEdicion.value = servicio.id
  alertaFidelidadMostrada.value = false
  fCliente.value = servicio.cliente
  fServicio.value = servicio.servicio
  fBarbero.value = servicio.barbero
  fFecha.value = servicio.fecha
  fHora.value = servicio.hora
  fPrecio.value = servicio.precio
  fPropina.value = servicio.propina || ''
  fMetodoPago.value = servicio.metodoPago
  fEstadoPago.value = servicio.estadoPago
  fMontoAbonado.value = servicio.montoAbonado || ''
  fServiciosExtra.value = servicio.serviciosExtra ? [...servicio.serviciosExtra] : []
  fFotoAntes.value = servicio.fotoAntes || ''
  fFotoDespues.value = servicio.fotoDespues || ''
  modalAbierto.value = true
}

function cerrarModal() {
  modalAbierto.value = false
}

/* ---------- guardar ---------- */

function guardarServicio() {
  let fechaHoraSinCambios = false
  if (modoEdicion.value) {
    for (let i = 0; i < servicios.value.length; i++) {
      const original = servicios.value[i]
      if (original.id === idEdicion.value && original.fecha === fFecha.value && original.hora === fHora.value) {
        fechaHoraSinCambios = true
      }
    }
  }

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
  if (!fechaHoraSinCambios && fFecha.value < obtenerFechaHoy()) {
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
  if (!fechaHoraSinCambios && fFecha.value === obtenerFechaHoy() && fHora.value < obtenerHoraActual()) {
    alertaError('Esa hora ya pasó, elige una hora posterior a la actual.')
    return
  }
  if (!fPrecio.value || Number(fPrecio.value) <= 0) {
    alertaError('Selecciona un servicio principal para calcular el precio total.')
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
  const propinaFinal = Number(fPropina.value || 0)

  if (modoEdicion.value) {
    for (let i = 0; i < servicios.value.length; i++) {
      if (servicios.value[i].id === idEdicion.value) {
        servicios.value[i].cliente = fCliente.value.trim()
        servicios.value[i].servicio = fServicio.value
        servicios.value[i].barbero = fBarbero.value
        servicios.value[i].fecha = fFecha.value
        servicios.value[i].hora = fHora.value
        servicios.value[i].precio = Number(fPrecio.value)
        servicios.value[i].propina = propinaFinal
        servicios.value[i].metodoPago = fMetodoPago.value
        servicios.value[i].estadoPago = fEstadoPago.value
        servicios.value[i].montoAbonado = montoAbonadoFinal
        servicios.value[i].serviciosExtra = [...fServiciosExtra.value]
        servicios.value[i].fotoAntes = fFotoAntes.value
        servicios.value[i].fotoDespues = fFotoDespues.value
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
      propina: propinaFinal,
      metodoPago: fMetodoPago.value,
      estadoPago: fEstadoPago.value,
      montoAbonado: montoAbonadoFinal,
      serviciosExtra: [...fServiciosExtra.value],
      fotoAntes: fFotoAntes.value,
      fotoDespues: fFotoDespues.value,
      archivado: false,
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

/* Un servicio solo se puede completar el día de la cita o después (nunca antes). */
function puedeCompletarse(servicio) {
  return servicio.fecha <= obtenerFechaHoy()
}

function marcarCompletado(servicio) {
  if (!puedeCompletarse(servicio)) {
    alertaError(`Este servicio es para el ${formatearFecha(servicio.fecha)}. Solo se puede completar ese día o después.`)
    return
  }
  servicio.etapa = 'calificando'
  servicio.calificacion = 5
  servicio.montoPago = ''
}

function actualizarMontoPago(servicio, evento) {
  const soloDigitos = evento.target.value.replace(/\D/g, '')
  servicio.montoPago = soloDigitos === '' ? '' : Number(soloDigitos)
  evento.target.value = servicio.montoPago === '' ? '' : formatearNumero(servicio.montoPago)
}

function registrarPagoFaltante(servicio) {
  const monto = Number(servicio.montoPago || 0)

  if (!monto || monto <= 0) {
    alertaError('Ingresa el monto que se va a pagar.')
    return
  }

  const abonadoPrevio = servicio.montoAbonado || 0
  const nuevoAbonado = abonadoPrevio + monto
  const saldoRestante = servicio.precio - nuevoAbonado

  if (saldoRestante > 0) {
    servicio.montoAbonado = nuevoAbonado
    servicio.estadoPago = 'Abonado'
    servicio.montoPago = ''
    Swal.fire({
      ...swalBase,
      icon: 'warning',
      title: 'Saldo no cubierto',
      text: `Aún queda un saldo de $${formatearNumero(saldoRestante)} CLP. Debes cancelarlo por completo para poder calificar el servicio.`
    })
    return
  }

  servicio.montoAbonado = servicio.precio
  servicio.estadoPago = 'Pagado'
  servicio.montoPago = ''
  alertaExito('Pago registrado. Cuenta saldada.')
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
  servicio.calificacion = 0
}

/* ---------- ordenar servicios ---------- */

const verArchivados = ref(false)
const campoOrden = ref('fecha')
const direccionOrden = ref('desc')

function ordenarPor(campo) {
  if (campoOrden.value === campo) {
    direccionOrden.value = direccionOrden.value === 'asc' ? 'desc' : 'asc'
  } else {
    campoOrden.value = campo
    direccionOrden.value = 'desc'
  }
}

const serviciosOrdenados = computed(() => {
  const activos = []
  for (let i = 0; i < servicios.value.length; i++) {
    if (Boolean(servicios.value[i].archivado) === verArchivados.value) activos.push(servicios.value[i])
  }

  activos.sort((a, b) => {
    let valorA = 0
    let valorB = 0

    if (campoOrden.value === 'precio') {
      valorA = a.precio + (a.propina || 0)
      valorB = b.precio + (b.propina || 0)
    } else if (campoOrden.value === 'calificacion') {
      valorA = a.calificacion || 0
      valorB = b.calificacion || 0
    } else {
      valorA = `${a.fecha} ${a.hora}`
      valorB = `${b.fecha} ${b.hora}`
    }

    if (valorA < valorB) return direccionOrden.value === 'asc' ? -1 : 1
    if (valorA > valorB) return direccionOrden.value === 'asc' ? 1 : -1
    return 0
  })

  return activos
})

const cantidadArchivados = computed(() => {
  let contador = 0
  for (let i = 0; i < servicios.value.length; i++) {
    if (servicios.value[i].archivado) contador++
  }
  return contador
})

/* ---------- turnos del día: grupos fijos Mañana / Tarde / Noche ---------- */

const nombresTurnos = ['Mañana', 'Tarde', 'Noche']

const gruposTurnos = computed(() => {
  const grupos = []
  for (let i = 0; i < nombresTurnos.length; i++) {
    const items = []
    for (let j = 0; j < serviciosOrdenados.value.length; j++) {
      if (obtenerTurno(serviciosOrdenados.value[j].hora) === nombresTurnos[i]) items.push(serviciosOrdenados.value[j])
    }
    grupos.push({ nombre: nombresTurnos[i], items })
  }
  return grupos
})

/* ---------- estadísticas básicas ---------- */

const estadisticas = computed(() => {
  let totalVendido = 0
  let numeroServicios = 0
  let sumaCalificaciones = 0
  let cantidadCalificados = 0
  const conteoPorBarberoHoy = {}

  for (let i = 0; i < servicios.value.length; i++) {
    const s = servicios.value[i]
    if (!esDelDia(s)) continue

    numeroServicios++
    totalVendido += s.precio + (s.propina || 0)

    if (s.etapa === 'completado' && s.calificacion > 0) {
      sumaCalificaciones += s.calificacion
      cantidadCalificados++
    }

    if (esDelDia(s)) {
      conteoPorBarberoHoy[s.barbero] = (conteoPorBarberoHoy[s.barbero] || 0) + 1
    }
  }

  let barberoTop = null
  let cortesBarberoTop = 0
  for (const nombre in conteoPorBarberoHoy) {
    if (conteoPorBarberoHoy[nombre] > cortesBarberoTop) {
      cortesBarberoTop = conteoPorBarberoHoy[nombre]
      barberoTop = nombre
    }
  }

  return {
    totalVendido,
    numeroServicios,
    promedioCalificacion: cantidadCalificados > 0 ? sumaCalificaciones / cantidadCalificados : 0,
    barberoTop,
    cortesBarberoTop
  }
})

/* ---------- historial por cliente ---------- */

const busquedaCliente = ref('')

const historialCliente = computed(() => {
  const nombre = busquedaCliente.value.trim().toLowerCase()
  if (nombre === '') return null

  let visitas = 0
  let totalGastado = 0

  for (let i = 0; i < servicios.value.length; i++) {
    const s = servicios.value[i]
    if (s.cliente.toLowerCase().includes(nombre)) {
      visitas++
      totalGastado += s.precio + (s.propina || 0)
    }
  }

  return { visitas, totalGastado }
})

/* ---------- recordatorio de deudas (estados Pendiente y Abonado) ---------- */

const deudasClientes = computed(() => {
  const mapa = {}

  for (let i = 0; i < servicios.value.length; i++) {
    const s = servicios.value[i]
    if (s.estadoPago === 'Pendiente' || s.estadoPago === 'Abonado') {
      const saldo = s.precio - (s.montoAbonado || 0)
      mapa[s.cliente] = (mapa[s.cliente] || 0) + saldo
    }
  }

  const lista = []
  for (const nombre in mapa) {
    lista.push({ cliente: nombre, total: mapa[nombre] })
  }
  return lista
})

function cobrarDeuda(deuda) {
  Swal.fire({
    ...swalBase,
    icon: 'question',
    title: `¿${deuda.cliente} pagó su deuda?`,
    text: `Se marcarán como pagados $${formatearNumero(deuda.total)} CLP.`,
    showCancelButton: true,
    confirmButtonText: 'Sí, marcar pagado',
    cancelButtonText: 'Volver'
  }).then((resultado) => {
    if (!resultado.isConfirmed) return
    for (let i = 0; i < servicios.value.length; i++) {
      const s = servicios.value[i]
      if ((s.estadoPago === 'Pendiente' || s.estadoPago === 'Abonado') && s.cliente === deuda.cliente) {
        s.estadoPago = 'Pagado'
        s.montoAbonado = s.precio
      }
    }
    alertaExito('Deuda saldada.')
  })
}

/* ---------- comisiones de hoy por barbero ---------- */

const comisionesHoy = computed(() => {
  const resultado = []

  for (let i = 0; i < barberos.length; i++) {
    const nombre = barberos[i]
    let totalFacturado = 0

    for (let j = 0; j < servicios.value.length; j++) {
      const s = servicios.value[j]
      if (s.barbero === nombre && esDelDia(s)) {
        totalFacturado += s.precio
      }
    }

    const porcentaje = comisiones.value[nombre] || 0
    resultado.push({
      nombre,
      totalFacturado,
      porcentaje,
      montoComision: (totalFacturado * porcentaje) / 100
    })
  }

  return resultado
})

/* ---------- cierre de caja diario ---------- */

const cierreCajaAbierto = ref(false)
const resumenCierre = ref(null)

function calcularResumenCaja() {
  let efectivo = 0
  let transferencia = 0
  let tarjeta = 0
  let pendientePorCobrar = 0
  const idsDelDia = []

  for (let i = 0; i < servicios.value.length; i++) {
    const s = servicios.value[i]
    if (!esDelDia(s)) continue

    // Solo cuenta como cobrado lo que realmente entró a caja.
    let cobrado = 0
    if (s.estadoPago === 'Pagado') cobrado = s.precio + (s.propina || 0)
    else if (s.estadoPago === 'Abonado') cobrado = s.montoAbonado || 0

    if (s.metodoPago === 'Efectivo') {
      efectivo += cobrado
    } else if (s.metodoPago === 'Transferencia') {
      transferencia += cobrado
    } else {
      tarjeta += cobrado
    }

    if (s.estadoPago !== 'Pagado') {
      pendientePorCobrar += s.precio - (s.montoAbonado || 0)
    }

    idsDelDia.push(s.id)
  }

  return { efectivo, transferencia, tarjeta, pendientePorCobrar, idsDelDia, comisiones: comisionesHoy.value.map((c) => ({ ...c })) }
}

function abrirCierreCaja() {
  const resumen = calcularResumenCaja()
  if (resumen.idsDelDia.length === 0) {
    alertaError('No hay servicios del día para cerrar caja.')
    return
  }
  resumenCierre.value = resumen
  cierreCajaAbierto.value = true
}

function cerrarVentanaCierre() {
  cierreCajaAbierto.value = false
}

function confirmarCierreCaja() {
  let sinCompletar = 0
  for (let i = 0; i < servicios.value.length; i++) {
    const s = servicios.value[i]
    if (resumenCierre.value.idsDelDia.includes(s.id) && s.etapa !== 'completado') sinCompletar++
  }
  const avisoSinCompletar = sinCompletar > 0
    ? ` Ojo: ${sinCompletar} servicio(s) de hoy todavía no están marcados como completados y también se archivarán.`
    : ''

  Swal.fire({
    ...swalBase,
    icon: 'question',
    title: '¿Archivar los servicios de hoy?',
    text: 'Dejarán de aparecer en la vista principal y quedan guardados en "Ver archivados".' + avisoSinCompletar,
    showCancelButton: true,
    confirmButtonText: 'Sí, archivar',
    cancelButtonText: 'Volver'
  }).then((resultado) => {
    if (!resultado.isConfirmed) return

    for (let i = 0; i < servicios.value.length; i++) {
      if (resumenCierre.value.idsDelDia.includes(servicios.value[i].id)) {
        servicios.value[i].archivado = true
      }
    }

    cierreCajaAbierto.value = false
    alertaExito('Caja cerrada. Los servicios de hoy fueron archivados.')
  })
}

/* ---------- catálogo de servicios: panel de gestión ---------- */

const panelCatalogoAbierto = ref(false)
const idEdicionServicioCatalogo = ref(null)
const fNombreServicioCatalogo = ref('')
const fPrecioServicioCatalogo = ref('')
const fTipoServicioCatalogo = ref('principal')

function abrirPanelCatalogo() {
  panelCatalogoAbierto.value = true
}

function limpiarFormularioCatalogo() {
  idEdicionServicioCatalogo.value = null
  fNombreServicioCatalogo.value = ''
  fPrecioServicioCatalogo.value = ''
  fTipoServicioCatalogo.value = 'principal'
}

function cancelarEdicionCatalogo() {
  limpiarFormularioCatalogo()
}

function cerrarPanelCatalogo() {
  panelCatalogoAbierto.value = false
  limpiarFormularioCatalogo()
}

function editarServicioCatalogo(servicio) {
  idEdicionServicioCatalogo.value = servicio.id
  fNombreServicioCatalogo.value = servicio.nombre
  fPrecioServicioCatalogo.value = servicio.precio
  fTipoServicioCatalogo.value = servicio.tipo
}

function guardarServicioCatalogo() {
  if (fNombreServicioCatalogo.value.trim() === '' || !fPrecioServicioCatalogo.value) {
    alertaError('Completa el nombre y el precio base sugerido.')
    return
  }

  const nombreNuevo = fNombreServicioCatalogo.value.trim().toLowerCase()
  for (let i = 0; i < catalogoServicios.value.length; i++) {
    const item = catalogoServicios.value[i]
    if (item.nombre.toLowerCase() === nombreNuevo && item.id !== idEdicionServicioCatalogo.value) {
      alertaError('Ya existe un servicio con ese nombre.')
      return
    }
  }

  if (idEdicionServicioCatalogo.value !== null) {
    for (let i = 0; i < catalogoServicios.value.length; i++) {
      if (catalogoServicios.value[i].id === idEdicionServicioCatalogo.value) {
        catalogoServicios.value[i].nombre = fNombreServicioCatalogo.value.trim()
        catalogoServicios.value[i].precio = Number(fPrecioServicioCatalogo.value)
        catalogoServicios.value[i].tipo = fTipoServicioCatalogo.value
      }
    }
  } else {
    catalogoServicios.value.push({
      id: Date.now(),
      nombre: fNombreServicioCatalogo.value.trim(),
      tipo: fTipoServicioCatalogo.value,
      precio: Number(fPrecioServicioCatalogo.value)
    })
  }

  limpiarFormularioCatalogo()
}

function eliminarServicioCatalogo(id) {
  const nuevaLista = []
  for (let i = 0; i < catalogoServicios.value.length; i++) {
    if (catalogoServicios.value[i].id !== id) nuevaLista.push(catalogoServicios.value[i])
  }
  catalogoServicios.value = nuevaLista
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
      <div class="grupo-botones-encabezado">
        <button class="boton-secundario" @click="abrirPanelCatalogo">
          <span class="material-symbols-outlined icono">content_cut</span> Catálogo
        </button>
        <button class="boton-secundario" @click="abrirCierreCaja">
          <span class="material-symbols-outlined icono">point_of_sale</span> Cerrar caja
        </button>
        <button class="boton-nuevo" @click="abrirModalNuevo">
          <span class="material-symbols-outlined icono">add_circle</span> + Nuevo servicio
        </button>
      </div>
    </div>

    <!-- ---------- estadísticas básicas ---------- -->
    <div class="grid-estadisticas">
      <div class="tarjeta-estadistica">
        <p class="etiqueta-dato">Total vendido</p>
        <p class="valor-estadistica">${{ formatearNumero(estadisticas.totalVendido) }}</p>
      </div>
      <div class="tarjeta-estadistica">
        <p class="etiqueta-dato">Servicios registrados</p>
        <p class="valor-estadistica">{{ estadisticas.numeroServicios }}</p>
      </div>
      <div class="tarjeta-estadistica">
        <p class="etiqueta-dato">Calificación promedio</p>
        <p class="valor-estadistica">
          {{ estadisticas.promedioCalificacion > 0 ? estadisticas.promedioCalificacion.toFixed(1) : '—' }}
          <span class="unidad-estadistica" v-if="estadisticas.promedioCalificacion > 0">★</span>
        </p>
      </div>
      <div class="tarjeta-estadistica">
        <p class="etiqueta-dato">Barbero del día</p>
        <p class="valor-estadistica valor-estadistica-texto" v-if="estadisticas.barberoTop">
          {{ estadisticas.barberoTop }}
          <span class="unidad-estadistica">{{ estadisticas.cortesBarberoTop }} cortes</span>
        </p>
        <p class="valor-estadistica valor-estadistica-texto" v-else>Sin datos hoy</p>
      </div>
    </div>

    <!-- ---------- recordatorio de deudas ---------- -->
    <div v-if="deudasClientes.length" class="panel-alerta panel-deudas">
      <p class="titulo-aviso"><span class="material-symbols-outlined icono">payments</span> Deudas por cobrar</p>
      <div class="lista-deudas">
        <div class="item-deuda" v-for="deuda in deudasClientes" :key="deuda.cliente">
          <span class="nombre-deuda">{{ deuda.cliente }}</span>
          <span class="monto-deuda">${{ formatearNumero(deuda.total) }} CLP</span>
          <button type="button" class="boton-cobrar" @click="cobrarDeuda(deuda)">Marcar pagado</button>
        </div>
      </div>
    </div>

    <!-- ---------- comisiones por barbero ---------- -->
    <div class="panel-comisiones">
      <p class="titulo-panel">Comisiones de hoy por barbero</p>
      <div class="grid-comisiones">
        <div class="tarjeta-comision" v-for="c in comisionesHoy" :key="c.nombre">
          <p class="nombre-comision">{{ c.nombre }}</p>
          <p class="etiqueta-dato">Facturado hoy</p>
          <p class="valor-dato">${{ formatearNumero(c.totalFacturado) }}</p>
          <div class="fila-comision-porcentaje">
            <label>Comisión %</label>
            <input type="number" min="0" max="100" v-model.number="comisiones[c.nombre]">
          </div>
          <p class="etiqueta-dato">Le corresponde</p>
          <p class="precio-dato dorado">${{ formatearNumero(c.montoComision) }}</p>
        </div>
      </div>
    </div>

    <!-- ---------- barra de herramientas: ordenar y buscar cliente ---------- -->
    <div class="barra-herramientas">
      <div class="grupo-orden">
        <span class="etiqueta-dato etiqueta-orden">Ordenar por</span>
        <button
          class="boton-orden"
          :class="{ activo: campoOrden === 'fecha' }"
          @click="ordenarPor('fecha')"
        >Fecha {{ campoOrden === 'fecha' ? (direccionOrden === 'asc' ? '↑' : '↓') : '' }}</button>
        <button
          class="boton-orden"
          :class="{ activo: campoOrden === 'precio' }"
          @click="ordenarPor('precio')"
        >Precio {{ campoOrden === 'precio' ? (direccionOrden === 'asc' ? '↑' : '↓') : '' }}</button>
        <button
          class="boton-orden"
          :class="{ activo: campoOrden === 'calificacion' }"
          @click="ordenarPor('calificacion')"
        >Calificación {{ campoOrden === 'calificacion' ? (direccionOrden === 'asc' ? '↑' : '↓') : '' }}</button>
      </div>

      <button
        class="boton-orden boton-archivados"
        :class="{ activo: verArchivados }"
        @click="verArchivados = !verArchivados"
      >
        <span class="material-symbols-outlined icono">inventory_2</span>
        {{ verArchivados ? 'Volver a la vista principal' : `Ver archivados (${cantidadArchivados})` }}
      </button>

      <div class="buscador-cliente">
        <input type="text" v-model="busquedaCliente" placeholder="Buscar historial de un cliente...">
        <div class="resultado-cliente" v-if="historialCliente">
          <span v-if="historialCliente.visitas > 0">
            Ha venido <strong>{{ historialCliente.visitas }}</strong> {{ historialCliente.visitas === 1 ? 'vez' : 'veces' }}
            y ha gastado <strong>${{ formatearNumero(historialCliente.totalGastado) }}</strong> CLP
          </span>
          <span v-else>Ese cliente no tiene servicios registrados.</span>
        </div>
      </div>
    </div>

    <p v-if="serviciosOrdenados.length === 0" class="sin-resultados">
      {{ verArchivados ? 'No hay servicios archivados.' : 'Todavía no hay servicios registrados.' }}
    </p>

    <template v-for="grupo in gruposTurnos" :key="grupo.nombre">
      <section v-if="grupo.items.length" class="grupo-turno">
        <div class="separador-turno">
          <span>{{ grupo.nombre }}</span>
          <span class="contador-turno">{{ grupo.items.length }} {{ grupo.items.length === 1 ? 'servicio' : 'servicios' }}</span>
        </div>

        <div class="lista-tarjetas">
        <div class="tarjeta-servicio" v-for="servicio in grupo.items" :key="servicio.id">
          <div class="tarjeta-encabezado">
            <div class="cliente-info">
              <div class="avatar">{{ servicio.cliente.charAt(0).toUpperCase() }}</div>
              <h3>{{ servicio.cliente }}</h3>
            </div>
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
              <p class="etiqueta-dato">Servicio principal</p>
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

          <div class="seccion-tarjeta" v-if="servicio.serviciosExtra && servicio.serviciosExtra.length">
            <p class="etiqueta-dato">Servicios extra</p>
            <div class="lista-extras">
              <span class="chip-extra" v-for="extra in servicio.serviciosExtra" :key="extra">● {{ extra }}</span>
            </div>
          </div>

          <div class="seccion-tarjeta" v-if="servicio.fotoAntes || servicio.fotoDespues">
            <p class="etiqueta-dato">Antes / Después</p>
            <div class="grid-fotos-tarjeta">
              <img v-if="servicio.fotoAntes" :src="servicio.fotoAntes" alt="Foto antes" class="miniatura-foto">
              <img v-if="servicio.fotoDespues" :src="servicio.fotoDespues" alt="Foto después" class="miniatura-foto">
            </div>
          </div>

          <div class="divisor-tarjeta"></div>

          <div v-if="servicio.estadoPago === 'Abonado'" class="seccion-tarjeta caja-financiera">
            <div class="datos-tarjeta">
              <div>
                <p class="etiqueta-dato">Total</p>
                <p class="precio-dato">
                  ${{ formatearNumero(servicio.precio) }}
                  <span class="propina-dato" v-if="servicio.propina">+ ${{ formatearNumero(servicio.propina) }} propina</span>
                  <span>CLP</span>
                </p>
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

          <div v-else class="seccion-tarjeta datos-tarjeta caja-financiera">
            <div>
              <p class="etiqueta-dato">Total</p>
              <p class="precio-dato">
                ${{ formatearNumero(servicio.precio) }}
                <span class="propina-dato" v-if="servicio.propina">+ ${{ formatearNumero(servicio.propina) }} propina</span>
                <span>CLP</span>
              </p>
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
              <button class="boton-completar" :disabled="!puedeCompletarse(servicio)" @click="marcarCompletado(servicio)">
                <span class="material-symbols-outlined icono">check</span> Marcar como completado &amp; calificar
              </button>
              <p class="nota-fecha-futura" v-if="!puedeCompletarse(servicio)">
                Disponible el {{ formatearFecha(servicio.fecha) }}. Todavía no se puede completar.
              </p>
            </div>
            <div class="fila-acciones-tarjeta">
              <button class="enlace-reprogramar" @click="abrirModalEditar(servicio)">Reprogramar</button>
              <button class="boton-cancelar-cita" @click="pedirConfirmacionEliminar(servicio)">Cancelar cita</button>
            </div>
          </div>

          <div class="seccion-tarjeta" v-else-if="servicio.etapa === 'calificando'">
            <div class="bloque-calificacion caja-financiera">
              <template v-if="servicio.estadoPago !== 'Pagado'">
                <p class="titulo-aviso"><span class="material-symbols-outlined icono">payments</span> Pago pendiente</p>
                <p>Debes cancelar el saldo faltante antes de poder calificar el servicio.</p>
                <div class="fila-calificacion">
                  <div>
                    <p class="etiqueta-dato">Saldo a pagar</p>
                    <p class="precio-dato dorado">${{ formatearNumero(servicio.precio - (servicio.montoAbonado || 0)) }} <span>CLP</span></p>
                  </div>
                </div>
                <label class="etiqueta-dato">Monto que va a pagar (CLP)</label>
                <input
                  type="text"
                  inputmode="numeric"
                  placeholder="0"
                  :value="servicio.montoPago === '' || servicio.montoPago == null ? '' : formatearNumero(servicio.montoPago)"
                  @input="actualizarMontoPago(servicio, $event)"
                >
                <div class="fila-acciones-tarjeta">
                  <button class="boton-completar" @click="registrarPagoFaltante(servicio)">
                    <span class="material-symbols-outlined icono">check</span> Registrar pago
                  </button>
                  <button class="boton-volver" @click="volverSinCalificar(servicio)">Cancelar y volver</button>
                </div>
              </template>

              <template v-else>
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
              </template>
            </div>
          </div>

          <div class="seccion-tarjeta" v-else>
            <div class="bloque-calificacion bloque-calificacion-lectura caja-financiera">
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
          </div>
        </div>
        </div>
      </section>
    </template>

    <!-- ---------- modal: nuevo / editar servicio ---------- -->
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

          <div v-if="clienteEsFrecuente" class="aviso-fidelidad">
            <span class="material-symbols-outlined icono">military_tech</span>
            ¡Cliente frecuente, aplica 10% de descuento!
          </div>

          <div class="fila-formulario">
            <div class="campo-formulario">
              <label>Servicio principal (corte)</label>
              <select v-model="fServicio" @change="alCambiarServicioPrincipal">
                <option value="">Seleccionar servicio...</option>
                <option v-for="s in serviciosPrincipales" :key="s.id" :value="s.nombre">{{ s.nombre }}</option>
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

          <div class="campo-formulario">
            <div class="encabezado-campo">
              <label>Servicios adicionales</label>
              <span class="badge-opcional">Opcional</span>
            </div>
            <div class="grid-extras">
              <label class="opcion-extra" v-for="extra in serviciosExtraDisponibles" :key="extra.id">
                <input type="checkbox" v-model="fServiciosExtra" :value="extra.nombre" @change="recalcularPrecio">
                <span>
                  <span class="nombre-extra">{{ extra.nombre }}</span>
                  <span class="precio-extra">+${{ formatearNumero(extra.precio) }}</span>
                </span>
              </label>
            </div>
          </div>

          <div class="campo-formulario">
            <div class="encabezado-campo">
              <label>Fotos antes / después</label>
              <span class="badge-opcional">Máx. 1 por lado</span>
            </div>
            <div class="grid-fotos">
              <div class="subida-foto">
                <p class="etiqueta-dato">Antes</p>
                <img v-if="fFotoAntes" :src="fFotoAntes" class="miniatura-foto">
                <input type="file" accept="image/*" @change="manejarFoto($event, 'antes')">
                <button v-if="fFotoAntes" type="button" class="boton-quitar-foto" @click="quitarFoto('antes')">Quitar</button>
              </div>
              <div class="subida-foto">
                <p class="etiqueta-dato">Después</p>
                <img v-if="fFotoDespues" :src="fFotoDespues" class="miniatura-foto">
                <input type="file" accept="image/*" @change="manejarFoto($event, 'despues')">
                <button v-if="fFotoDespues" type="button" class="boton-quitar-foto" @click="quitarFoto('despues')">Quitar</button>
              </div>
            </div>
          </div>

          <div class="fila-formulario">
            <div class="campo-formulario">
              <label>Precio total (CLP)</label>
              <div class="campo-dinero">
                <span class="simbolo-dinero">$</span>
                <input
                  type="text"
                  :value="fPrecioDisplay"
                  readonly
                  tabindex="-1"
                  placeholder="Selecciona un servicio"
                >
              </div>
            </div>
            <div class="campo-formulario">
              <label>Propina (opcional)</label>
              <div class="campo-dinero">
                <span class="simbolo-dinero">$</span>
                <input type="text" inputmode="numeric" v-model="fPropinaDisplay" placeholder="0">
              </div>
            </div>
          </div>

          <div class="fila-formulario">
            <div class="campo-formulario">
              <label>Método de pago acordado</label>
              <select v-model="fMetodoPago">
                <option value="">Selecciona...</option>
                <option v-for="metodo in metodosPago" :key="metodo" :value="metodo">{{ metodo }}</option>
              </select>
            </div>
            <div class="campo-formulario">
              <label>Estado de pago</label>
              <select v-model="fEstadoPago">
                <option value="Pendiente">Pendiente</option>
                <option value="Abonado">Abonado (Parcial)</option>
                <option value="Pagado">Pagado (Total)</option>
              </select>
            </div>
          </div>

          <div class="campo-formulario" v-if="fEstadoPago === 'Abonado'">
            <label>Monto abonado (CLP)</label>
            <div class="campo-dinero">
              <span class="simbolo-dinero">$</span>
              <input type="text" inputmode="numeric" v-model="fMontoAbonadoDisplay" placeholder="Ej: 5.000">
            </div>
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

    <!-- ---------- modal: catálogo de servicios ---------- -->
    <div v-if="panelCatalogoAbierto" class="fondo-modal">
      <div class="caja-modal">
        <button class="boton-cerrar" @click="cerrarPanelCatalogo">✕</button>
        <p class="eyebrow-modal">● Don Ramiro · Catálogo</p>
        <h2>Servicios de la barbería</h2>
        <p class="descripcion-modal">Crea, edita o elimina los servicios y su precio base sugerido.</p>

        <form @submit.prevent="guardarServicioCatalogo" class="fila-formulario-catalogo">
          <div class="campo-formulario">
            <label>Nombre del servicio</label>
            <input type="text" v-model="fNombreServicioCatalogo" placeholder="Ej: Corte Texturizado">
          </div>
          <div class="campo-formulario campo-tipo-catalogo">
            <label>Tipo</label>
            <select v-model="fTipoServicioCatalogo">
              <option value="principal">Principal (corte)</option>
              <option value="adicional">Adicional (barba, cejas...)</option>
            </select>
          </div>
          <div class="campo-formulario campo-precio-catalogo">
            <label>Precio base (CLP)</label>
            <div class="campo-dinero">
              <span class="simbolo-dinero">$</span>
              <input type="number" min="0" v-model="fPrecioServicioCatalogo" placeholder="0">
            </div>
          </div>
          <div class="campo-formulario campo-boton-catalogo">
            <button type="submit" class="boton-guardar">{{ idEdicionServicioCatalogo ? 'Guardar' : '+ Agregar' }}</button>
            <button v-if="idEdicionServicioCatalogo" type="button" class="boton-cancelar boton-cancelar-catalogo" @click="cancelarEdicionCatalogo">Cancelar</button>
          </div>
        </form>

        <div class="lista-catalogo">
          <div class="item-catalogo" v-for="s in catalogoServicios" :key="s.id">
            <span class="nombre-item-catalogo">{{ s.nombre }}</span>
            <span class="tipo-item-catalogo" :class="{ adicional: s.tipo === 'adicional' }">{{ s.tipo === 'adicional' ? 'Adicional' : 'Principal' }}</span>
            <span class="precio-item-catalogo">${{ formatearNumero(s.precio) }}</span>
            <div class="acciones-item-catalogo">
              <button type="button" class="boton-icono" @click="editarServicioCatalogo(s)">
                <span class="material-symbols-outlined icono">edit</span>
              </button>
              <button type="button" class="boton-icono boton-icono-eliminar" @click="eliminarServicioCatalogo(s.id)">
                <span class="material-symbols-outlined icono">delete</span>
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- ---------- modal: cierre de caja ---------- -->
    <div v-if="cierreCajaAbierto" class="fondo-modal">
      <div class="caja-modal">
        <button class="boton-cerrar" @click="cerrarVentanaCierre">✕</button>
        <p class="eyebrow-modal">● Don Ramiro · Cierre</p>
        <h2>Resumen de caja del día</h2>
        <p class="descripcion-modal">Revisa los totales antes de archivar los servicios de hoy.</p>

        <div class="resumen-cierre" v-if="resumenCierre">
          <div class="fila-resumen">
            <span>Total en efectivo</span>
            <strong>${{ formatearNumero(resumenCierre.efectivo) }} CLP</strong>
          </div>
          <div class="fila-resumen">
            <span>Total en transferencia</span>
            <strong>${{ formatearNumero(resumenCierre.transferencia) }} CLP</strong>
          </div>
          <div class="fila-resumen">
            <span>Total con tarjeta / débito</span>
            <strong>${{ formatearNumero(resumenCierre.tarjeta) }} CLP</strong>
          </div>
          <div class="divisor-tarjeta"></div>
          <div class="fila-resumen fila-resumen-pendiente">
            <span>Pendiente por cobrar</span>
            <strong>${{ formatearNumero(resumenCierre.pendientePorCobrar) }} CLP</strong>
          </div>
          <div class="divisor-tarjeta"></div>
          <p class="etiqueta-dato">Comisiones del día</p>
          <div class="fila-resumen" v-for="c in resumenCierre.comisiones" :key="c.nombre">
            <span>{{ c.nombre }} ({{ c.porcentaje }}%)</span>
            <strong>${{ formatearNumero(c.montoComision) }} CLP</strong>
          </div>
        </div>

        <div class="botones-modal">
          <button type="button" class="boton-cancelar" @click="cerrarVentanaCierre">Cancelar</button>
          <button type="button" class="boton-guardar" @click="confirmarCierreCaja">Archivar y cerrar caja</button>
        </div>
      </div>
    </div>
  </div>
</template>