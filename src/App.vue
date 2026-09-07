<script setup>
import { ref } from 'vue'
import { useLocalStorage } from '@vueuse/core'

const servicios = useLocalStorage('barberiaRamiro', [])

const verFormulario = ref(false)//Controla si el formulario está visible.
const verEliminar = ref(false)//Controla si aparece la ventana de confirmación para eliminar.
const editando = ref(false)//Indica si estamos creando un servicio o editando uno existente.
const servicioSeleccionado = ref(null)//Guarda el ID del servicio seleccionado.

const nombre = ref('')
const servicio = ref('')
const barbero = ref('')
const hora = ref('')
const precio = ref('')
const pago = ref('')
const estado = ref('')
const nota = ref('')
const mensaje = ref('')//Sirve para almacenar mensajes de error.
const editandoNota = ref(null)//Guarda el ID del servicio cuya nota se está editando.
const notaTemporal = ref('')//Guarda temporalmente el texto de la nota mientras se está modificando.
//Se crean las variables del formrulario.


function nuevoServicio() {
  limpiar()
  editando.value = false
  verFormulario.value = true
}//Esta función se ejecuta cuando queremos crear un nuevo servicio.

function limpiar() {
  nombre.value = ''
  servicio.value = ''
  barbero.value = ''
  fecha.value = ''
  hora.value = ''
  precio.value = ''
  pago.value = ''
  estado.value = ''
  nota.value = ''
  mensaje.value = ''
  servicioSeleccionado.value = null
}

function cerrarFormulario() {
  verFormulario.value = false
  limpiar()
}

function cambiarServicio() {
  if (servicio.value === 'Corte clásico') {
    precio.value = 15000
  } else if (servicio.value === 'Corte moderno') {
    precio.value = 18000
  } else if (servicio.value === 'Barba') {
    precio.value = 10000
  } else if (servicio.value === 'Corte + barba') {
    precio.value = 25000
  } else if (servicio.value === 'Cejas') {
    precio.value = 5000
  } else if (servicio.value === 'Tinte') {
    precio.value = 30000
  } else {
    precio.value = ''
  }
}

