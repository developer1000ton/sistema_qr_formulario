
<script setup>
import { ref, computed } from 'vue'
import Swal from 'sweetalert2'

// ==========================================
// 1. CONFIGURACIÓN INSTITUCIONAL
// ==========================================

const nombreSistema = 'Microsistema de Visitas USFA'
const nombreEvento = ref('Jornada de puertas abiertas')

// ==========================================
// 2. DATOS DE DEMOSTRACIÓN
// Posteriormente vendrán desde Flask y MySQL
// ==========================================

const carreras = ref([
  'Ingeniería de Sistemas',
  'Derecho',
  'Psicología',
  'Administración de Empresas',
  'Contaduría Pública',
  'Ingeniería Comercial',
])

const participantes = ref([
  {
    id: 1,
    nombre: 'María Fernández',
    colegio: 'Colegio Nacional',
    curso: '6to de secundaria',
    carrera: 'Ingeniería de Sistemas',
    fecha: '10/10/2026',
  },
  {
    id: 2,
    nombre: 'Carlos Mendoza',
    colegio: 'Unidad Educativa Central',
    curso: '5to de secundaria',
    carrera: 'Derecho',
    fecha: '10/10/2026',
  },
  {
    id: 3,
    nombre: 'Andrea Rojas',
    colegio: 'Colegio Nacional',
    curso: '6to de secundaria',
    carrera: 'Psicología',
    fecha: '10/10/2026',
  },
])

// ==========================================
// 3. ESTADO REACTIVO
// Vue actualizará la interfaz automáticamente
// ==========================================

const busqueda = ref('')
const filtroCarrera = ref('')
const mostrarFormulario = ref(false)

const nuevoParticipante = ref({
  nombre: '',
  colegio: '',
  curso: '',
  carrera: '',
})

// ==========================================
// 4. ESTADÍSTICAS CALCULADAS
// ==========================================

const totalParticipantes = computed(() => participantes.value.length)

const totalCarreras = computed(() => {
  return new Set(participantes.value.map(p => p.carrera)).size
})

const participantesFiltrados = computed(() => {
  const texto = busqueda.value.trim().toLowerCase()

  return participantes.value.filter(p => {
    const coincideTexto =
      p.nombre.toLowerCase().includes(texto) ||
      p.colegio.toLowerCase().includes(texto)

    const coincideCarrera =
      !filtroCarrera.value || p.carrera === filtroCarrera.value

    return coincideTexto && coincideCarrera
  })
})

const estadisticasCarreras = computed(() => {
  return carreras.value.map(carrera => ({
    nombre: carrera,
    cantidad: participantes.value.filter(
      p => p.carrera === carrera
    ).length,
  }))
})

// ==========================================
// 5. REGISTRAR PARTICIPANTE
// ==========================================

async function registrarParticipante() {
  const datos = nuevoParticipante.value

  if (
    !datos.nombre.trim() ||
    !datos.colegio.trim() ||
    !datos.curso ||
    !datos.carrera
  ) {
    await Swal.fire({
      icon: 'warning',
      title: 'Formulario incompleto',
      text: 'Completa todos los campos antes de registrar.',
      confirmButtonColor: '#004F98',
    })
    return
  }

  const confirmacion = await Swal.fire({
    icon: 'question',
    title: '¿Registrar participante?',
    text: `Se registrará a ${datos.nombre}.`,
    showCancelButton: true,
    confirmButtonText: 'Sí, registrar',
    cancelButtonText: 'Cancelar',
    confirmButtonColor: '#004F98',
    cancelButtonColor: '#64748b',
  })

  if (!confirmacion.isConfirmed) return

  participantes.value.unshift({
    id: Date.now(),
    nombre: datos.nombre.trim(),
    colegio: datos.colegio.trim(),
    curso: datos.curso,
    carrera: datos.carrera,
    fecha: new Date().toLocaleDateString('es-BO'),
  })

  nuevoParticipante.value = {
    nombre: '',
    colegio: '',
    curso: '',
    carrera: '',
  }

  mostrarFormulario.value = false

  await Swal.fire({
    icon: 'success',
    title: 'Registro exitoso',
    text: 'El participante se agregó a la lista de demostración.',
    confirmButtonColor: '#004F98',
  })
}

// ==========================================
// 6. ELIMINAR PARTICIPANTE
// ==========================================

