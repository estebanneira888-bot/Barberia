<script setup>
import { ref } from 'vue'
import { useLocalStorage } from '@vueuse/core'

const servicios = useLocalStorage('barberiaRamiro', [])
const catalogo = useLocalStorage('barberiaCatalogo', [
  { id: 1, nombre: 'Corte tradicional', precio: 20000 },
  { id: 2, nombre: 'Corte moderno', precio: 25000 },
  { id: 3, nombre: 'Barba', precio: 15000 },
  { id: 4, nombre: 'Limpieza facial', precio: 15000 },
  { id: 5, nombre: 'Cejas', precio: 10000 },
  { id: 6, nombre: 'Tinte', precio: 50000 }
])

const verFormulario = ref(false)
const verEliminar = ref(false)
const verCaja = ref(false)
const verCatalogo = ref(false)
const editando = ref(false)
const servicioSeleccionado = ref(null)
const fecha = ref('')
const nombre = ref('')
const servicio = ref([])
const barbero = ref('')
const hora = ref('')
const precio = ref(0)
const propina = ref(0)
const pago = ref('')
const estado = ref('')
const nota = ref('')
const mensaje = ref('')
const editandoNota = ref(null)
const notaTemporal = ref('')
const orden = ref('fecha')
const historialNombre = ref('')
const historialVeces = ref(0)
const historialGastado = ref(0)
const nuevoNombreServicio = ref('')
const nuevoPrecioServicio = ref('')
const servicioCatalogoEditando = ref(null)
const cajaCerrada = ref(false)
const mensajeCaja = ref('')
const antesBase64 = ref('')
const despuesBase64 = ref('')
const imagenesMensaje = ref('')

const barberos = [
  { nombre: 'Don Ramiro', porcentaje: 50 },
  { nombre: 'Carlos', porcentaje: 50 },
  { nombre: 'Miguel', porcentaje: 50 }
]

function obtenerFechaHoy() {
  const hoy = new Date()
  const año = hoy.getFullYear()
  const mes = String(hoy.getMonth() + 1).padStart(2, '0')
  const dia = String(hoy.getDate()).padStart(2, '0')
  return `${año}-${mes}-${dia}`
}

const fechaMinima = obtenerFechaHoy()

function obtenerHoraActual() {
  const ahora = new Date()
  return String(ahora.getHours()).padStart(2, '0') + ':' + String(ahora.getMinutes()).padStart(2, '0')
}

function formatearPrecio(valor) {
  if (valor === '' || valor === null || valor === undefined) return '$0'
  return Number(valor).toLocaleString('es-CO', { style: 'currency', currency: 'COP', minimumFractionDigits: 0 })
}

function nombreValido(nombreTexto) {
  return /^[A-Za-zÁÉÍÓÚáéíóúÑñÜü\s]{3,}$/.test(nombreTexto.trim())
}

function obtenerPrecioServicios(lista) {
  let total = 0
  for (let i = 0; i < lista.length; i++) {
    for (let j = 0; j < catalogo.value.length; j++) {
      if (lista[i] === catalogo.value[j].nombre) total += Number(catalogo.value[j].precio)
    }
  }
  return total
}

function cambiarServicio() {
  precio.value = obtenerPrecioServicios(servicio.value)
}

function calcularTotal(item) {
  if (!item) return Number(precio.value) + Number(propina.value || 0) - calcularDescuentoActual()
  if (item.total !== undefined && item.total !== null) return Number(item.total)
  return Number(item.precio || 0) + Number(item.propina || 0) - Number(item.descuento || 0)
}

function descuentoCliente(nombreCliente, excluirId = null) {
  const nombreLimpio = nombreCliente.trim().toLowerCase()
  let cantidad = 0
  for (let i = 0; i < servicios.value.length; i++) {
    const item = servicios.value[i]
    if (item.id !== excluirId && item.nombre.trim().toLowerCase() === nombreLimpio) cantidad++
  }
  return cantidad >= 5
}

function calcularDescuentoActual() {
  return descuentoCliente(nombre.value, editando.value ? servicioSeleccionado.value : null) ? Math.round(Number(precio.value) * 0.10) : 0
}

function totalFormulario() {
  const subtotal = Number(precio.value) + Number(propina.value || 0)
  return subtotal - calcularDescuentoActual()
}

function nuevoServicio() {
  limpiar()
  editando.value = false
  fecha.value = fechaMinima
  verFormulario.value = true
}

function limpiar() {
  nombre.value = ''
  servicio.value = []
  barbero.value = ''
  fecha.value = ''
  hora.value = ''
  precio.value = 0
  propina.value = 0
  pago.value = ''
  estado.value = ''
  nota.value = ''
  mensaje.value = ''
  servicioSeleccionado.value = null
  antesBase64.value = ''
  despuesBase64.value = ''
  imagenesMensaje.value = ''
}

function cerrarFormulario() {
  verFormulario.value = false
  limpiar()
}

function validarHora() {
  if (fecha.value === fechaMinima && hora.value !== '' && hora.value < obtenerHoraActual()) {
    hora.value = ''
    mensaje.value = 'La hora seleccionada ya pasó. Elige una hora posterior a la actual.'
  } else if (mensaje.value.includes('hora')) {
    mensaje.value = ''
  }
}

function seleccionarServicio(nombreServicio) {
  const posicion = servicio.value.indexOf(nombreServicio)
  if (posicion >= 0) servicio.value.splice(posicion, 1)
  else servicio.value.push(nombreServicio)
  cambiarServicio()
}

