# Documento de defensa de requerimientos funcionales

Proyecto: **Sistema Web de Gestión Inmobiliaria para Arriendo y Administración de Propiedades**

Este documento explica donde esta implementado cada requerimiento funcional (RF), en que parte de la pagina se evidencia, que hace y como funciona internamente. Esta pensado como apoyo para defender el codigo durante la evaluacion.

## 1. Vision general del proyecto

La entrega actual funciona como un **prototipo frontend en HTML, CSS y JavaScript** con datos de ejemplo. No hay todavia un servidor Express ni rutas API implementadas; el backend contiene modelos Mongoose que sirven como base para MongoDB.

### Estructura principal

| Carpeta / archivo | Funcion |
| --- | --- |
| `frontend/index.html` | Pagina de inicio con presentacion del servicio y acceso al catalogo. |
| `frontend/pages/propiedades.html` | Catalogo de propiedades con filtros. |
| `frontend/pages/detalle-propiedad.html` | Ficha de una propiedad y formulario para solicitar visita. |
| `frontend/pages/login.html` | Inicio de sesion por correo/RUT, clave y perfil. |
| `frontend/pages/registro.html` | Registro de clientes o propietarios. |
| `frontend/pages/dashboard.html` | Panel posterior al login, cambia segun perfil. |
| `frontend/js/datos.js` | Datos de ejemplo: propiedades, usuarios, visitas y funciones de sesion/localStorage. |
| `frontend/js/catalogo.js` | Renderiza las tarjetas del catalogo y aplica filtros. |
| `frontend/js/detalle.js` | Carga la informacion de una propiedad en la ficha de detalle. |
| `frontend/js/formularios.js` | Valida registro y solicitud de visita; guarda datos en localStorage. |
| `frontend/js/login.js` | Valida credenciales y crea la sesion del usuario. |
| `frontend/js/dashboard.js` | Construye el panel de cliente, propietario, ejecutivo o administrador. |
| `frontend/js/validacion.js` | Funciones reutilizables para validar RUT, correo, campos y mensajes. |
| `frontend/css/styles.css` | Estilos visuales, grillas, formularios, tablas, responsive y estados. |
| `backend/models/Usuario.js` | Modelo MongoDB para usuarios y roles. |
| `backend/models/Propiedad.js` | Modelo MongoDB para propiedades, propietario asociado y estados. |

### Como se almacenan los datos en esta entrega

- Las propiedades, usuarios demo y una visita inicial estan definidos en `frontend/js/datos.js`.
- Los registros nuevos se guardan en `localStorage` con la clave `gestionUrbanaRegistros`.
- Las visitas solicitadas desde la ficha de propiedad se guardan en `localStorage` con la clave `gestionUrbanaVisitas`.
- La sesion activa se guarda en `sessionStorage` con la clave `gestionUrbanaSesion`.
- Esto permite demostrar flujos en el navegador, pero no reemplaza una API real ni persistencia en MongoDB.

## 2. Mapeo RF por RF

### RF1 - Registro, autenticacion y administracion de usuarios segun perfiles

**Prioridad:** Alta<br>
**Estado en el prototipo:** Implementado parcialmente en frontend.

**En que parte de la pagina se ve**

- `pages/registro.html`: formulario "Crear cuenta" para cliente o propietario.
- `pages/login.html`: formulario "Iniciar Sesion" con selector de perfil.
- `pages/dashboard.html`: "Mi Panel", que cambia el contenido segun el perfil autenticado.

**Donde esta en el codigo**