function guardar() {
  mensaje.value = ''

  if (
    nombre.value === '' ||
    servicio.value === '' ||
    barbero.value === '' ||
    fecha.value === '' ||
    hora.value === '' ||
    precio.value === '' ||
    pago.value === '' ||
    estado.value === ''
  ) {
    mensaje.value = 'Debes completar todos los campos obligatorios'
    return
  }

  if (Number(precio.value) <= 0) {
    mensaje.value = 'El precio debe ser mayor a cero'
    return
  }

  if (editando.value === false) {
    servicios.value.push({
      id: Date.now(),
      nombre: nombre.value,
      servicio: servicio.value,
      barbero: barbero.value,
      fecha: fecha.value,
      hora: hora.value,
      precio: Number(precio.value),
      pago: pago.value,
      estado: estado.value,
      estrellas: 0,
      nota: nota.value
    })
  } else {
    for (let i = 0; i < servicios.value.length; i++) {
      if (servicios.value[i].id === servicioSeleccionado.value) {
        servicios.value[i].nombre = nombre.value
        servicios.value[i].servicio = servicio.value
        servicios.value[i].barbero = barbero.value
        servicios.value[i].fecha = fecha.value
        servicios.value[i].hora = hora.value
        servicios.value[i].precio = Number(precio.value)
        servicios.value[i].pago = pago.value
        servicios.value[i].estado = estado.value
        servicios.value[i].nota = nota.value
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
  servicio.value = item.servicio
  barbero.value = item.barbero
  fecha.value = item.fecha
  hora.value = item.hora || ''
  precio.value = item.precio
  pago.value = item.pago
  estado.value = item.estado
  nota.value = item.nota || ''
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

function dineroRecibido() {
  let total = 0

  for (let i = 0; i < servicios.value.length; i++) {
    if (servicios.value[i].estado === 'Pagado') {
      total = total + servicios.value[i].precio
    }
  }

  return total
}

function cantidadPendientes() {
  let cantidad = 0

  for (let i = 0; i < servicios.value.length; i++) {
    if (
      servicios.value[i].estado === 'Pendiente' ||
      servicios.value[i].estado === 'Fiado'
    ) {
      cantidad++
    }
  }

  return cantidad
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
</script>

<template>
  <main>

    <section class="inicio">

      <div>

        <p class="pequeno">
          SISTEMA DE REGISTRO
        </p>

        <h1>
          💈 Barbería Don Ramiro
        </h1>

        <p class="descripcion">
          Control de servicios, clientes y pagos.
        </p>

      </div>

      <button
        class="boton-principal"
        @click="nuevoServicio"
      >
        + Registrar servicio
      </button>

    </section>

    <section class="resumen">

      <div class="caja-resumen">

        <span>
          ✂️
        </span>

        <div>

          <small>
            Servicios
          </small>

          <h2>
            {{ servicios.length }}
          </h2>

        </div>

      </div>

      <div class="caja-resumen">

        <span>
          💰
        </span>

        <div>

          <small>
            Dinero recibido
          </small>

          <h2>
            ${{ dineroRecibido() }}
          </h2>

        </div>

      </div>

      <div class="caja-resumen">

        <span>
          ⚠️
        </span>

        <div>

          <small>
            Sin pagar
          </small>

          <h2>
            {{ cantidadPendientes() }}
          </h2>

        </div>

      </div>

    </section>

    <section class="contenido">

      <div class="titulo-lista">

        <h2>
          Registro de servicios
        </h2>

        <p>
          {{ servicios.length }} servicios guardados
        </p>

      </div>

      <div
        v-if="servicios.length === 0"
        class="vacio"
      >

        <div class="icono-vacio">
          😕
        </div>

        <h2>
          No hay servicios registrados
        </h2>

        <p>
          Comienza registrando el primer cliente del día.
        </p>

        <button @click="nuevoServicio">
          Registrar ahora
        </button>

      </div>

      <div
        v-else
        class="grid"
      >

        <article
          v-for="item in servicios"
          :key="item.id"
          class="servicio-card"
          :class="{
            sinPagar: item.estado === 'Pendiente',
            fiadoCard: item.estado === 'Fiado'
          }"
        >

          <div class="card-superior">

            <div class="avatar">
              {{ item.nombre.charAt(0) }}
            </div>

            <div>

              <h3>
                {{ item.nombre }}
              </h3>

              <p>
                {{ item.servicio }}
              </p>

            </div>

            <span
              v-if="item.estado === 'Pagado'"
              class="estado pagado"
            >
              ✓ Pagado
            </span>

            <span
              v-else-if="item.estado === 'Pendiente'"
              class="estado pendiente"
            >
              ⏳ Pendiente
            </span>

            <span
              v-else
              class="estado fiado"
            >
              ⚠ Fiado
            </span>

          </div>

          <div class="datos">

            <div>

              <small>
                BARBERO
              </small>

              <p>
                👤 {{ item.barbero }}
              </p>

            </div>

            <div>

              <small>
                PRECIO
              </small>

              <p>
                ${{ item.precio }}
              </p>

            </div>

            <div>

              <small>
                FECHA
              </small>

              <p>
                📅 {{ item.fecha }}
              </p>

            </div>

            <div>

              <small>
                HORA
              </small>

              <p>
                ⏳ {{ item.hora || 'No registrada' }}
              </p>

            </div>

            <div>

              <small>
                PAGO
              </small>

              <p v-if="item.pago === 'Efectivo'">
                💵 Efectivo
              </p>

              <p v-else-if="item.pago === 'Transferencia'">
                📲 Transferencia
              </p>

              <p v-else>
                💳 Tarjeta
              </p>

            </div>

          </div>

          <div
            class="calificacion"
            :class="{
              baja: (item.estrellas || 0) > 0 &&
                    (item.estrellas || 0) <= 2
            }"
          >

            <div class="estrellas-card">

              <button
                v-for="numero in 5"
                :key="numero"
                type="button"
                class="estrella"
                :class="{
                  activa: numero <= (item.estrellas || 0)
                }"
                @click="seleccionarEstrellas(item, numero)"
                :title="numero + ' estrella' + (numero > 1 ? 's' : '')"
              >
                {{
                  numero <= (item.estrellas || 0)
                    ? '★'
                    : '☆'
                }}
              </button>

            </div>

            <small>
              {{ item.estrellas || 0 }}/5
            </small>

          </div>

          <div class="observacion">

            <div class="titulo-observacion">

              <strong>
                📝 Observaciones
              </strong>

              <button
                v-if="editandoNota !== item.id"
                class="btn-editar-nota"
                type="button"
                @click="editarNota(item)"
              >
                ✏️
              </button>

            </div>

            <div
              v-if="editandoNota !== item.id"
              class="texto-observacion"
            >

              <span v-if="item.nota && item.nota.trim() !== ''">
                {{ item.nota }}
              </span>

              <span
                v-else
                class="sin-observacion"
              >
                Sin observaciones
              </span>

            </div>

            <div
              v-else
              class="editar-observacion"
            >

              <textarea
                v-model="notaTemporal"
                placeholder="Escriba una observación..."
              ></textarea>

              <div class="botones-nota">

                <button
                  type="button"
                  class="btn-cancelar-nota"
                  @click="cancelarNota"
                >
                  Cancelar
                </button>

                <button
                  type="button"
                  class="btn-guardar-nota"
                  @click="guardarNota(item)"
                >
                  💾 Guardar
                </button>

              </div>

            </div>

          </div>

          <div class="acciones">

            <button
              class="btn-editar"
              @click="editar(item)"
            >
              ✏️ Editar
            </button>

            <button
              class="btn-borrar"
              @click="confirmarEliminar(item.id)"
            >
              🗑
            </button>

          </div>

        </article>

      </div>

    </section>

    <div
      v-show="verFormulario"
      class="fondo-modal"
    >

      <div class="ventana">

        <div class="cabecera-modal">

          <div>

            <h2 v-if="editando === false">
              Nuevo servicio💈
            </h2>

            <h2 v-else>
              Editar servicio
            </h2>

            <p>
              Complete la información del cliente.
            </p>

          </div>

          <button
            class="x"
            @click="cerrarFormulario"
          >
            ❌
          </button>

        </div>

        <form @submit.prevent="guardar">

          <div
            v-if="mensaje !== ''"
            class="mensaje-error"
          >
            ⚠️ {{ mensaje }}
          </div>

          <div class="grupo">

            <label>
              Nombre del cliente😉
            </label>

            <input
              v-model="nombre"
              type="text"
              placeholder="Escriba el nombre"
            >

          </div>

          <div class="grupo">

            <label>
              Servicio🙌
            </label>

            <select
              v-model="servicio"
              @change="cambiarServicio"
            >

              <option value="">
                Seleccione un servicio
              </option>

              <option value="Corte clásico">
                Corte clásico
              </option>

              <option value="Corte moderno">
                Corte moderno
              </option>

              <option value="Barba">
                Barba
              </option>

              <option value="Corte + barba">
                Corte + barba
              </option>

              <option value="Cejas">
                Cejas
              </option>

              <option value="Tinte">
                Tinte
              </option>

            </select>

          </div>

          <div class="dos-columnas">

            <div class="grupo">

              <label>
                Barbero👦🏻
              </label>

              <select v-model="barbero">

                <option value="">
                  Seleccione
                </option>

                <option value="Don Ramiro">
                  Don Ramiro
                </option>

                <option value="Carlos">
                  Carlos
                </option>

                <option value="Miguel">
                  Miguel
                </option>

              </select>

            </div>

            <div class="grupo">

              <label>
                Precio💸
              </label>

              <input
                v-model="precio"
                type="number"
                placeholder="Seleccione un servicio"
                readonly
              >

            </div>

          </div>

          <div class="dos-columnas">

            <div class="grupo">

              <label>
                Fecha📅
              </label>

              <input
                v-model="fecha"
                type="date"
              >

            </div>

            <div class="grupo">

              <label>
                Hora⏳
              </label>

              <input
                v-model="hora"
                type="time"
              >

            </div>

          </div>

          <div class="dos-columnas">

            <div class="grupo">

              <label>
                Método de pago *
              </label>

              <select v-model="pago">

                <option value="">
                  Seleccione
                </option>

                <option value="Efectivo">
                  Efectivo
                </option>

                <option value="Transferencia">
                  Transferencia
                </option>

                <option value="Tarjeta">
                  Tarjeta
                </option>

              </select>

            </div>

            <div class="grupo">

              <label>
                Estado *
              </label>

              <select v-model="estado">

                <option value="">
                  Seleccione
                </option>

                <option value="Pagado">
                  Pagado
                </option>

                <option value="Pendiente">
                  Pendiente
                </option>

                <option value="Fiado">
                  Fiado
                </option>

              </select>

            </div>

          </div>

          <div class="pie-modal">

            <button
              type="button"
              class="cancelar"
              @click="cerrarFormulario"
            >
              Cancelar
            </button>

            <button
              type="submit"
              class="guardar"
            >

              <span v-if="editando === false">
                Guardar servicio
              </span>

              <span v-else>
                Actualizar servicio
              </span>

            </button>

          </div>

        </form>

      </div>

    </div>

    <div
      v-show="verEliminar"
      class="fondo-modal"
    >

      <div class="confirmar">

        <div class="alerta">
          🗑️
        </div>

        <h2>
          ¿Eliminar registro?
        </h2>

        <p>
          El servicio será eliminado y no podrás recuperarlo.
        </p>

        <div class="botones-confirmar">

          <button
            class="cancelar"
            @click="cancelarEliminar"
          >
            No, cancelar
          </button>

          <button
            class="btn-borrar"
            @click="eliminar"
          >
            Sí, eliminar
          </button>

        </div>

      </div>

    </div>

  </main>
</template>

<style>

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: Arial, sans-serif;
  background: #f1f3f5;
}

main {
  min-height: 100vh;
  background: #a2b7bd;
}

.inicio {
  background:
    linear-gradient(
      rgba(20, 20, 20, 0.70),
      rgba(20, 20, 20, 0.80)
    ),
    url('https://www.shutterstock.com/image-vector/barber-tools-haircut-icons-set-260nw-2619940863.jpg');

  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;

  color: white;

  padding: 45px 8%;

  min-height: 250px;

  display: flex;
  justify-content: space-between;
  align-items: center;
}

.pequeno {
  font-size: 12px;
  letter-spacing: 2px;
  color: #f0c45b;
  font-weight: bold;
}

h2{
  font-family:'Times New Roman', Times, serif;
  font-size: 30
  px;
}

.inicio h1 {
  margin: 8px 0;
  font-size: 50px;
  font-family:'Times New Roman', Times, serif;
}

.descripcion {
  color: #eeeeee;
}

.boton-principal {
  background: #ff6700;
  color: white;
  font-size: 15px;
}

.boton-principal:hover {
  background: #ffe600;
}

button {
  border: none;
  padding: 11px 18px;
  border-radius: 7px;
  cursor: pointer;
  font-weight: bold;
}

button:hover {
  background: #ffd900;
}

.resumen {
  max-width: 1200px;
  margin: 25px auto;
  padding: 0 20px;

  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 15px;
}

.caja-resumen {
  background: white;
  padding: 20px;
  border-radius: 12px;

  display: flex;
  gap: 15px;
  align-items: center;

  box-shadow: 0 2px 8px #ddd;
}

.caja-resumen span {
  font-size: 30px;
}

.caja-resumen small {
  color: #777;
}

.caja-resumen h2 {
  margin-top: 5px;
}

.contenido {
  max-width: 1200px;
  margin: auto;
  padding: 0 20px 40px;
}

.titulo-lista {
  margin-bottom: 20px;
}

.titulo-lista p {
  color: #777;
  margin-top: 5px;
}

.grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(320px, 1fr));
  gap: 20px;
}