function manejarImagen(evento, tipo) {
  const archivo = evento.target.files[0]
  if (!archivo) return
  if (archivo.size > 900000) {
    imagenesMensaje.value = 'Cada imagen debe pesar menos de 900 KB.'
    evento.target.value = ''
    return
  }
  const lector = new FileReader()
  lector.onload = () => {
    const imagen = new Image()
    imagen.onload = () => {
      const maximo = 700
      let ancho = imagen.width
      let alto = imagen.height
      if (ancho > maximo || alto > maximo) {
        if (ancho > alto) {
          alto = Math.round(alto * maximo / ancho)
          ancho = maximo
        } else {
          ancho = Math.round(ancho * maximo / alto)
          alto = maximo
        }
      }
      const lienzo = document.createElement('canvas')
      lienzo.width = ancho
      lienzo.height = alto
      lienzo.getContext('2d').drawImage(imagen, 0, 0, ancho, alto)
      const base64 = lienzo.toDataURL('image/jpeg', 0.60)
      if (tipo === 'antes') antesBase64.value = base64
      else despuesBase64.value = base64
      imagenesMensaje.value = ''
    }
    imagen.src = lector.result
  }
  lector.readAsDataURL(archivo)
}

function guardar() {
  mensaje.value = ''
  nombre.value = nombre.value.trim()

  const faltantes = []
  if (!nombre.value) faltantes.push('Nombre')
  if (!servicio.value.length) faltantes.push('Servicio')
  if (!barbero.value) faltantes.push('Barbero')
  if (!fecha.value) faltantes.push('Fecha')
  if (!hora.value) faltantes.push('Hora')
  if (!pago.value) faltantes.push('Método de pago')
  if (!estado.value) faltantes.push('Estado')

  if (faltantes.length) {
    mensaje.value = faltantes.length === 1 ? `Completa el campo: ${faltantes[0]}` : `Completa los campos: ${faltantes.join(', ')}`
    return
  }
  if (!nombreValido(nombre.value)) {
    mensaje.value = 'El nombre debe contener solamente letras y espacios'
    return
  }
  if (fecha.value < fechaMinima) {
    mensaje.value = 'La fecha no puede ser anterior a hoy'
    return
  }
  if (fecha.value === fechaMinima && hora.value < obtenerHoraActual()) {
    mensaje.value = 'La hora no puede ser anterior a la hora actual'
    return
  }
  if (Number(propina.value) < 0) {
    mensaje.value = 'La propina no puede ser negativa'
    return
  }

  const descuento = calcularDescuentoActual()
  const total = Number(precio.value) + Number(propina.value || 0) - descuento

  if (total <= 0) {
    mensaje.value = 'El precio debe ser mayor a cero'
    return
  }

  if (!editando.value) {
    servicios.value.push({
      id: Date.now(),
      nombre: nombre.value,
      servicio: [...servicio.value],
      barbero: barbero.value,
      fecha: fecha.value,
      hora: hora.value,
      precio: Number(precio.value),
      propina: Number(propina.value || 0),
      descuento: descuento,
      total: total,
      pago: pago.value,
      estado: estado.value,
      estrellas: 0,
      nota: nota.value,
      archivado: false,
      antes: antesBase64.value,
      despues: despuesBase64.value
    })
  } else {
    for (let i = 0; i < servicios.value.length; i++) {
      if (servicios.value[i].id === servicioSeleccionado.value) {
        servicios.value[i].nombre = nombre.value
        servicios.value[i].servicio = [...servicio.value]
        servicios.value[i].barbero = barbero.value
        servicios.value[i].fecha = fecha.value
        servicios.value[i].hora = hora.value
        servicios.value[i].precio = Number(precio.value)
        servicios.value[i].propina = Number(propina.value || 0)
        servicios.value[i].descuento = descuento
        servicios.value[i].total = total
        servicios.value[i].pago = pago.value
        servicios.value[i].estado = estado.value
        servicios.value[i].nota = nota.value
        servicios.value[i].antes = antesBase64.value
        servicios.value[i].despues = despuesBase64.value
      }
    }
  }
  cerrarFormulario()
}

function editar(item) {
  editando.value = true
  verFormulario.value = true
  servicioSeleccionado.value = item.id
  nombre.value = item.nombre
  servicio.value = Array.isArray(item.servicio) ? [...item.servicio] : item.servicio ? [item.servicio] : []
  barbero.value = item.barbero
  fecha.value = item.fecha
  hora.value = item.hora || ''
  precio.value = Number(item.precio || obtenerPrecioServicios(servicio.value))
  propina.value = Number(item.propina || 0)
  pago.value = item.pago
  estado.value = item.estado
  nota.value = item.nota || ''
  antesBase64.value = item.antes || ''
  despuesBase64.value = item.despues || ''
  mensaje.value = ''
}

function confirmarEliminar(id) {
  servicioSeleccionado.value = id
  verEliminar.value = true
}

function eliminar() {
  for (let i = 0; i < servicios.value.length; i++) {
    if (servicios.value[i].id === servicioSeleccionado.value) {
      servicios.value.splice(i, 1)
      break
    }
  }
  verEliminar.value = false
  servicioSeleccionado.value = null
}

function cancelarEliminar() {
  verEliminar.value = false
  servicioSeleccionado.value = null
}

function serviciosActivos() {
  const lista = []
  for (let i = 0; i < servicios.value.length; i++) {
    if (!servicios.value[i].archivado) lista.push(servicios.value[i])
  }
  return lista
}

function serviciosParaMostrar() {
  const lista = serviciosActivos().slice()
  if (orden.value === 'precio') lista.sort((a, b) => calcularTotal(b) - calcularTotal(a))
  else if (orden.value === 'calificacion') lista.sort((a, b) => Number(b.estrellas || 0) - Number(a.estrellas || 0))
  else lista.sort((a, b) => `${a.fecha} ${a.hora}`.localeCompare(`${b.fecha} ${b.hora}`))
  return lista
}

function totalVendido() {
  let total = 0
  for (let i = 0; i < servicios.value.length; i++) {
    if (servicios.value[i].estado === 'Pagado') total += calcularTotal(servicios.value[i])
  }
  return total
}