async function eliminarParticipante(id) {
  const confirmacion = await Swal.fire({
    icon: 'warning',
    title: '¿Eliminar registro?',
    text: 'Esta acción quitará al participante de la lista local.',
    showCancelButton: true,
    confirmButtonText: 'Eliminar',
    cancelButtonText: 'Cancelar',
    confirmButtonColor: '#dc2626',
  })

  if (confirmacion.isConfirmed) {
    participantes.value = participantes.value.filter(
      p => p.id !== id
    )

    await Swal.fire({
      icon: 'success',
      title: 'Registro eliminado',
      confirmButtonColor: '#004F98',
    })
  }
}

// ==========================================
// 7. EXPORTAR DATOS A CSV
// ==========================================

function exportarCSV() {
  if (participantes.value.length === 0) {
    Swal.fire({
      icon: 'info',
      title: 'Sin datos',
      text: 'No hay participantes para exportar.',
      confirmButtonColor: '#004F98',
    })
    return
  }

  const columnas = ['Nombre', 'Colegio', 'Curso', 'Carrera', 'Fecha']

  const filas = participantes.value.map(p => [
    p.nombre,
    p.colegio,
    p.curso,
    p.carrera,
    p.fecha,
  ])

  const escapar = valor =>
    `"${String(valor).replace(/"/g, '""')}"`

  const contenido = [columnas, ...filas]
    .map(fila => fila.map(escapar).join(';'))
    .join('\r\n')

  const archivo = new Blob(
    ['\uFEFF' + contenido],
    { type: 'text/csv;charset=utf-8;' }
  )

  const enlace = document.createElement('a')
  enlace.href = URL.createObjectURL(archivo)
  enlace.download = 'participantes_usfa.csv'
  enlace.click()
  URL.revokeObjectURL(enlace.href)
}
</script>

