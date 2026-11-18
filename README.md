# Gestión Inmobiliaria
# Gestión Urbana Propiedades
Aplicación web para administrar el arriendo de propiedades: catálogo público, agenda de visitas,
solicitudes de arriendo con doble aprobación y administración de usuarios por rol
(admin, ejecutivo, propietario, cliente).
Sistema web fullstack para la gestion inmobiliaria de propiedades en arriendo. La aplicacion permite publicar inmuebles, agendar visitas, administrar solicitudes de arriendo, gestionar usuarios por rol y aplicar reglas de negocio como estados de propiedad, agenda sin choques y doble aprobacion de arriendos.
- **Backend:** Node.js + Express 5 + MongoDB (Mongoose) — carpeta `backend/`
- **Frontend:** React + Vite — carpeta `frontend/`
- **Modelo de datos y reglas de negocio:** `docs/modelo-logico.md`
## Tabla de contenidos
## Estructura del backend
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
backend/
├── server.js            Servidor Express: middleware, rutas, manejo de errores y tareas horarias
├── config/
│   ├── db.js            Conexión a MongoDB (Mongoose)
│   └── constantes.js    Roles, estados y parámetros de negocio
├── models/              Schemas de Mongoose: Usuario, Propiedad, Visita, Arriendo
├── routes/              Solo URLs y permisos: método + ruta + middleware → controlador
├── controllers/         Lógica de cada endpoint (validaciones, consultas, transacciones)
├── middleware/auth.js   Verificación del JWT y de los roles
├── utils/               Reglas reutilizables: agenda, permisos, cascadas, RUT, correo
├── data/semilla.js      Datos de demostración
├── seed.js              Carga la semilla en la base (npm run seed)
└── test/                Pruebas unitarias (node --test)
```
## Puesta en marcha
## Instalacion local
Requisitos: Node.js 18 o superior y una base MongoDB que sea **replica set** (MongoDB Atlas lo es;
se necesita para las transacciones).
### Requisitos previos
### 1. Backend
- Node.js 18 o superior.
- npm.
- MongoDB Atlas o MongoDB local configurado como replica set.
> Importante: el backend usa transacciones. MongoDB Atlas ya funciona como replica set; si se usa MongoDB local, debe iniciarse con replica set.
### 1. Clonar el repositorio