function dineroRecibido() {
  return totalVendido()
}

function cantidadPendientes() {
  let cantidad = 0
  const lista = serviciosActivos()
  for (let i = 0; i < lista.length; i++) if (lista[i].estado === 'Pendiente' || lista[i].estado === 'Fiado') cantidad++
  return cantidad
}

function promedioCalificacion() {
  let total = 0
  let cantidad = 0
  for (let i = 0; i < servicios.value.length; i++) {
    if (Number(servicios.value[i].estrellas || 0) > 0) {
      total += Number(servicios.value[i].estrellas)
      cantidad++
    }
  }
  return cantidad ? (total / cantidad).toFixed(1) : '0.0'
}

function barberoMasCortes() {
  const conteo = {}
  const hoy = obtenerFechaHoy()
  const lista = servicios.value
  for (let i = 0; i < lista.length; i++) {
    if (lista[i].fecha === hoy) conteo[lista[i].barbero] = (conteo[lista[i].barbero] || 0) + 1
  }
  let ganador = 'Sin servicios'
  let mayor = 0
  for (const nombreBarbero in conteo) {
    if (conteo[nombreBarbero] > mayor) {
      mayor = conteo[nombreBarbero]
      ganador = `${nombreBarbero} (${mayor})`
    }
  }
  return ganador
}

function seleccionarEstrellas(item, numero) {
  item.estrellas = numero
}

function editarNota(item) {
  editandoNota.value = item.id
  notaTemporal.value = item.nota || ''
}

function guardarNota(item) {
  item.nota = notaTemporal.value
  editandoNota.value = null
  notaTemporal.value = ''
}

function cancelarNota() {
  editandoNota.value = null
  notaTemporal.value = ''
}

function historialCliente() {
  const buscado = historialNombre.value.trim().toLowerCase()
  let veces = 0
  let gastado = 0
  if (buscado) {
    for (let i = 0; i < servicios.value.length; i++) {
      const item = servicios.value[i]
      if (item.nombre.trim().toLowerCase() === buscado) {
        veces++
        gastado += calcularTotal(item)
      }
    }
  }
  historialVeces.value = veces
  historialGastado.value = gastado
}

function esClienteFrecuente() {
  return nombre.value.trim() !== '' && descuentoCliente(nombre.value, editando.value ? servicioSeleccionado.value : null)
}

function turno(item) {
  const horaItem = Number((item.hora || '00:00').split(':')[0])
  if (horaItem < 12) return 'Mañana'
  if (horaItem < 18) return 'Tarde'
  return 'Noche'
}

function mostrarSeparador(item, index, lista) {
  if (index === 0) return true
  return turno(item) !== turno(lista[index - 1])
}

function totalEfectivo() {
  let total = 0
  const lista = serviciosActivos()
  for (let i = 0; i < lista.length; i++) if (lista[i].fecha === fechaMinima && lista[i].pago === 'Efectivo' && lista[i].estado === 'Pagado') total += calcularTotal(lista[i])
  return total
}

function totalTransferencia() {
  let total = 0
  const lista = serviciosActivos()
  for (let i = 0; i < lista.length; i++) if (lista[i].fecha === fechaMinima && lista[i].pago === 'Transferencia' && lista[i].estado === 'Pagado') total += calcularTotal(lista[i])
  return total
}

function pendientesCobrar() {
  let total = 0
  const lista = serviciosActivos()
  for (let i = 0; i < lista.length; i++) if (lista[i].fecha === fechaMinima && (lista[i].estado === 'Pendiente' || lista[i].estado === 'Fiado')) total += calcularTotal(lista[i])
  return total
}

function cerrarCaja() {
  verCaja.value = true
  mensajeCaja.value = ''
}

function archivarServiciosDelDia() {
  const lista = servicios.value
  for (let i = 0; i < lista.length; i++) if (lista[i].fecha === fechaMinima && !lista[i].archivado) lista[i].archivado = true
  cajaCerrada.value = true
  verCaja.value = false
  mensajeCaja.value = 'Caja cerrada y servicios del día archivados.'
}

function comisionesBarberos() {
  const resultado = []
  const lista = servicios.value
  for (let i = 0; i < barberos.length; i++) {
    let base = 0
    let cortes = 0
    for (let j = 0; j < lista.length; j++) {
      if (lista[j].fecha === fechaMinima && lista[j].barbero === barberos[i].nombre) {
        base += Number(lista[j].precio || 0) - Number(lista[j].descuento || 0)
        cortes++
      }
    }
    resultado.push({ ...barberos[i], cortes, base, comision: Math.round(base * barberos[i].porcentaje / 100) })
  }
  return resultado
}

function deudasClientes() {
  const deudas = {}
  const lista = servicios.value
  for (let i = 0; i < lista.length; i++) {
    if (lista[i].estado === 'Fiado') {
      deudas[lista[i].nombre] = (deudas[lista[i].nombre] || 0) + calcularTotal(lista[i])
    }
  }
  const resultado = []
  for (const cliente in deudas) resultado.push({ nombre: cliente, total: deudas[cliente] })
  return resultado
}

function iniciarEdicionCatalogo(item) {
  servicioCatalogoEditando.value = item.id
  nuevoNombreServicio.value = item.nombre
  nuevoPrecioServicio.value = item.precio
}