<template>
  <div class="aplicacion">

    <!-- BARRA LATERAL -->
    <aside class="sidebar">
      <div class="marca">
        <div class="logo">USFA</div>
        <div>
          <strong>USFA</strong>
          <small>Gestión de visitas</small>
        </div>
      </div>

      <div class="menu-titulo">MENÚ PRINCIPAL</div>

      <nav class="menu">
        <a href="#resumen" class="menu-item activo">
          <span>▦</span> Resumen
        </a>

        <a href="#participantes" class="menu-item">
          <span>♙</span> Participantes
        </a>

        <a href="#carreras" class="menu-item">
          <span>▤</span> Carreras de interés
        </a>

        <a href="#registro" class="menu-item" @click="mostrarFormulario = true">
          <span>＋</span> Nuevo registro
        </a>
      </nav>

      <div class="sidebar-pie">
        <span class="indicador"></span>
        Sistema de demostración
      </div>
    </aside>

    <!-- CONTENIDO PRINCIPAL -->
    <main class="contenido">

      <header class="encabezado">
        <div>
          <span class="subtitulo">PANEL ADMINISTRATIVO</span>
          <h1>{{ nombreSistema }}</h1>
          <p>Control y seguimiento de visitantes universitarios.</p>
        </div>

        <div class="estado-evento">
          <span class="punto"></span>
          Evento activo
        </div>
      </header>

      <!-- INFORMACIÓN DEL EVENTO -->
      <section class="evento">
        <div>
          <span class="etiqueta">EVENTO ACTUAL</span>
          <h2>{{ nombreEvento }}</h2>
          <p>Registro y consulta de estudiantes visitantes.</p>
        </div>

        <div class="editar-evento">
          <label for="evento">Nombre del evento</label>
          <input
            id="evento"
            v-model="nombreEvento"
            type="text"
            placeholder="Nombre del evento"
          />
        </div>
      </section>

      <!-- ESTADÍSTICAS -->
      <section id="resumen" class="estadisticas">

        <article class="tarjeta estadistica">
          <div class="icono azul">♙</div>
          <div>
            <p>Total de participantes</p>
            <strong>{{ totalParticipantes }}</strong>
            <small>Registros en esta sesión</small>
          </div>
        </article>

        <article class="tarjeta estadistica">
          <div class="icono dorado">▤</div>
          <div>
            <p>Carreras seleccionadas</p>
            <strong>{{ totalCarreras }}</strong>
            <small>Con al menos un interesado</small>
          </div>
        </article>

        <article class="tarjeta estadistica">
          <div class="icono celeste">✓</div>
          <div>
            <p>Registros visibles</p>
            <strong>{{ participantesFiltrados.length }}</strong>
            <small>Según búsqueda y filtros</small>
          </div>
        </article>
      </section>

      <!-- GRÁFICO SIMPLIFICADO -->
      <section id="carreras" class="tarjeta seccion-carreras">
        <div class="titulo-seccion">
          <div>
            <h2>Interés por carrera</h2>
            <p>Distribución de las preferencias registradas.</p>
          </div>
        </div>

        <div class="barras">
          <div
            v-for="item in estadisticasCarreras"
            :key="item.nombre"
            class="barra-fila"
          >
            <span class="barra-nombre">{{ item.nombre }}</span>

            <div class="barra-fondo">
              <div
                class="barra-valor"
                :style="{
                  width: (totalParticipantes
                    ? item.cantidad / totalParticipantes * 100
                    : 0) + '%'
                }"
              ></div>
            </div>

            <strong class="barra-cantidad">{{ item.cantidad }}</strong>
          </div>
        </div>
      </section>

      <!-- PARTICIPANTES -->
      <section id="participantes" class="tarjeta seccion-participantes">
        <div class="titulo-seccion">
          <div>
            <h2>Participantes registrados</h2>
            <p>Consulta, filtra y administra los registros.</p>
          </div>

          <button class="boton boton-principal" @click="mostrarFormulario = !mostrarFormulario">
            {{ mostrarFormulario ? 'Cerrar formulario' : '+ Registrar' }}
          </button>
        </div>

        <!-- FORMULARIO -->
        <form
          v-if="mostrarFormulario"
          id="registro"
          class="formulario"
          @submit.prevent="registrarParticipante"
        >
          <h3>Nuevo participante</h3>

          <div class="campos">
            <div class="campo">
              <label for="nombre">Nombre completo</label>
              <input
                id="nombre"
                v-model="nuevoParticipante.nombre"
                type="text"
                placeholder="Ej. Juan Pérez"
                required
              />
            </div>

            <div class="campo">
              <label for="colegio">Unidad educativa</label>
              <input
                id="colegio"
                v-model="nuevoParticipante.colegio"
                type="text"
                placeholder="Nombre del colegio"
                required
              />
            </div>

            <div class="campo">
              <label for="curso">Curso</label>
              <select id="curso" v-model="nuevoParticipante.curso" required>
                <option value="" disabled>Seleccionar curso</option>
                <option>1ro de secundaria</option>
                <option>2do de secundaria</option>
                <option>3ro de secundaria</option>
                <option>4to de secundaria</option>
                <option>5to de secundaria</option>
                <option>6to de secundaria</option>
                <option>Otro</option>
              </select>
            </div>

            <div class="campo">
              <label for="carrera">Carrera de interés</label>
              <select
                id="carrera"
                v-model="nuevoParticipante.carrera"
                required
              >
                <option value="" disabled>Seleccionar carrera</option>
                <option
                  v-for="carrera in carreras"
                  :key="carrera"
                  :value="carrera"
                >
                  {{ carrera }}
                </option>
              </select>
            </div>
          </div>

          <div class="acciones-formulario">
            <button type="submit" class="boton boton-principal">
              Guardar participante
            </button>

            <button
              type="button"
              class="boton boton-secundario"
              @click="mostrarFormulario = false"
            >
              Cancelar
            </button>
          </div>
        </form>

        <!-- BUSCADOR Y FILTRO -->
        <div class="herramientas">
          <input
            v-model="busqueda"
            class="buscador"
            type="search"
            placeholder="Buscar por nombre o colegio..."
          />

          <select v-model="filtroCarrera" class="filtro">
            <option value="">Todas las carreras</option>
            <option
              v-for="carrera in carreras"
              :key="carrera"
              :value="carrera"
            >
              {{ carrera }}
            </option>
          </select>

          <button class="boton boton-exportar" @click="exportarCSV">
            ↓ Exportar CSV
          </button>
        </div>

        <!-- TABLA -->
        <div class="tabla-contenedor">
          <table>
            <thead>
              <tr>
                <th>Participante</th>
                <th>Unidad educativa</th>
                <th>Curso</th>
                <th>Carrera de interés</th>
                <th>Fecha</th>
                <th>Acción</th>
              </tr>
            </thead>

            <tbody>
              <tr
                v-for="persona in participantesFiltrados"
                :key="persona.id"
              >
                <td>
                  <div class="persona">
                    <div class="avatar">
                      {{ persona.nombre.charAt(0).toUpperCase() }}
                    </div>
                    <strong>{{ persona.nombre }}</strong>
                  </div>
                </td>

                <td>{{ persona.colegio }}</td>
                <td>{{ persona.curso }}</td>

                <td>
                  <span class="etiqueta-carrera">
                    {{ persona.carrera }}
                  </span>
                </td>

                <td>{{ persona.fecha }}</td>

                <td>
                  <button
                    class="boton-eliminar"
                    title="Eliminar participante"
                    @click="eliminarParticipante(persona.id)"
                  >
                    Eliminar
                  </button>
                </td>
              </tr>

              <tr v-if="participantesFiltrados.length === 0">
                <td colspan="6" class="sin-datos">
                  No se encontraron participantes.
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <div class="pie-tabla">
          Mostrando {{ participantesFiltrados.length }} de
          {{ totalParticipantes }} participantes
        </div>
      </section>

      <footer class="pie-pagina">
        <span>USFA · Microsistema de Visitas</span>
        <span>Versión inicial de desarrollo</span>
      </footer>
    </main>
  </div>