- `frontend/pages/registro.html`: campos de nombre, RUT, correo, telefono, clave, confirmacion y tipo de cuenta.
- `frontend/pages/login.html`: campos de usuario/RUT/correo, password y perfil.
- `frontend/js/formularios.js`: funcion `iniciarRegistro()`, valida el formulario y guarda usuarios nuevos.
- `frontend/js/login.js`: valida credenciales y perfil; si son correctos llama a `guardarSesion()`.
- `frontend/js/dashboard.js`: decide que panel mostrar segun `sesion.perfil`.
- `frontend/js/datos.js`: arreglo `DATOS.usuarios`, constante `NOMBRES_PERFIL`, funciones `guardarSesion()`, `obtenerSesion()`, `cerrarSesion()`, `buscarUsuario()` y `guardarRegistro()`.
- `backend/models/Usuario.js`: modelo de usuario con `rol` permitido (`admin`, `ejecutivo`, `propietario`, `cliente`) y `estado`.

**Que hace**

- Permite registrar cuentas nuevas de tipo cliente o propietario.
- Permite iniciar sesion usando correo o RUT, clave y perfil.
- Evita que el usuario entre si la clave no corresponde o si selecciona un perfil distinto al de la cuenta.
- Muestra un panel distinto para cliente, propietario, ejecutivo o administrador.

**Como funciona**

1. El registro valida nombre, RUT chileno, correo, telefono, password y confirmacion.
2. Antes de guardar, revisa duplicidad con `buscarUsuario(email)` y `buscarUsuario(rut)`.
3. Los usuarios nuevos quedan en `localStorage`.
4. En login, `buscarUsuario()` revisa usuarios demo + usuarios registrados.
5. Si la clave y el perfil coinciden, `guardarSesion()` guarda la sesion en `sessionStorage`.
6. `dashboard.js` lee la sesion y ejecuta:
   - `renderCliente()`
   - `renderPropietario()`
   - `renderEjecutivo()`
   - `renderAdmin()`

**Limitacion importante para defender**

La autenticacion todavia es simulada en el navegador. El modelo `backend/models/Usuario.js` existe, pero no hay controlador, rutas Express, hashing de passwords ni JWT/sesion segura implementados.

---

### RF2 - Registrar y administrar propietarios asociados a propiedades

**Prioridad:** Alta<br>
**Estado en el prototipo:** Parcial.

**En que parte de la pagina se ve**

- `pages/registro.html`: permite registrar un usuario con perfil propietario.
- `pages/dashboard.html`: al iniciar sesion como propietario, muestra "Estado de mis inmuebles".

**Donde esta en el codigo**

- `frontend/pages/registro.html`: selector `registro-perfil` permite `propietario`.
- `frontend/js/datos.js`: usuario demo `u-propietario` y propiedades con `propietarioId`.
- `frontend/js/dashboard.js`: funcion `renderPropietario()`, filtra propiedades y muestra tabla.
- `backend/models/Propiedad.js`: campo `propietario` como referencia a `Usuario`.
- `backend/models/Usuario.js`: rol `propietario`.

**Que hace**

- Permite demostrar que una propiedad tiene un propietario asociado.
- Permite que el propietario vea sus propiedades y sus estados desde el panel.

**Como funciona**

1. En los datos de ejemplo, cada propiedad tiene `propietarioId`.
2. El usuario demo Luis Soto tiene id `u-propietario`.
3. Al entrar al panel con perfil propietario, `renderPropietario()` arma una lista de propiedades relacionadas y llama a `tablaPropiedades()`.
4. En backend, el modelo `Propiedad` ya esta preparado para relacionar una propiedad con un usuario mediante `ref: 'Usuario'`.

**Limitacion importante para defender**

No existe aun una pantalla CRUD completa para crear, editar, activar/inactivar propietarios. La asociacion esta representada en datos demo y modelo, pero no con API ni formulario administrativo real.

---

### RF3 - Registrar y administrar propiedades con datos completos

**Prioridad:** Alta<br>
**Estado en el prototipo:** Parcial.

**En que parte de la pagina se ve**

- `pages/propiedades.html`: tarjetas del catalogo.
- `pages/detalle-propiedad.html`: ficha completa de una propiedad.
- `pages/dashboard.html`: tabla de propiedades para propietario, ejecutivo y administrador.

**Donde esta en el codigo**