function guardarServicioCatalogo() {
  const nombreLimpio = nuevoNombreServicio.value.trim()
  const valor = Number(nuevoPrecioServicio.value)
  if (!nombreLimpio || valor <= 0) return
  if (servicioCatalogoEditando.value === null) {
    catalogo.value.push({ id: Date.now(), nombre: nombreLimpio, precio: valor })
  } else {
    for (let i = 0; i < catalogo.value.length; i++) {
      if (catalogo.value[i].id === servicioCatalogoEditando.value) {
        const anterior = catalogo.value[i].nombre
        catalogo.value[i].nombre = nombreLimpio
        catalogo.value[i].precio = valor
        for (let j = 0; j < servicios.value.length; j++) {
          if (Array.isArray(servicios.value[j].servicio)) {
            for (let k = 0; k < servicios.value[j].servicio.length; k++) if (servicios.value[j].servicio[k] === anterior) servicios.value[j].servicio[k] = nombreLimpio
          }
        }
      }
    }
  }
  nuevoNombreServicio.value = ''
  nuevoPrecioServicio.value = ''
  servicioCatalogoEditando.value = null
  cambiarServicio()
}

function cancelarEdicionCatalogo() {
  nuevoNombreServicio.value = ''
  nuevoPrecioServicio.value = ''
  servicioCatalogoEditando.value = null
}

function eliminarServicioCatalogo(id) {
  if (catalogo.value.length <= 1) return
  for (let i = 0; i < catalogo.value.length; i++) if (catalogo.value[i].id === id) catalogo.value.splice(i, 1)
  servicio.value = servicio.value.filter(nombreServicio => catalogo.value.some(item => item.nombre === nombreServicio))
  cambiarServicio()
}
</script>