.servicio-card {
  background: white;
  padding: 20px;
  border-radius: 12px;
  border-top: 5px solid #333;
  box-shadow: 0 3px 10px #ddd;
}

.sinPagar {
  border-top-color: #f0a500;
}

.fiadoCard {
  border-top-color: #d63031;
}

.card-superior {
  display: flex;
  align-items: center;
  gap: 10px;
}

.avatar {
  width: 45px;
  height: 45px;

  background: #202020;
  color: white;

  border-radius: 50%;

  display: flex;
  align-items: center;
  justify-content: center;

  font-size: 20px;
  text-transform: uppercase;
}

.card-superior h3 {
  font-size: 18px;
}

.card-superior p {
  color: #777;
  font-size: 14px;
}

.estado {
  margin-left: auto;
  font-size: 12px;
  padding: 6px;
  border-radius: 5px;
}

.pagado {
  background: #d7f5df;
  color: #187a35;
}

.pendiente {
  background: #fff1cc;
  color: #a56c00;
}

.fiado {
  background: #ffd9d9;
  color: #b00000;
}

.datos {
  margin-top: 20px;

  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 15px;
}

.datos small {
  color: #999;
  font-size: 10px;
}

.datos p {
  margin-top: 5px;
}

.calificacion {
  margin-top: 20px;

  background: #fff8d6;

  padding: 10px;

  border-radius: 7px;

  display: flex;
  justify-content: space-between;
  align-items: center;
}