- `frontend/js/datos.js`: arreglo `DATOS.propiedades`, con tipo, direccion, comuna, precio, gastos comunes, dormitorios, banos, estacionamientos, superficie, estado, propietarioId, imagen, descripcion y caracteristicas.
- `frontend/js/catalogo.js`: funcion `renderTarjetas()`, crea las tarjetas desde `DATOS.propiedades`.
- `frontend/js/detalle.js`: carga una propiedad por id y completa la ficha.
- `frontend/js/dashboard.js`: `tablaPropiedades()` muestra propiedades en tabla.
- `backend/models/Propiedad.js`: define el esquema con `titulo`, `tipo`, `propietario`, `comuna`, `direccion`, `superficie`, `dormitorios`, `banos`, `estacionamientos`, `precioMensual`, `gastosComunes`, `caracteristicas`, `imagenes` y `estado`.

**Que hace**

- Muestra propiedades de distintos tipos: departamento, casa, oficina y local comercial.
- Expone sus datos comerciales y tecnicos.
- Permite navegar desde catalogo hacia una ficha detallada.

**Como funciona**

1. `catalogo.js` recorre `DATOS.propiedades`.
2. Por cada propiedad crea un `<article>` con imagen, estado, tipo, titulo, comuna, datos tecnicos, precio y enlace "Ver Detalle".
3. El enlace manda a `detalle-propiedad.html?id=ID`.
4. `detalle.js` lee el parametro `id` con `URLSearchParams`.
5. Busca la propiedad usando `obtenerPropiedadPorId()`.
6. Completa imagen, estado, ubicacion, descripcion, gastos comunes, superficie y caracteristicas.

**Limitacion importante para defender**

La administracion de propiedades no permite crear/editar/eliminar desde la interfaz. Hoy las propiedades se administran modificando `DATOS.propiedades` en codigo y estan modeladas para futura persistencia en MongoDB.

---

### RF4 - Controlar estados de una propiedad y transiciones

**Prioridad:** Alta<br>
**Estado en el prototipo:** Parcial.

**En que parte de la pagina se ve**

- `pages/propiedades.html`: cada tarjeta muestra una insignia de estado.
- `pages/detalle-propiedad.html`: muestra el estado en la ficha.
- Si la propiedad esta reservada, el formulario de visita se oculta y aparece un aviso.
- `pages/dashboard.html`: las tablas muestran el estado de cada propiedad.

**Donde esta en el codigo**

- `frontend/js/datos.js`: campo `estado` en cada propiedad (`Disponible`, `Reservada`, etc.).
- `frontend/js/catalogo.js`: pinta la insignia con `propiedad.estado`.
- `frontend/js/detalle.js`: valida `const disponible = propiedad.estado === "Disponible"` y oculta el formulario si no esta disponible.
- `frontend/js/dashboard.js`: `contarPorEstado()` calcula estadisticas por estado.
- `frontend/css/styles.css`: estilos `.insignia-estado.disponible` y `.insignia-estado.reservada`.
- `backend/models/Propiedad.js`: enum de estados permitidos: `Disponible`, `Publicada`, `Reservada`, `Arrendada`, `En mantencion`, `Inactiva`.

**Que hace**

- Permite ver el estado actual de una propiedad.
- Impide en frontend solicitar visita para una propiedad que no esta disponible.
- Muestra conteos de propiedades disponibles o reservadas en el panel.

**Como funciona**

1. El estado viene desde `DATOS.propiedades`.
2. `catalogo.js` lo usa para crear una etiqueta visible en la tarjeta.
3. `detalle.js` revisa si el estado es exactamente `Disponible`.
4. Si no esta disponible, oculta `form-visita` y muestra `aviso-reserva`.

**Limitacion importante para defender**

No existen todavia reglas de transicion completas entre todos los estados ni validacion en backend. El control actual es visual y de bloqueo basico del formulario de visita.

---

### RF5 - Publicar propiedades disponibles y consultar catalogo con filtros

**Prioridad:** Alta<br>
**Estado en el prototipo:** Implementado en frontend.