<template>
  <main>
    <section class="inicio">
      <div>
        <p class="pequeno">SISTEMA DE REGISTRO</p>
        <h1>💈 Barbería Don Ramiro</h1>
        <p class="descripcion">Servicio 24/7<br>Control de servicios, clientes y pagos.</p>
      </div>
      <button class="boton-principal" @click="nuevoServicio">+ Registrar servicio</button>
    </section>

    <section class="resumen">
      <div class="caja-resumen"><span>✂️</span><div><small>Servicios</small><h2>{{ serviciosActivos().length }}</h2></div></div>
      <div class="caja-resumen"><span>💰</span><div><small>Total vendido</small><h2>{{ formatearPrecio(totalVendido()) }}</h2></div></div>
      <div class="caja-resumen"><span>⭐</span><div><small>Promedio</small><h2>{{ promedioCalificacion() }}/5</h2></div></div>
      <div class="caja-resumen"><span>🏆</span><div><small>Más cortes hoy</small><h2>{{ barberoMasCortes() }}</h2></div></div>
    </section>

    <section class="paneles-superiores">
      <div class="panel">
        <div class="panel-titulo"><h2>📊 Estadísticas del día</h2><span>{{ fechaMinima }}</span></div>
        <div class="estadisticas">
          <div><strong>{{ servicios.filter(item => item.fecha === fechaMinima).length }}</strong><small>Servicios hoy</small></div>
          <div><strong>{{ formatearPrecio(totalVendido()) }}</strong><small>Total vendido</small></div>
          <div><strong>{{ promedioCalificacion() }}</strong><small>Calificación</small></div>
          <div><strong>{{ barberoMasCortes() }}</strong><small>Barbero destacado</small></div>
        </div>
      </div>

      <div class="panel">
        <div class="panel-titulo"><h2>🔎 Historial por cliente</h2></div>
        <input v-model="historialNombre" @input="historialCliente" class="input-panel" placeholder="Escriba el nombre del cliente">
        <div v-if="historialNombre.trim()" class="historial-resultado">
          <div><strong>{{ historialVeces }}</strong><span>veces ha venido</span></div>
          <div><strong>{{ formatearPrecio(historialGastado) }}</strong><span>gastado en total</span></div>
        </div>
      </div>
    </section>

    <section class="paneles-superiores">
      <div class="panel">
        <div class="panel-titulo"><h2>💸 Comisiones por barbero</h2></div>
        <div class="tabla-comisiones">
          <div v-for="item in comisionesBarberos()" :key="item.nombre" class="fila-comision">
            <span>{{ item.nombre }} <small>{{ item.porcentaje }}%</small></span>
            <span>{{ item.cortes }} cortes</span>
            <strong>{{ formatearPrecio(item.comision) }}</strong>
          </div>
        </div>
      </div>

      <div class="panel panel-deudas">
        <div class="panel-titulo"><h2>⚠️ Recordatorios de deudas</h2></div>
        <div v-if="deudasClientes().length === 0" class="sin-datos">No hay clientes con deuda.</div>
        <div v-for="deuda in deudasClientes()" :key="deuda.nombre" class="fila-deuda">
          <span>{{ deuda.nombre }}</span><strong>{{ formatearPrecio(deuda.total) }}</strong>
        </div>
      </div>
    </section>

    <section class="contenido">
      <div class="titulo-lista">
        <div><h2>Registro de servicios</h2><p>{{ serviciosActivos().length }} servicios activos</p></div>
        <div class="acciones-lista">
          <button :class="{ activo: orden === 'fecha' }" @click="orden = 'fecha'">📅 Fecha</button>
          <button :class="{ activo: orden === 'precio' }" @click="orden = 'precio'">💰 Precio</button>
          <button :class="{ activo: orden === 'calificacion' }" @click="orden = 'calificacion'">⭐ Calificación</button>
          <button class="btn-catalogo" @click="verCatalogo = true">⚙️ Catálogo</button>
          <button class="btn-caja" @click="cerrarCaja">🔒 Cerrar caja</button>
        </div>
      </div>

      <div v-if="mensajeCaja" class="mensaje-exito">✓ {{ mensajeCaja }}</div>

      <div v-if="serviciosActivos().length === 0" class="vacio">
        <div class="icono-vacio">😕</div><h2>No hay servicios registrados</h2><p>Comienza registrando el primer cliente del día.</p><button @click="nuevoServicio">Registrar ahora</button>
      </div>

      <div v-else class="lista-turnos">
        <template v-for="(item, index) in serviciosParaMostrar()" :key="item.id">
          <div v-if="mostrarSeparador(item, index, serviciosParaMostrar())" class="separador-turno"><span>{{ turno(item) }}</span></div>
          <article class="servicio-card" :class="{ sinPagar: item.estado === 'Pendiente', fiadoCard: item.estado === 'Fiado' }">
            <div class="card-superior">
              <div class="avatar">{{ item.nombre.charAt(0) }}</div>
              <div><h3>{{ item.nombre }}</h3><p>{{ Array.isArray(item.servicio) ? item.servicio.join(' + ') : item.servicio }}</p></div>
              <span v-if="item.estado === 'Pagado'" class="estado pagado">✓ Pagado</span>
              <span v-else-if="item.estado === 'Pendiente'" class="estado pendiente">⏳ Pendiente</span>
              <span v-else class="estado fiado">⚠ Fiado</span>
            </div>

            <div class="datos">
              <div><small>BARBERO</small><p>👤 {{ item.barbero }}</p></div>
              <div><small>PRECIO</small><p>{{ formatearPrecio(item.precio) }}<span v-if="item.propina"> + {{ formatearPrecio(item.propina) }} propina</span></p></div>
              <div><small>TOTAL</small><p><strong>{{ formatearPrecio(calcularTotal(item)) }}</strong></p></div>
              <div><small>FECHA</small><p>📅 {{ item.fecha }}</p></div>
              <div><small>HORA</small><p>⏳ {{ item.hora || 'No registrada' }}</p></div>
              <div><small>PAGO</small><p>{{ item.pago === 'Efectivo' ? '💵' : item.pago === 'Transferencia' ? '📲' : '💳' }} {{ item.pago }}</p></div>
            </div>

            <div v-if="item.descuento" class="descuento-card">🎉 Descuento de fidelidad: -{{ formatearPrecio(item.descuento) }}</div>

            <div class="calificacion"><div class="estrellas-card"><button v-for="numero in 5" :key="numero" type="button" class="estrella" :class="{ activa: numero <= (item.estrellas || 0) }" @click="seleccionarEstrellas(item, numero)">{{ numero <= (item.estrellas || 0) ? '★' : '☆' }}</button></div><small>{{ item.estrellas || 0 }}/5</small></div>

            <div v-if="item.antes || item.despues" class="fotos-card">
              <div v-if="item.antes"><small>ANTES</small><img :src="item.antes" alt="Antes"></div>
              <div v-if="item.despues"><small>DESPUÉS</small><img :src="item.despues" alt="Después"></div>
            </div>

            <div class="observacion">
              <div class="titulo-observacion"><strong>📝 Observaciones</strong><button v-if="editandoNota !== item.id" class="btn-editar-nota" type="button" @click="editarNota(item)">✏️</button></div>
              <div v-if="editandoNota !== item.id" class="texto-observacion"><span v-if="item.nota && item.nota.trim() !== ''">{{ item.nota }}</span><span v-else class="sin-observacion">Sin observaciones</span></div>
              <div v-else class="editar-observacion"><textarea v-model="notaTemporal" placeholder="Escriba una observación..."></textarea><div class="botones-nota"><button type="button" class="btn-cancelar-nota" @click="cancelarNota">Cancelar</button><button type="button" class="btn-guardar-nota" @click="guardarNota(item)">💾 Guardar</button></div></div>
            </div>

            <div class="acciones"><button class="btn-editar" @click="editar(item)">✏️ Editar</button><button class="btn-borrar" @click="confirmarEliminar(item.id)">🗑</button></div>
          </article>
        </template>
      </div>
    </section>

    <div v-show="verFormulario" class="fondo-modal">
      <div class="ventana">
        <div class="cabecera-modal"><div><h2>{{ editando ? 'Editar servicio' : 'Nuevo servicio💈' }}</h2><p>Complete la información del cliente.</p></div><button class="x" @click="cerrarFormulario">❌</button></div>
        <form @submit.prevent="guardar">
          <div v-if="mensaje" class="mensaje-error">⚠️ {{ mensaje }}</div>
          <div v-if="esClienteFrecuente()" class="alerta-fidelidad">🎉 ¡Cliente frecuente, aplica 10% de descuento!</div>
          <div class="grupo"><label>Nombre del cliente😉</label><input v-model="nombre" @input="historialNombre = nombre; historialCliente()" type="text" placeholder="Escriba el nombre completo" maxlength="50"></div>
          <div class="grupo"><label>Servicios a realizar🙌</label><div class="servicios-cuadricula"><button v-for="item in catalogo" :key="item.id" type="button" class="cuadro-servicio" :class="{ seleccionado: servicio.includes(item.nombre) }" @click="seleccionarServicio(item.nombre)"><span class="cuadro-check">{{ servicio.includes(item.nombre) ? '✓' : '' }}</span><span class="nombre-servicio">{{ item.nombre }}</span><span class="precio-servicio">{{ formatearPrecio(item.precio) }}</span></button></div></div>
          <div class="dos-columnas">
            <div class="grupo"><label>Barbero👦🏻</label><select v-model="barbero"><option value="">Seleccione</option><option v-for="item in barberos" :key="item.nombre" :value="item.nombre">{{ item.nombre }} — {{ item.porcentaje }}%</option></select></div>
            <div class="grupo"><label>Precio💸</label><input :value="formatearPrecio(precio)" type="text" readonly></div>
          </div>
          <div class="dos-columnas">
            <div class="grupo"><label>Fecha📅</label><input v-model="fecha" type="date" :min="fechaMinima" @change="validarHora"></div>
            <div class="grupo"><label>Hora⏳</label><input v-model="hora" type="time" :min="fecha === fechaMinima ? obtenerHoraActual() : '00:00'" @change="validarHora"></div>
          </div>
          <div class="dos-columnas">
            <div class="grupo"><label>Método de pago 😅</label><select v-model="pago"><option value="">Seleccione</option><option>Efectivo</option><option>Transferencia</option><option>Tarjeta</option></select></div>
            <div class="grupo"><label>Estado ❓</label><select v-model="estado"><option value="">Seleccione</option><option>Pagado</option><option>Pendiente</option><option>Fiado</option></select></div>
          </div>
          <div class="dos-columnas">
            <div class="grupo"><label>Propina opcional 💵</label><input v-model.number="propina" type="number" min="0" step="1000" placeholder="0"></div>
            <div class="grupo"><label>Total</label><input :value="formatearPrecio(totalFormulario())" type="text" readonly></div>
          </div>
          <div v-if="esClienteFrecuente()" class="total-descuento">Subtotal: {{ formatearPrecio(Number(precio) + Number(propina || 0)) }} · Descuento: {{ formatearPrecio(calcularDescuentoActual()) }} · Total: {{ formatearPrecio(totalFormulario()) }}</div>
          <div class="grupo"><label>Fotos antes y después 📷</label><div class="fotos-inputs"><label class="subir-foto">Antes<input type="file" accept="image/*" @change="manejarImagen($event, 'antes')"></label><label class="subir-foto">Después<input type="file" accept="image/*" @change="manejarImagen($event, 'despues')"></label></div><small class="ayuda">Máximo 1 foto de antes y 1 de después por servicio.</small><p v-if="imagenesMensaje" class="mensaje-error">{{ imagenesMensaje }}</p></div>
          <div v-if="antesBase64 || despuesBase64" class="previsualizaciones"><div v-if="antesBase64"><small>Antes</small><img :src="antesBase64" alt="Vista previa antes"></div><div v-if="despuesBase64"><small>Después</small><img :src="despuesBase64" alt="Vista previa después"></div></div>
          <div class="grupo"><label>Observación 📝</label><textarea v-model="nota" placeholder="Escriba una observación..."></textarea></div>
          <div class="pie-modal"><button type="button" class="cancelar" @click="cerrarFormulario">Cancelar</button><button type="submit" class="guardar">{{ editando ? 'Actualizar servicio' : 'Guardar servicio' }}</button></div>
        </form>
      </div>
    </div>

    <div v-show="verEliminar" class="fondo-modal"><div class="confirmar"><div class="alerta">🗑️</div><h2>¿Eliminar registro?</h2><p>El servicio será eliminado y no podrás recuperarlo.</p><div class="botones-confirmar"><button class="cancelar" @click="cancelarEliminar">No, cancelar</button><button class="btn-borrar" @click="eliminar">Sí, eliminar</button></div></div></div>

    <div v-show="verCaja" class="fondo-modal"><div class="ventana caja-ventana"><div class="cabecera-modal"><div><h2>🔒 Cierre de caja</h2><p>Resumen de los servicios de {{ fechaMinima }}.</p></div><button class="x" @click="verCaja = false">❌</button></div><div class="resumen-caja"><div><span>💵</span><small>Total en efectivo</small><strong>{{ formatearPrecio(totalEfectivo()) }}</strong></div><div><span>📲</span><small>Total en transferencia</small><strong>{{ formatearPrecio(totalTransferencia()) }}</strong></div><div><span>⚠️</span><small>Pendiente por cobrar</small><strong>{{ formatearPrecio(pendientesCobrar()) }}</strong></div></div><p class="aviso-caja">Al archivar, los servicios de hoy dejarán de aparecer en la vista principal.</p><div class="pie-modal"><button class="cancelar" @click="verCaja = false">Cancelar</button><button class="guardar" @click="archivarServiciosDelDia">Archivar servicios y cerrar caja</button></div></div></div>

    <div v-show="verCatalogo" class="fondo-modal"><div class="ventana catalogo-ventana"><div class="cabecera-modal"><div><h2>⚙️ Catálogo de servicios</h2><p>Don Ramiro puede crear y editar sus servicios.</p></div><button class="x" @click="verCatalogo = false">❌</button></div><div class="catalogo-form"><input v-model="nuevoNombreServicio" placeholder="Nombre del servicio"><input v-model.number="nuevoPrecioServicio" type="number" min="1" placeholder="Precio base sugerido"><button class="guardar" @click="guardarServicioCatalogo">{{ servicioCatalogoEditando === null ? 'Agregar' : 'Actualizar' }}</button><button v-if="servicioCatalogoEditando !== null" class="cancelar" @click="cancelarEdicionCatalogo">Cancelar</button></div><div class="catalogo-lista"><div v-for="item in catalogo" :key="item.id" class="fila-catalogo"><div><strong>{{ item.nombre }}</strong><span>{{ formatearPrecio(item.precio) }}</span></div><div><button class="btn-editar" @click="iniciarEdicionCatalogo(item)">✏️</button><button class="btn-borrar" @click="eliminarServicioCatalogo(item.id)">🗑</button></div></div></div></div></div>
  </main>