.calificacion.baja {
  background: #ffdede;
}

.estrellas-card {
  display: flex;
  gap: 2px;
}

.estrella {
  background: transparent;
  color: #999;

  padding: 0;

  font-size: 22px;
  line-height: 1;
}

.estrella:hover {
  background: transparent;
  color: #d8a436;
}

.estrella.activa {
  color: #d8a436;
}

.observacion {
  margin-top: 12px;

  background: #f5f5f5;

  padding: 12px;

  border-radius: 7px;
}

.titulo-observacion {
  display: flex;
  justify-content: space-between;
  align-items: center;

  margin-bottom: 8px;
}

.titulo-observacion strong {
  font-size: 14px;
}

.btn-editar-nota {
  background: transparent;
  padding: 3px 6px;
  font-size: 15px;
}

.btn-editar-nota:hover {
  background: #ddd;
}

.texto-observacion {
  color: #555;
  font-size: 14px;
  line-height: 1.4;
}

.sin-observacion {
  color: #999;
  font-style: italic;
}

.editar-observacion textarea {
  width: 100%;
  min-height: 70px;

  padding: 10px;

  border: 1px solid #ccc;
  border-radius: 6px;

  resize: vertical;

  font-family: Arial, sans-serif;
}

.botones-nota {
  display: flex;
  justify-content: flex-end;
  gap: 8px;

  margin-top: 8px;
}