**En que parte de la pagina se ve**

- `pages/propiedades.html`: catalogo "Explora nuestras propiedades disponibles".
- Barra de filtros por tipo, comuna, precio maximo, dormitorios y banos.
- Botones "Buscar" y "Limpiar".
- Contador de resultados.

**Donde esta en el codigo**

- `frontend/pages/propiedades.html`: formulario `form-filtros`.
- `frontend/js/catalogo.js`: render de tarjetas y funcion `aplicarFiltros()`.
- `frontend/js/datos.js`: datos usados por el catalogo.

**Que hace**

- Publica las propiedades disponibles en tarjetas.
- Permite consultar el catalogo filtrando por:
  - tipo,
  - comuna,
  - precio maximo,
  - dormitorios minimos,
  - banos minimos.

**Como funciona**

1. `renderTarjetas()` crea una tarjeta por propiedad.
2. Cada tarjeta guarda datos de filtro en `dataset`: tipo, comuna, precio, dormitorios y banos.
3. `aplicarFiltros()` compara los valores del formulario con los `dataset`.
4. Si una tarjeta no coincide, agrega la clase `oculta`.
5. Actualiza el mensaje "Mostrando X de Y propiedades".
6. El boton "Limpiar" resetea el formulario y vuelve a mostrar todos los resultados.

**Punto fuerte para defender**

Este RF esta bien evidenciado en interfaz y JavaScript: se puede demostrar en vivo cambiando filtros y viendo como se actualizan las tarjetas sin recargar la pagina.

---

### RF6 - Registrar clientes interesados evitando duplicidad

**Prioridad:** Alta<br>
**Estado en el prototipo:** Parcial.

**En que parte de la pagina se ve**

- `pages/registro.html`: registro de cliente.
- `pages/detalle-propiedad.html`: formulario de solicitud de visita con nombre y correo/RUT.

**Donde esta en el codigo**

- `frontend/js/formularios.js`: `iniciarRegistro()` valida y guarda nuevos clientes/propietarios.
- `frontend/js/datos.js`: `buscarUsuario()` normaliza correo/RUT y busca coincidencias.
- `frontend/js/validacion.js`: `validarEmail()`, `validarRut()` y `validarUsuario()`.

**Que hace**

- Permite registrar clientes.
- Evita duplicar una cuenta si ya existe el correo o RUT.
- En solicitud de visita exige contacto valido como correo o RUT.

**Como funciona**

1. El usuario llena el formulario de registro.
2. El sistema valida formato de RUT y correo.
3. `buscarUsuario()` normaliza los identificadores quitando puntos y guiones y pasando a minuscula.
4. Si ya existe correo o RUT, muestra el mensaje "Ya existe una cuenta con ese correo o RUT."
5. Si no existe, guarda el registro en `localStorage`.

**Limitacion importante para defender**

La duplicidad se controla para cuentas registradas, pero no existe aun un modulo completo de "clientes interesados" administrado por ejecutivo ni una coleccion backend de clientes.

---

### RF7 - Solicitar visitas y permitir al ejecutivo confirmar/modificar/rechazar/cancelar

**Prioridad:** Alta<br>
**Estado en el prototipo:** Parcial.

**En que parte de la pagina se ve**

- `pages/detalle-propiedad.html`: formulario "Solicitar visita".
- `pages/dashboard.html`: panel ejecutivo muestra "Agenda de visitas".

**Donde esta en el codigo**

- `frontend/pages/detalle-propiedad.html`: formulario `form-visita` con nombre, contacto, fecha, hora y observaciones.
- `frontend/js/formularios.js`: funcion `iniciarVisita()`, valida y guarda la visita.
- `frontend/js/datos.js`: `obtenerVisitas()` y `guardarVisita()`.
- `frontend/js/dashboard.js`: `renderEjecutivo()` y `listaVisitas()` muestran las visitas.

**Que hace**