</template>

<style>
/* ==========================================
   COLORES INSTITUCIONALES USFA
========================================== */

:root {
  --usfa-azul: #06233b;
  --usfa-azul-oscuro: #003767;
  --usfa-dorado: #F3A81B;
  --usfa-blanco: #FFFFFF;
  --fondo: #F3F6FA;
  --borde: #E1E7EF;
  --texto: #263445;
  --texto-suave: #718096;
  font-family: Inter, "Segoe UI", Arial, sans-serif;
  color: var(--texto);
  background: var(--fondo);
  font-synthesis: none;
  text-rendering: optimizeLegibility;
}

* {
  box-sizing: border-box;
}

body {
  margin: 0;
  min-width: 320px;
  min-height: 100vh;
}

button,
input,
select {
  font: inherit;
}

button {
  cursor: pointer;
  transition: 0.2s ease;
}

/* ESTRUCTURA PRINCIPAL */

.aplicacion {
  min-height: 100vh;
}

.sidebar {
  position: fixed;
  inset: 0 auto 0 0;
  width: 250px;
  padding: 27px 17px;
  display: flex;
  flex-direction: column;
  background: var(--usfa-azul);
  color: white;
  z-index: 10;
}

.marca {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 0 8px 34px;
}

.logo {
  width: 48px;
  height: 48px;
  display: grid;
  place-items: center;
  border-radius: 13px;
  background: var(--usfa-dorado);
  color: var(--usfa-azul-oscuro);
  font-weight: 900;
  font-size: 15px;
}

.marca strong,
.marca small {
  display: block;
}

.marca strong {
  font-size: 18px;
  letter-spacing: 1px;
}

.marca small {
  margin-top: 4px;
  color: #d9e9fa;
  font-size: 12px;
}

.menu-titulo {
  padding: 10px;
  color: #bdd4ed;
  font-size: 10px;
  font-weight: 800;
  letter-spacing: 1.3px;
}

.menu {
  display: grid;
  gap: 8px;
}

.menu-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 13px 12px;
  border-radius: 9px;
  color: #e7f0fa;
  text-decoration: none;
  font-size: 13px;
  transition: 0.2s;
}

.menu-item span {
  width: 20px;
  text-align: center;
  font-size: 18px;
}

.menu-item:hover,
.menu-item.activo {
  background: rgba(255, 255, 255, 0.14);
  color: white;
}

.menu-item.activo {
  border-left: 3px solid var(--usfa-dorado);
}

.sidebar-pie {
  margin-top: auto;
  padding: 14px 5px;
  color: #d9e9fa;
  font-size: 11px;
  border-top: 1px solid rgba(255, 255, 255, 0.18);
}

.indicador,
.punto {
  display: inline-block;
  width: 8px;
  height: 8px;
  margin-right: 7px;
  border-radius: 50%;
  background: var(--usfa-dorado);
}

/* CONTENIDO */

.contenido {
  margin-left: 250px;
  padding: 35px;
  max-width: 1800px;
}

.encabezado {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 20px;
  margin-bottom: 28px;
}

.subtitulo,
.etiqueta {
  color: var(--usfa-azul);
  font-size: 10px;
  font-weight: 800;
  letter-spacing: 1.2px;
}

h1 {
  margin: 8px 0;
  font-size: clamp(23px, 2.5vw, 31px);
  letter-spacing: -0.7px;
}

