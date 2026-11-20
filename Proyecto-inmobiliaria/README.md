# Gestión Urbana Propiedades

Sistema web fullstack para la gestion inmobiliaria de propiedades en arriendo. La aplicacion permite publicar inmuebles, agendar visitas, administrar solicitudes de arriendo, gestionar usuarios por rol y aplicar reglas de negocio como estados de propiedad, agenda sin choques y doble aprobacion de arriendos.

## Tabla de contenidos

- [Descripcion general](#descripcion-general)
- [Stack tecnologico](#stack-tecnologico)
- [Funcionalidades principales](#funcionalidades-principales)
- [Roles del sistema](#roles-del-sistema)
- [Estructura del repositorio](#estructura-del-repositorio)
- [Instalacion local](#instalacion-local)
- [Variables de entorno](#variables-de-entorno)
- [Usuarios de demostracion](#usuarios-de-demostracion)
- [Scripts disponibles](#scripts-disponibles)
- [API REST](#api-rest)
- [Reglas de negocio destacadas](#reglas-de-negocio-destacadas)
- [Pruebas y calidad](#pruebas-y-calidad)
- [Despliegue](#despliegue)
- [Documentacion adicional](#documentacion-adicional)

## Descripcion general

Gestion Urbana Propiedades es una plataforma academica para centralizar el proceso de arriendo y administracion de inmuebles. El sistema cubre el flujo principal del negocio:

1. Publicacion y consulta de propiedades.
2. Solicitud y gestion de visitas.
3. Registro e inicio de sesion de usuarios.
4. Administracion de propiedades por propietarios, ejecutivos y administradores.
5. Solicitudes de arriendo con evaluacion del ejecutivo y aprobacion del propietario.
6. Paneles diferenciados por rol.
7. Control de permisos, estados y reglas de agenda desde el backend.

La aplicacion esta separada en dos proyectos:

- `frontend/`: interfaz React construida con Vite.
- `backend/`: API REST en Node.js/Express conectada a MongoDB mediante Mongoose.

## Stack tecnologico

### Frontend

- React 18
- Vite
- React Router
- CSS modular por estructura de carpetas
- Fetch API para comunicacion con backend
- Context API para estado de autenticacion, propiedades, visitas y arriendos

### Backend

- Node.js 18+
- Express 5
- MongoDB Atlas / MongoDB replica set
- Mongoose
- JWT para autenticacion
- bcryptjs para hash de contrasenas
- Nodemailer para avisos por correo o salida por consola
- Node Test Runner para pruebas
- ESLint para calidad de codigo

## Funcionalidades principales

### Catalogo publico

- Listado de propiedades publicadas.
- Filtros por comuna, tipo, precio y caracteristicas.
- Ficha de detalle por propiedad.
- Ocultamiento de datos privados como RUT, propietario y direccion exacta para visitantes.

### Autenticacion y usuarios

- Registro de clientes y propietarios.
- Activacion por correo.
- Inicio de sesion con JWT.
- Perfil personal.
- Cambio de contrasena.
- Gestion de usuarios por administradores y ejecutivos autorizados.

### Propiedades

- Creacion y edicion de propiedades.
- Estados: `Disponible`, `Publicada`, `Reservada`, `Arrendada`, `En mantención` e `Inactiva`.
- Asociacion con propietario y ejecutivo responsable.
- Publicacion controlada por usuarios con permisos.
- Baja logica para conservar trazabilidad.

### Visitas

- Solicitud de visita desde la ficha de una propiedad.
- Horarios por bloques de 30 minutos.
- Agenda de lunes a viernes entre 09:00 y 18:00.
- Control de feriados, traslados y choques de agenda.
- Estados: `Pendiente`, `Confirmada`, `Rechazada`, `Cancelada` y `Realizada`.
- Registro de resultado de la visita.

### Solicitudes de arriendo

- Solicitud formal por parte de clientes activos.
- Evaluacion del ejecutivo responsable.
- Segunda aprobacion del propietario.
- Control de vencimiento cuando el propietario no responde.
- Reserva de propiedad al aprobarse una solicitud.
- Rechazo automatico de solicitudes incompatibles cuando una propiedad queda reservada.

### Paneles por rol

- Cliente: proximas visitas y solicitudes de arriendo.
- Propietario: propiedades propias, interes generado y solicitudes pendientes de aprobacion.
- Ejecutivo: cartera de propiedades, visitas y solicitudes de su responsabilidad.
- Administrador: vision global del sistema y gestion completa de usuarios.

## Roles del sistema

| Rol | Alcance principal |
| --- | --- |
| Visitante | Explora el catalogo y puede solicitar visitas dejando datos de contacto. |
| Cliente | Agenda visitas, solicita arriendos y revisa sus procesos. |
| Propietario | Administra sus propiedades y aprueba o rechaza solicitudes de arriendo. |
| Ejecutivo | Gestiona cartera, visitas, publicaciones y primera evaluacion de arriendos. |
| Administrador | Tiene permisos globales y administra usuarios del sistema. |

## Estructura del repositorio

```text
Proyecto-inmobiliaria/
├── backend/
│   ├── config/              # Conexion DB, roles, estados y constantes de negocio
│   ├── controllers/         # Logica de endpoints y reglas por recurso
│   ├── data/                # Datos semilla de demostracion
│   ├── middleware/          # Autenticacion JWT y control de roles
│   ├── models/              # Schemas Mongoose
│   ├── routes/              # Rutas Express por modulo
│   ├── test/                # Pruebas unitarias del backend
│   ├── utils/               # Agenda, permisos, cascadas, errores, RUT, correos
│   ├── .env.example         # Variables requeridas para la API
│   ├── seed.js              # Carga datos demo
│   ├── server.js            # Servidor Express
│   └── vercel.json          # Configuracion de deploy backend
├── frontend/
│   ├── src/
│   │   ├── components/      # Componentes reutilizables
│   │   ├── context/         # Providers de autenticacion, propiedades, visitas y arriendos
│   │   ├── data/            # Datos y constantes del frontend
│   │   ├── pages/           # Paginas/rutas de la aplicacion
│   │   ├── services/        # Cliente API, storage, filtros y utilidades
│   │   └── styles/          # Estilos globales y responsive
│   ├── .env.example         # URL del backend
│   ├── index.html
│   └── vercel.json          # Rewrites para React Router
├── docs/
│   └── modelo-logico.md     # Modelo de datos, permisos y reglas de negocio
└── README.md
```

## Instalacion local

### Requisitos previos

- Node.js 18 o superior.
- npm.
- MongoDB Atlas o MongoDB local configurado como replica set.

> Importante: el backend usa transacciones. MongoDB Atlas ya funciona como replica set; si se usa MongoDB local, debe iniciarse con replica set.

### 1. Clonar el repositorio

```bash
git clone <url-del-repositorio>
cd <nombre-del-repositorio>/Proyecto-inmobiliaria
```

### 2. Configurar backend

```bash
cd backend
npm install
cp .env.example .env
```

Editar `backend/.env` y completar como minimo:

```env
MONGODB_URI=mongodb+srv://USUARIO:CONTRASENA@CLUSTER.mongodb.net/?retryWrites=true&w=majority
MONGODB_DB=gestion_inmobiliaria
JWT_SECRET=un-secreto-largo-y-seguro
FRONTEND_URL=http://localhost:5173
PORT=3000
```

Cargar datos de demostracion:

```bash
npm run seed
```

Levantar la API:

```bash
npm run dev
```

La API queda disponible en:

```text
http://localhost:3000
```

Comprobacion rapida:

```text
GET http://localhost:3000/api/salud
```

Respuesta esperada:

```json
{
  "api": "ok",
  "mongodb": "conectada"
}
```

### 3. Configurar frontend

En otra terminal:

```bash
cd frontend
npm install
cp .env.example .env
```

Verificar `frontend/.env`:

```env
VITE_API_URL=http://localhost:3000
```

Levantar la interfaz:

```bash
npm run dev
```

La aplicacion queda disponible en:

```text
http://localhost:5173
```

## Variables de entorno

### Backend

| Variable | Obligatoria | Descripcion |
| --- | --- | --- |
| `MONGODB_URI` | Si | Cadena de conexion a MongoDB. |
| `MONGODB_DB` | No | Nombre de la base de datos. Por defecto: `gestion_inmobiliaria`. |
| `JWT_SECRET` | Si | Secreto para firmar tokens JWT. |
| `FRONTEND_URL` | Si | Origen permitido por CORS y usado en enlaces de correo. |
| `PORT` | No | Puerto local de la API. Por defecto: `3000`. |
| `SMTP_HOST` | No | Servidor SMTP para envio real de correos. |
| `SMTP_PORT` | No | Puerto SMTP. |
| `SMTP_USER` | No | Usuario SMTP. |
| `SMTP_PASS` | No | Contrasena o app password SMTP. |
| `SMTP_FROM` | No | Remitente de correos. |
| `CRON_SECRET` | No | Secreto para tareas programadas en Vercel. |

Si no se configuran variables SMTP, los correos de activacion y avisos se imprimen en la terminal del backend.

### Frontend

| Variable | Obligatoria | Descripcion |
| --- | --- | --- |
| `VITE_API_URL` | Si | URL base de la API Express. |

## Usuarios de demostracion

Despues de ejecutar `npm run seed`, se crean usuarios ficticios para probar el sistema.

| Rol | Usuario | Contrasena |
| --- | --- | --- |
| Administrador | `admin` | `admin123` |
| Ejecutivo | `ejecutivo` | `ejecutivo123` |
| Propietario | `propietario` | `propietario123` |
| Cliente | `cliente` | `cliente123` |

Tambien existen usuarios adicionales en `backend/data/semilla.js` para probar carteras, solicitudes, visitas y estados distintos.

## Scripts disponibles

### Backend

Ejecutar desde `Proyecto-inmobiliaria/backend`:

| Comando | Descripcion |
| --- | --- |
| `npm run dev` | Levanta la API con `node --watch server.js`. |
| `npm start` | Levanta la API en modo normal. |
| `npm run seed` | Borra y recarga los datos demo. |
| `npm test` | Ejecuta pruebas con Node Test Runner. |
| `npm run lint` | Ejecuta ESLint. |

### Frontend

Ejecutar desde `Proyecto-inmobiliaria/frontend`:

| Comando | Descripcion |
| --- | --- |
| `npm run dev` | Levanta Vite en desarrollo. |
| `npm run build` | Genera build de produccion. |
| `npm run preview` | Sirve localmente el build generado. |
| `npm run lint` | Ejecuta ESLint. |

## API REST

La API responde JSON. Las rutas protegidas requieren:

```http
Authorization: Bearer <token>
```

Los errores devuelven un objeto con formato:

```json
{
  "mensaje": "Descripcion del error"
}
```

### Endpoints principales

| Recurso | Rutas |
| --- | --- |
| Salud | `GET /api/salud` |
| Auth | `POST /api/auth/login`, `POST /api/auth/registro`, `POST /api/auth/verificar`, `POST /api/auth/reenviar-verificacion`, `GET /api/auth/yo`, `GET /api/auth/perfil`, `PATCH /api/auth/perfil`, `PATCH /api/auth/password` |
| Propiedades | `GET /api/propiedades`, `GET /api/propiedades/:id`, `GET /api/propiedades/:id/horarios`, `POST /api/propiedades`, `PATCH /api/propiedades/:id`, `POST /api/propiedades/:id/estado`, `DELETE /api/propiedades/:id` |
| Visitas | `GET /api/visitas`, `POST /api/visitas`, `POST /api/visitas/:id/estado`, `PATCH /api/visitas/:id`, `DELETE /api/visitas/:id` |
| Arriendos | `GET /api/arriendos`, `POST /api/arriendos`, `POST /api/arriendos/:id/estado`, `DELETE /api/arriendos/:id` |
| Usuarios | `GET /api/usuarios`, `POST /api/usuarios`, `PATCH /api/usuarios/:id`, acciones de verificacion, rechazo, baja, reactivacion y eliminacion |
| Tareas | `GET /api/tareas` para procesos programados |

## Reglas de negocio destacadas

- Los roles validos son `admin`, `ejecutivo`, `propietario` y `cliente`.
- Las cuentas autorregistradas requieren verificacion de correo.
- Los propietarios requieren validacion adicional del ejecutivo o administrador antes de gestionar.
- Una propiedad puede estar `Disponible`, `Publicada`, `Reservada`, `Arrendada`, `En mantención` o `Inactiva`.
- Solo propiedades `Publicada` y `Reservada` aparecen en el catalogo publico.
- Las visitas se agendan en bloques de 30 minutos.
- La agenda considera horario laboral, feriados, traslado entre propiedades y choques por ejecutivo o propiedad.
- Un cliente no puede mantener visitas activas duplicadas para una misma propiedad.
- Las solicitudes de arriendo tienen doble aprobacion: ejecutivo y propietario.
- Al aprobarse una solicitud, la propiedad queda reservada y se rechazan solicitudes incompatibles.
- Las bajas relevantes son logicas para conservar historial.

La referencia completa esta en [`docs/modelo-logico.md`](docs/modelo-logico.md).

## Pruebas y calidad

### Backend

```bash
cd backend
npm test
npm run lint
```

Las pruebas cubren reglas como agenda, autenticacion, permisos, semilla y reasignacion.

### Frontend

```bash
cd frontend
npm run lint
npm run build
```

## Despliegue

El repositorio incluye configuracion para Vercel:

- `frontend/vercel.json`: redirige todas las rutas a `index.html` para que funcione React Router.
- `backend/vercel.json`: define Express como framework y una tarea cron para `/api/tareas`.

### Variables para deploy

En el backend desplegado se deben configurar:

- `MONGODB_URI`
- `MONGODB_DB`
- `JWT_SECRET`
- `FRONTEND_URL`
- variables SMTP opcionales
- `CRON_SECRET` si se protege la tarea programada

En el frontend desplegado:

- `VITE_API_URL` apuntando a la URL publica del backend.

## Documentacion adicional

- Modelo logico, reglas, permisos y entidades: [`docs/modelo-logico.md`](docs/modelo-logico.md)
- Datos demo: [`backend/data/semilla.js`](backend/data/semilla.js)
- Constantes de negocio: [`backend/config/constantes.js`](backend/config/constantes.js)
- Rutas principales de frontend: [`frontend/src/App.jsx`](frontend/src/App.jsx)

## Estado del proyecto

Proyecto academico funcional con frontend, backend, persistencia en MongoDB, autenticacion por JWT, roles, reglas de negocio, semilla de datos y pruebas automatizadas para la capa backend.