- Un cliente puede pedir una visita indicando fecha, hora y observaciones.
- La visita queda con estado inicial `Pendiente`.
- El ejecutivo puede ver las solicitudes en su panel.

**Como funciona**

1. El cliente entra al detalle de una propiedad disponible.
2. Completa el formulario de visita.
3. `iniciarVisita()` valida:
   - nombre minimo,
   - correo o RUT valido,
   - fecha obligatoria y no pasada,
   - hora obligatoria entre 09:00 y 18:00.
4. Si todo es valido, `guardarVisita()` guarda la visita en `localStorage`.
5. El panel ejecutivo recupera las visitas con `obtenerVisitas()` y las lista.

**Limitacion importante para defender**

La parte de confirmar, modificar, rechazar o cancelar visitas no esta implementada con botones ni cambio de estado. Actualmente el ejecutivo solo visualiza la agenda.

---

### RF8 - Gestionar visitas evitando superposiciones y registrando resultado

**Prioridad:** Alta<br>
**Estado en el prototipo:** Pendiente / muy parcial.

**En que parte de la pagina se ve**

- `pages/dashboard.html`: panel ejecutivo muestra "Agenda de visitas".

**Donde esta en el codigo**

- `frontend/js/dashboard.js`: `renderEjecutivo()` lista visitas.
- `frontend/js/formularios.js`: `iniciarVisita()` crea visitas pendientes.
- `frontend/js/datos.js`: arreglo inicial `DATOS.visitas` y funcion `obtenerVisitas()`.

**Que hace hoy**

- Permite visualizar las visitas registradas.
- Guarda fecha, hora, estado y observaciones.

**Que falta para cumplir completamente**

- Validar que un ejecutivo no tenga dos visitas en la misma fecha y hora.
- Asignar visitas a un ejecutivo especifico.
- Registrar resultado de la visita, por ejemplo: realizada, no asistio, interesado, no interesado.
- Incorporar acciones de gestion desde el panel.

**Como deberia funcionar en una version completa**

1. Al solicitar o confirmar visita, el backend revisaria la agenda del ejecutivo.
2. Si ya existe una visita activa en el mismo bloque horario, rechazaria la operacion.
3. Una vez realizada, el ejecutivo registraria resultado y observaciones.

**Limitacion importante para defender**

Este RF no esta resuelto como regla de negocio. Solo existe la base visual y de datos para listar visitas.

---

### RF9 - Generar solicitud formal de arriendo para una propiedad

**Prioridad:** Alta<br>
**Estado en el prototipo:** Pendiente.

**En que parte de la pagina se ve**

- No hay pagina o formulario especifico de solicitud formal de arriendo.
- La pagina de detalle solo contiene solicitud de visita.

**Donde podria integrarse**

- `frontend/pages/detalle-propiedad.html`: podria agregarse un segundo formulario "Solicitar arriendo".
- `frontend/js/formularios.js`: podria agregarse `iniciarSolicitudArriendo()`.
- Backend futuro: modelo `SolicitudArriendo` o similar.

**Que deberia hacer**

- Permitir que el cliente presente antecedentes para evaluacion.
- Guardar la solicitud con estado inicial `Pendiente` o `En evaluacion`.
- Relacionarla con cliente y propiedad.

**Situacion actual para defender**

El flujo implementado llega hasta la solicitud de visita. La solicitud formal de arriendo todavia no esta implementada.

---

### RF10 - Evaluar solicitudes de arriendo y mantener trazabilidad

**Prioridad:** Alta<br>
**Estado en el prototipo:** Pendiente.

**En que parte de la pagina se ve**

- No hay interfaz de evaluacion de solicitudes de arriendo.

**Donde podria integrarse**

- `frontend/pages/dashboard.html`: dentro del panel ejecutivo.
- `frontend/js/dashboard.js`: una nueva seccion para solicitudes de arriendo.
- Backend futuro: endpoints para cambiar estado y guardar observaciones.

**Que deberia hacer**