</template>

<style>
* { margin: 0; padding: 0; box-sizing: border-box; }
body { font-family: Arial, sans-serif; background: #f1f3f5; }
main { min-height: 100vh; background: #a2b7bd; }
button { border: none; padding: 11px 18px; border-radius: 7px; cursor: pointer; font-weight: bold; }
button:hover { filter: brightness(.96); }
.inicio { background: linear-gradient(rgba(20,20,20,.7),rgba(20,20,20,.8)),url('https://www.shutterstock.com/image-vector/barber-tools-haircut-icons-set-260nw-2619940863.jpg'); background-size: cover; background-position: center; color: white; padding: 45px 8%; min-height: 250px; display: flex; justify-content: space-between; align-items: center; }
.pequeno { font-size: 12px; letter-spacing: 2px; color: #f0c45b; font-weight: bold; }
.inicio h1,h2 { font-family: 'Times New Roman',Times,serif; }
.inicio h1 { margin: 8px 0; font-size: 50px; }
.descripcion { color: #eee; }
.boton-principal { background: #ff6700; color: white; font-size: 15px; }
.resumen { max-width: 1200px; margin: 25px auto; padding: 0 20px; display: grid; grid-template-columns: repeat(4,1fr); gap: 15px; }
.caja-resumen { background: white; padding: 20px; border-radius: 12px; display: flex; gap: 15px; align-items: center; box-shadow: 0 2px 8px #ddd; min-width: 0; }
.caja-resumen > span { font-size: 30px; }.caja-resumen small { color:#777; }.caja-resumen h2 { margin-top:5px; font-size:23px; }
.paneles-superiores { max-width:1200px; margin:0 auto 20px; padding:0 20px; display:grid; grid-template-columns:1fr 1fr; gap:15px; }
.panel { background:white; border-radius:12px; padding:18px; box-shadow:0 2px 8px #ddd; }.panel-titulo { display:flex; justify-content:space-between; gap:10px; align-items:center; margin-bottom:15px; }.panel-titulo h2 { font-size:22px; }.panel-titulo span { color:#777; font-size:13px; }
.estadisticas { display:grid; grid-template-columns:repeat(4,1fr); gap:10px; }.estadisticas div { background:#f7f8f9; border-radius:9px; padding:12px; text-align:center; }.estadisticas strong { display:block; font-size:18px; }.estadisticas small { color:#777; }
.input-panel { width:100%; padding:12px; border:1px solid #ccd3d7; border-radius:7px; outline:none; }.historial-resultado { display:grid; grid-template-columns:1fr 1fr; gap:10px; margin-top:12px; }.historial-resultado div { background:#f2f7ff; padding:12px; border-radius:8px; text-align:center; }.historial-resultado strong,.historial-resultado span { display:block; }.historial-resultado span { color:#777; font-size:12px; }
.tabla-comisiones { display:flex; flex-direction:column; gap:8px; }.fila-comision,.fila-deuda { display:grid; grid-template-columns:1.5fr 1fr 1fr; align-items:center; gap:10px; padding:10px; background:#f7f8f9; border-radius:7px; }.fila-comision small { color:#777; }.fila-comision strong,.fila-deuda strong { text-align:right; }.panel-deudas { max-height:250px; overflow:auto; }.fila-deuda { grid-template-columns:1fr 1fr; }.sin-datos { color:#777; padding:10px; }
.contenido { max-width:1200px; margin:auto; padding:0 20px 40px; }.titulo-lista { margin-bottom:20px; display:flex; justify-content:space-between; gap:15px; align-items:end; }.titulo-lista h2 { font-size:30px; }.titulo-lista p { color:#777; margin-top:5px; }.acciones-lista { display:flex; flex-wrap:wrap; gap:7px; }.acciones-lista button { background:#fff; border:1px solid #d5dadd; padding:9px 12px; }.acciones-lista .activo { background:#202020; color:white; }.acciones-lista .btn-catalogo { background:#e9eef0; }.acciones-lista .btn-caja { background:#d63031; color:white; }
.lista-turnos { display:flex; flex-direction:column; gap:15px; }.separador-turno { display:flex; align-items:center; gap:12px; margin:8px 0 0; color:#333; font-weight:bold; }.separador-turno:before,.separador-turno:after { content:''; height:1px; background:#8d9ca1; flex:1; }.separador-turno span { background:#dce5e7; padding:8px 15px; border-radius:20px; }
.servicio-card { background:white; padding:20px; border-radius:12px; border-top:5px solid #333; box-shadow:0 3px 10px #ddd; }.sinPagar { border-top-color:#f0a500; }.fiadoCard { border-top-color:#d63031; }.card-superior { display:flex; align-items:center; gap:10px; }.avatar { width:45px;height:45px;background:#202020;color:white;border-radius:50%;display:flex;align-items:center;justify-content:center;font-size:20px;text-transform:uppercase; }.card-superior h3 { font-size:18px; }.card-superior p { color:#777;font-size:14px; }.estado { margin-left:auto;font-size:12px;padding:6px;border-radius:5px; }.pagado { background:#d7f5df;color:#187a35; }.pendiente { background:#fff1cc;color:#a56c00; }.fiado { background:#ffdede;color:#a71919; }
.datos { margin-top:18px; display:grid; grid-template-columns:repeat(3,1fr); gap:12px; }.datos small { color:#888;font-size:10px;font-weight:bold; }.datos p { margin-top:4px;color:#444;font-size:14px; }.datos p span { color:#198754;font-weight:bold; }.descuento-card { margin-top:12px; padding:10px; background:#e8f8ed;color:#187a35;border-radius:7px;font-weight:bold; }
.calificacion { display:flex;align-items:center;gap:8px;margin-top:15px; }.estrellas-card { display:flex; }.estrella { padding:2px 4px;background:transparent;font-size:22px;color:#bbb; }.estrella.activa { color:#f0a500; }.fotos-card,.previsualizaciones { display:flex; gap:12px; margin-top:15px; }.fotos-card div,.previsualizaciones div { display:flex; flex-direction:column; gap:4px; }.fotos-card small,.previsualizaciones small { color:#777;font-size:10px;font-weight:bold; }.fotos-card img { width:95px;height:75px;object-fit:cover;border-radius:7px; }.observacion { margin-top:15px;padding-top:15px;border-top:1px solid #eee; }.titulo-observacion { display:flex;justify-content:space-between; }.btn-editar-nota { padding:3px 7px;background:#eee; }.texto-observacion { margin-top:8px;color:#555;font-size:14px; }.sin-observacion { color:#999; }.editar-observacion textarea,textarea { width:100%;min-height:70px;border:1px solid #ccd3d7;border-radius:7px;padding:10px;resize:vertical;font-family:Arial; }.botones-nota { display:flex;justify-content:flex-end;gap:8px;margin-top:7px; }.btn-cancelar-nota { background:#eee; }.btn-guardar-nota { background:#222;color:#fff; }.acciones { margin-top:15px;padding-top:15px;border-top:1px solid #eee;display:flex;gap:8px; }.btn-editar { background:#eef2f4;color:#333; }.btn-borrar { background:#d63031;color:white; }
.vacio { background:white;padding:50px;text-align:center;border-radius:12px;box-shadow:0 3px 10px #ddd; }.icono-vacio { font-size:50px; }.vacio h2 { margin:10px 0; }.vacio p { color:#777;margin-bottom:15px; }.vacio button { background:#ff6700;color:white; }
.fondo-modal { position:fixed;inset:0;background:rgba(0,0,0,.65);display:flex;align-items:center;justify-content:center;padding:20px;z-index:20;overflow:auto; }.ventana,.confirmar { background:white;border-radius:14px;width:min(700px,100%);max-height:94vh;overflow:auto;padding:25px; }.confirmar { width:min(430px,100%);text-align:center; }.cabecera-modal { display:flex;justify-content:space-between;gap:15px;align-items:flex-start;margin-bottom:20px; }.cabecera-modal h2 { font-size:27px; }.cabecera-modal p { color:#777;margin-top:5px; }.x { background:transparent;padding:5px; }.grupo { margin-bottom:15px; }.grupo label { display:block;font-weight:bold;color:#374151;margin-bottom:7px; }.grupo input,.grupo select { width:100%;padding:12px;border:1px solid #ccd3d7;border-radius:7px;background:white;outline:none; }.grupo input:focus,.grupo select:focus,textarea:focus { border-color:#ff6700; }.dos-columnas { display:grid;grid-template-columns:1fr 1fr;gap:15px; }.servicios-cuadricula { display:grid;grid-template-columns:1fr 1fr;gap:10px; }.cuadro-servicio { position:relative;background:white;border:2px solid #d5d5d5;border-radius:10px;padding:14px;min-height:80px;text-align:left;display:flex;flex-direction:column;align-items:flex-start;justify-content:center;gap:4px; }.cuadro-servicio.seleccionado { border-color:#ff6700;background:#fff1e8; }.cuadro-check { position:absolute;top:8px;right:8px;width:21px;height:21px;border:2px solid #aaa;border-radius:5px;display:flex;align-items:center;justify-content:center;font-size:13px; }.seleccionado .cuadro-check { background:#ff6700;border-color:#ff6700;color:#fff; }.nombre-servicio { font-weight:bold;padding-right:25px; }.precio-servicio { color:#777;font-size:13px; }.alerta-fidelidad { background:#fff3cd;border:1px solid #ffe08a;color:#765b00;padding:12px;border-radius:8px;margin-bottom:15px;font-weight:bold; }.total-descuento { background:#eaf7ee;color:#187a35;padding:10px;border-radius:7px;margin-bottom:15px;font-size:13px; }.mensaje-error { background:#ffe1e1;color:#a71919;padding:11px;border-radius:7px;margin-bottom:15px; }.mensaje-exito { background:#dff5e6;color:#187a35;padding:12px;border-radius:8px;margin-bottom:15px; }.fotos-inputs { display:flex;gap:10px; }.subir-foto { flex:1;background:#f1f4f5;border:1px dashed #9ba8ad;border-radius:8px;padding:12px;text-align:center;cursor:pointer; }.subir-foto input { display:none; }.ayuda { color:#888;display:block;margin-top:6px; }.previsualizaciones img { width:110px;height:85px;object-fit:cover;border-radius:7px; }.pie-modal { display:flex;justify-content:flex-end;gap:10px;margin-top:20px;padding-top:15px;border-top:1px solid #eee; }.cancelar { background:#eee;color:#333; }.guardar { background:#ff6700;color:#fff; }.alerta { font-size:45px;margin-bottom:10px; }.botones-confirmar { display:flex;justify-content:center;gap:10px;margin-top:20px; }
.caja-ventana { width:min(650px,100%); }.resumen-caja { display:grid;grid-template-columns:repeat(3,1fr);gap:10px; }.resumen-caja div { background:#f5f7f8;padding:15px;border-radius:9px;text-align:center; }.resumen-caja span,.resumen-caja small,.resumen-caja strong { display:block; }.resumen-caja span { font-size:25px; }.resumen-caja small { color:#777;margin:5px 0; }.aviso-caja { margin-top:15px;background:#fff3cd;padding:12px;border-radius:8px;color:#765b00; }.catalogo-ventana { width:min(800px,100%); }.catalogo-form { display:grid;grid-template-columns:1.4fr 1fr auto;gap:8px;margin-bottom:15px; }.catalogo-form input { padding:11px;border:1px solid #ccd3d7;border-radius:7px; }.catalogo-lista { display:flex;flex-direction:column;gap:8px; }.fila-catalogo { display:flex;justify-content:space-between;align-items:center;background:#f6f7f8;padding:10px;border-radius:8px; }.fila-catalogo > div:first-child { display:flex;flex-direction:column;gap:3px; }.fila-catalogo span { color:#777;font-size:13px; }.fila-catalogo button { padding:7px 10px;margin-left:5px; }
@media (max-width:900px) { .resumen { grid-template-columns:repeat(2,1fr); }.paneles-superiores { grid-template-columns:1fr; }.estadisticas { grid-template-columns:repeat(2,1fr); } }
@media (max-width:650px) { .inicio { flex-direction:column;align-items:flex-start;gap:20px; }.inicio h1 { font-size:36px; }.resumen { grid-template-columns:1fr; }.titulo-lista { flex-direction:column;align-items:flex-start; }.dos-columnas,.servicios-cuadricula,.datos,.resumen-caja,.catalogo-form { grid-template-columns:1fr; }.card-superior { align-items:flex-start; }.estado { margin-left:auto; }.acciones-lista { width:100%; }.acciones-lista button { flex:1; }.fotos-inputs { flex-direction:column; } }
</style>