.encabezado p,
.evento p,
.titulo-seccion p {
  margin: 0;
  color: var(--texto-suave);
  font-size: 13px;
}

.estado-evento {
  flex-shrink: 0;
  padding: 10px 13px;
  border: 1px solid #F2D48E;
  border-radius: 20px;
  background: #FFF5D9;
  color: #7B5400;
  font-size: 12px;
  font-weight: 700;
}

.estado-evento .punto {
  background: #D99A0B;
}

/* EVENTO */

.evento {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 20px;
  padding: 25px;
  margin-bottom: 22px;
  border-radius: 14px;
  background: var(--usfa-azul);
  color: white;
  border-bottom: 5px solid var(--usfa-dorado);
  box-shadow: 0 8px 25px rgba(0, 79, 152, 0.12);
}

.evento .etiqueta {
  color: var(--usfa-dorado);
}

.evento h2 {
  margin: 9px 0;
  font-size: 22px;
}

.evento p {
  color: #dfebf8;
}

.editar-evento {
  width: min(100%, 260px);
}

.editar-evento label,
.campo label {
  display: block;
  margin-bottom: 7px;
  font-size: 12px;
  font-weight: 700;
}

.editar-evento label {
  color: #e5eef8;
}

.editar-evento input {
  width: 100%;
  padding: 10px;
  border: 1px solid #7ba5ce;
  border-radius: 7px;
  outline: none;
}

/* TARJETAS */

.estadisticas {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 17px;
  margin-bottom: 22px;
}

.tarjeta {
  min-width: 0;
  padding: 23px;
  border: 1px solid var(--borde);
  border-radius: 13px;
  background: var(--usfa-blanco);
  box-shadow: 0 3px 12px rgba(26, 50, 80, 0.025);
}

.estadistica {
  display: flex;
  align-items: center;
  gap: 15px;
}

.icono {
  display: grid;
  place-items: center;
  flex-shrink: 0;
  width: 46px;
  height: 46px;
  border-radius: 12px;
  font-size: 24px;
}

.icono.azul {
  color: var(--usfa-azul);
  background: #E5F0FC;
}

.icono.dorado {
  color: #805500;
  background: #FFF0C8;
}

.icono.celeste {
  color: #006D7E;
  background: #DDF5F8;
}

.estadistica p {
  margin: 0 0 7px;
  color: var(--texto-suave);
  font-size: 12px;
}

.estadistica strong {
  display: block;
  margin-bottom: 5px;
  color: var(--usfa-azul);
  font-size: 30px;
  line-height: 1.1;
}

.estadistica small {
  color: #8995A5;
  font-size: 10px;
}

/* SECCIONES */

.seccion-carreras,
.seccion-participantes {
  margin-bottom: 22px;
}

.titulo-seccion {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 15px;
  margin-bottom: 23px;
}

.titulo-seccion h2 {
  margin: 0 0 7px;
  font-size: 17px;
}

.titulo-seccion p {
  line-height: 1.5;
}

/* BARRAS DINÁMICAS */

.barras {
  display: grid;
  gap: 17px;
}

.barra-fila {
  display: grid;
  grid-template-columns: minmax(110px, 190px) minmax(30px, 1fr) 25px;
  align-items: center;
  gap: 15px;
}

.barra-nombre {
  font-size: 12px;
}

.barra-fondo {
  height: 10px;
  overflow: hidden;
  border-radius: 20px;
  background: #EDF1F6;
}