.btn-cancelar-nota {
  background: #ddd;
  color: #333;
  padding: 7px 10px;
}

.btn-guardar-nota {
  background: #202020;
  color: white;
  padding: 7px 10px;
}

.btn-guardar-nota:hover {
  background: #444;
}

.acciones {
  margin-top: 18px;

  display: flex;
  justify-content: space-between;
}

.btn-editar {
  background: #2980b9;
  color: white;
}

.btn-borrar {
  background: #d63031;
  color: white;
}

.vacio {
  background: white;
  text-align: center;

  padding: 60px 20px;

  border-radius: 12px;
}

.icono-vacio {
  font-size: 50px;
}

.vacio h2 {
  margin: 15px;
}

.vacio p {
  color: #777;
  margin-bottom: 20px;
}

.vacio button {
  background: #202020;
  color: white;
}

.fondo-modal {
  position: fixed;

  top: 0;
  left: 0;

  width: 100%;
  height: 100%;

  background: rgba(0, 0, 0, 0.65);

  display: flex;

  justify-content: center;
  align-items: center;

  padding: 20px;

  z-index: 10;
}

.ventana {
  background: white;

  width: 100%;
  max-width: 600px;

  padding: 25px;

  border-radius: 15px;

  max-height: 90vh;

  overflow-y: auto;
}

.cabecera-modal {
  display: flex;

  justify-content: space-between;

  margin-bottom: 20px;
}

.cabecera-modal p {
  color: #777;
  margin-top: 5px;
}

.x {
  background: transparent;
  color: #222;

  font-size: 30px;

  padding: 0;
}

.grupo {
  margin-bottom: 15px;
}

label {
  display: block;

  margin-bottom: 6px;

  font-weight: bold;
}

input,
select,
textarea {
  width: 100%;

  padding: 11px;

  border: 1px solid #ccc;

  border-radius: 7px;

  font-size: 14px;
}

input[readonly] {
  background: #eeeeee;
  color: #555;
  cursor: not-allowed;
}

textarea {
  height: 80px;
  resize: vertical;
}

.dos-columnas {
  display: grid;

  grid-template-columns: 1fr 1fr;

  gap: 15px;
}

.mensaje-error {
  background: #ffe1e1;

  color: #b00000;

  padding: 10px;

  margin-bottom: 15px;

  border-radius: 6px;
}

.pie-modal {
  display: flex;

  justify-content: flex-end;

  gap: 10px;

  margin-top: 20px;
}

.cancelar {
  background: #ddd;
  color: #333;
}

.guardar {
  background: #202020;
  color: white;
}

.confirmar {
  background: white;

  max-width: 400px;

  width: 100%;

  padding: 35px 25px;

  border-radius: 15px;

  text-align: center;
}

.alerta {
  font-size: 50px;

  margin-bottom: 15px;
}

.confirmar p {
  color: #666;

  margin: 15px 0 25px;
}

.botones-confirmar {
  display: flex;

  justify-content: center;

  gap: 10px;
}

@media (max-width: 700px) {

  .inicio {
    flex-direction: column;

    text-align: center;

    gap: 20px;

    min-height: 300px;

    padding: 40px 20px;
  }

  .inicio h1 {
    font-size: 28px;
  }

  .resumen {
    grid-template-columns: 1fr;
  }

  .dos-columnas {
    grid-template-columns: 1fr;
  }

}

</style>