- Mostrar solicitudes pendientes o en evaluacion.
- Permitir aprobar, rechazar o cancelar.
- Guardar observaciones del ejecutivo.
- Mantener historial de cambios.

**Situacion actual para defender**

No existe aun modulo de solicitudes de arriendo. El panel ejecutivo actualmente muestra visitas y cartera de propiedades.

---

### RF11 - Reservar propiedad con solicitud aprobada y evitar multiples aprobaciones incompatibles

**Prioridad:** Alta<br>
**Estado en el prototipo:** Parcial / pendiente como flujo completo.

**En que parte de la pagina se ve**

- `pages/propiedades.html`: se ve una propiedad en estado `Reservada`.
- `pages/detalle-propiedad.html`: si una propiedad esta reservada, se oculta el formulario de visita.

**Donde esta en el codigo**

- `frontend/js/datos.js`: propiedad "Local Comercial a Pie de Calle" con `estado: "Reservada"`.
- `frontend/js/detalle.js`: si `propiedad.estado !== "Disponible"`, oculta el formulario.
- `frontend/css/styles.css`: `.insignia-estado.reservada`.
- `backend/models/Propiedad.js`: enum incluye estado `Reservada`.

**Que hace hoy**

- Permite visualizar una propiedad reservada.
- Bloquea nuevas solicitudes de visita en la ficha de detalle cuando no esta disponible.

**Que falta para cumplir completamente**

- Crear solicitud de arriendo.
- Aprobar solicitud.
- Cambiar automaticamente la propiedad a `Reservada`.
- Evitar que dos solicitudes queden aprobadas para la misma propiedad.
- Liberar reserva si el proceso no se concreta.

**Situacion actual para defender**

El estado `Reservada` esta representado y afecta la interfaz, pero no existe el flujo de aprobacion/reserva completo ni control de concurrencia en backend.

---

### RF12 - Consultas de gestion sobre propiedades, visitas, solicitudes, pagos y requerimientos

**Prioridad:** Media<br>
**Estado en el prototipo:** Parcial.

**En que parte de la pagina se ve**

- `pages/dashboard.html`: panel con indicadores y tablas segun perfil.

**Donde esta en el codigo**

- `frontend/js/dashboard.js`:
  - `renderCliente()`: muestra visitas solicitadas y propiedades publicadas.
  - `renderPropietario()`: muestra mis propiedades, disponibles y reservadas.
  - `renderEjecutivo()`: muestra visitas pendientes, propiedades y disponibles.
  - `renderAdmin()`: muestra usuarios, propiedades y visitas.
  - `tablaPropiedades()`: tabla de propiedades.
  - `listaVisitas()`: lista de visitas.
  - `contarPorEstado()`: conteos por estado.

**Que hace**

- Entrega consultas resumidas en el panel:
  - cantidad de usuarios,
  - cantidad de propiedades,
  - cantidad de visitas,
  - visitas pendientes,
  - propiedades disponibles,
  - propiedades reservadas,
  - tabla de propiedades administradas.

**Como funciona**

1. Al entrar al dashboard, se lee la sesion.
2. Segun el perfil, se arma HTML dinamico con datos de `DATOS` y `localStorage`.
3. Las tarjetas de datos se crean con `tarjetaDato()`.
4. Las tablas y listas se crean con `tablaPropiedades()` y `listaVisitas()`.

**Limitacion importante para defender**

No hay consultas de pagos, requerimientos pendientes, contratos proximos a vencer ni solicitudes de arriendo porque esos modulos aun no existen.

---

## 3. Resumen de cobertura