.barra-valor {
  height: 100%;
  min-width: 0;
  border-radius: 20px;
  background: linear-gradient(90deg, var(--usfa-azul), #3689D0);
  transition: width 0.4s ease;
}

.barra-cantidad {
  color: var(--usfa-azul);
  text-align: right;
  font-size: 12px;
}

/* BOTONES */

.boton {
  padding: 11px 15px;
  border: 0;
  border-radius: 8px;
  font-size: 12px;
  font-weight: 700;
}

.boton:hover {
  transform: translateY(-1px);
  filter: brightness(1.06);
}

.boton-principal {
  background: var(--usfa-azul);
  color: white;
}

.boton-secundario {
  background: #EDF1F6;
  color: #475569;
}

.boton-exportar {
  background: #FFF0C8;
  color: #684600;
  border: 1px solid #F0D58E;
}

/* FORMULARIO */

.formulario {
  padding: 22px;
  margin-bottom: 24px;
  border: 1px solid #D7E5F5;
  border-radius: 12px;
  background: #F8FBFF;
}

.formulario h3 {
  margin: 0 0 20px;
  font-size: 15px;
}

.campos {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 18px;
}

.campo label {
  color: #40516A;
}

.campo input,
.campo select,
.buscador,
.filtro {
  width: 100%;
  min-width: 0;
  padding: 11px 12px;
  border: 1px solid #D6DEE8;
  border-radius: 8px;
  outline: none;
  background: white;
  color: var(--texto);
  font-size: 12px;
}

input:focus,
select:focus {
  border-color: var(--usfa-azul);
  box-shadow: 0 0 0 3px rgba(0, 79, 152, 0.1);
}

.acciones-formulario {
  display: flex;
  gap: 10px;
  margin-top: 20px;
}

/* BUSCADOR */

.herramientas {
  display: grid;
  grid-template-columns: minmax(150px, 1fr) minmax(160px, 220px) auto;
  gap: 12px;
  margin-bottom: 20px;
}

/* TABLA */

.tabla-contenedor {
  width: 100%;
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
  text-align: left;
  white-space: nowrap;
}

thead {
  background: #F5F8FC;
}

th {
  padding: 13px 12px;
  color: #63748A;
  font-size: 10px;
  font-weight: 800;
  text-transform: uppercase;
  letter-spacing: 0.4px;
}

td {
  padding: 15px 12px;
  border-bottom: 1px solid #EDF1F6;
  font-size: 11px;
}

tbody tr:hover {
  background: #FAFCFF;
}

.persona {
  display: flex;
  align-items: center;
  gap: 9px;
}

.persona strong {
  font-size: 11px;
}

.avatar {
  display: grid;
  place-items: center;
  flex-shrink: 0;
  width: 32px;
  height: 32px;
  border-radius: 10px;
  background: #E5F0FC;
  color: var(--usfa-azul);
  font-weight: 800;
}

.etiqueta-carrera {
  display: inline-block;
  max-width: 220px;
  padding: 6px 9px;
  overflow: hidden;
  border-radius: 6px;
  background: #EAF3FD;
  color: var(--usfa-azul);
  text-overflow: ellipsis;
  font-size: 10px;
}

.boton-eliminar {
  padding: 7px 9px;
  border: 1px solid #F4D1D1;
  border-radius: 6px;
  background: #FFF2F2;
  color: #B42323;
  font-size: 10px;
}

.boton-eliminar:hover {
  background: #FEE2E2;
}

.sin-datos {
  padding: 35px;
  color: var(--texto-suave);
  text-align: center;
}

.pie-tabla {
  padding-top: 17px;
  color: var(--texto-suave);
  font-size: 11px;
}

.pie-pagina {
  display: flex;
  justify-content: space-between;
  gap: 15px;
  padding: 15px 2px;
  color: #8793A3;
  font-size: 11px;
}

/* DISEÑO RESPONSIVO */

@media (max-width: 1100px) {
  .sidebar {
    width: 210px;
  }

  .contenido {
    margin-left: 210px;
    padding: 25px;
  }

  .estadisticas {
    grid-template-columns: 1fr;
  }

  .herramientas {
    grid-template-columns: 1fr 1fr;
  }

  .buscador {
    grid-column: 1 / -1;
  }
}

@media (max-width: 700px) {
  .sidebar {
    position: relative;
    width: 100%;
    padding: 16px;
  }

  .marca {
    padding-bottom: 15px;
  }

  .menu {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }

  .menu-titulo,
  .sidebar-pie {
    display: none;
  }

  .contenido {
    margin-left: 0;
    padding: 18px 12px;
  }

  .encabezado,
  .evento {
    flex-direction: column;
    align-items: stretch;
  }

  .estado-evento {
    align-self: flex-start;
  }

  .editar-evento {
    width: 100%;
  }

  .campos,
  .herramientas {
    grid-template-columns: 1fr;
  }

  .buscador {
    grid-column: auto;
  }

  .barra-fila {
    grid-template-columns: minmax(90px, 120px) minmax(25px, 1fr) 20px;
    gap: 8px;
  }

  .tarjeta {
    padding: 17px;
  }

  .titulo-seccion {
    align-items: flex-start;
    flex-wrap: wrap;
  }

  .pie-pagina {
    flex-direction: column;
  }
}
</style>