| RF | Cobertura actual | Evidencia principal |
| --- | --- | --- |
| RF1 | Parcial | Login, registro, sesion y dashboard por perfil. |
| RF2 | Parcial | Propietario demo, asociacion `propietarioId`, panel propietario y modelo backend. |
| RF3 | Parcial | Datos completos de propiedades, catalogo, detalle y modelo backend. |
| RF4 | Parcial | Estados visibles y bloqueo de visita si no esta disponible. |
| RF5 | Implementado en frontend | Catalogo con filtros funcionales. |
| RF6 | Parcial | Registro con validacion y control de duplicidad por correo/RUT. |
| RF7 | Parcial | Solicitud de visita y visualizacion por ejecutivo. |
| RF8 | Muy parcial | Lista de visitas; falta gestion real, superposiciones y resultado. |
| RF9 | Pendiente | No hay solicitud formal de arriendo. |
| RF10 | Pendiente | No hay evaluacion de solicitudes de arriendo. |
| RF11 | Parcial | Estado reservada visible; falta flujo de aprobacion/reserva. |
| RF12 | Parcial | Dashboard con indicadores basicos; faltan pagos/requerimientos/solicitudes. |

## 4. Guia rapida para defender en la pagina

### Flujo 1: catalogo y filtros (RF5)

1. Abrir `frontend/pages/propiedades.html`.
2. Mostrar las tarjetas de propiedades.
3. Filtrar por tipo, comuna, precio, dormitorios o banos.
4. Explicar que el filtro se aplica en `catalogo.js` usando los `dataset` de cada tarjeta.

### Flujo 2: detalle y solicitud de visita (RF3, RF4, RF7)

1. Desde el catalogo, hacer clic en "Ver Detalle".
2. Mostrar imagen, estado, ubicacion, descripcion, gastos comunes, superficie y caracteristicas.
3. Completar el formulario de visita.
4. Explicar que `formularios.js` valida fecha, hora y contacto.
5. Explicar que la visita queda en `localStorage` con estado `Pendiente`.

### Flujo 3: propiedad reservada (RF4, RF11 parcial)

1. Abrir el local comercial reservado desde el catalogo.
2. Mostrar que aparece el aviso de reserva.
3. Explicar que `detalle.js` oculta el formulario cuando `estado !== "Disponible"`.

### Flujo 4: registro y login (RF1, RF6)

1. Abrir `frontend/pages/registro.html`.
2. Mostrar validaciones de RUT, correo, telefono y password.
3. Registrar una cuenta cliente o propietario.
4. Intentar duplicar correo/RUT para mostrar el control de duplicidad.
5. Iniciar sesion desde `frontend/pages/login.html`.
6. Explicar que la sesion se guarda en `sessionStorage`.

### Flujo 5: panel por perfil (RF1, RF2, RF12)

Usar las cuentas demo de `login.html`, todas con clave `123456`:

| Perfil | Usuario |
| --- | --- |
| Cliente | `ana@correo.cl` |
| Propietario | `luis@correo.cl` |
| Ejecutivo | `elena@gestionurbana.cl` |
| Administrador | `admin@gestionurbana.cl` |

Mostrar que el dashboard cambia:

- Cliente: ve sus visitas.
- Propietario: ve estado de sus inmuebles.
- Ejecutivo: ve agenda de visitas y cartera.
- Administrador: ve usuarios y propiedades administradas.

## 5. Argumento tecnico final

La entrega demuestra la estructura base de un sistema inmobiliario con navegacion, catalogo, filtros, formularios validados, sesion simulada, roles, paneles diferenciados, estados de propiedad y modelos iniciales para MongoDB. Los RF mas evidentes en la interfaz son RF1, RF3, RF4, RF5, RF6, RF7 y RF12, aunque varios estan en nivel parcial porque aun no existe API backend.

Para una siguiente iteracion, las prioridades tecnicas deberian ser:

1. Crear servidor Node.js/Express.
2. Conectar MongoDB usando los modelos `Usuario` y `Propiedad`.
3. Agregar hashing de contrasenas y autenticacion real.
4. Crear modelos y rutas para propietarios, visitas, solicitudes de arriendo, pagos y requerimientos.
5. Implementar reglas backend para estados, agenda sin superposiciones, evaluacion de solicitudes y reservas.
6. Completar acciones del ejecutivo: confirmar/modificar/rechazar/cancelar visitas y registrar resultados.
