# Barbería Kanusi — Contexto del proyecto

## Objetivo

Web real para una barbería de pueblo (estilo underground) y proyecto de aprendizaje
para mí (2º DAW). Debe estar online 365 días al año.

## Nombre

- Negocio: **Kanusi**. (Yayah es el nombre del barbero, no del negocio.)

## Stack

- Frontend: React + Vite + Tailwind, enrutado con React Router
- Backend: Node.js + NestJS, 2 microservicios:
  - `auth`: usuarios, registro, login, roles (cliente / barbero / barbero admin),
    alta de barberos y búsqueda exacta de clientes por teléfono o correo
  - `citas`: servicios y precios, barberos, horarios, citas y agenda
- BD: PostgreSQL (una base por servicio; estructura a definir en el modelo de datos)
- Despliegue: VPS con Docker Compose + Caddy (HTTPS), CI/CD con GitHub Actions
- Repo: monorepo (frontend/, services/ con auth y citas, docs/, docker-compose.yml)
- Mapa de páginas y rutas: `docs/routes.md` (fuente de verdad de rutas y permisos)

## Diseño

- Herramientas: FigJam (wireframes) → Figma Design (diseño final con Variables,
  Auto Layout, componentes con variantes y Dev Mode/MCP para pasar a React + Tailwind)
- Paleta: #dccc8d (acento dorado), #b5b5b5, #ffffff, #616060, #0c0c0c
- Mobile-first. Frames de referencia: 390 (móvil) / 768 (tableta) / 1440 (escritorio)
- Lienzo de FigJam: sección `Mapa del Sitio` (tabla de rutas) + una sección por
  pantalla (ver "Estado actual" para la numeración vigente)
- Convención en wireframes: cada enlace del menú lleva su `href` al lado
  (sin flechas); las acciones (p. ej. Logout) se marcan como `action=`;
  anclas marcadas con su `id` a la izquierda del bloque.
- Jerarquía de títulos: `h1` = frase del hero (uno por página), `h2` = título de
  cada sección, `h3` = cada servicio / cada barbero.

## Roles

- Visitante (sin sesión), Cliente, Barbero y Barbero admin.
- Cliente sin cuenta: no es un rol ni un usuario; es quien reserva por teléfono.
  Su cita guarda nombre y teléfono y la crea un barbero o el admin.
- El registro público siempre crea un cliente. El primer admin se crea con script;
  los barberos los da de alta el admin desde `/panel`.

## Funcionalidades v1 (cerradas en el paso 3)

- Ver servicios y precios (con duración)
- Registro (con teléfono obligatorio y casilla obligatoria de aceptación de la política de privacidad) y login
- `/panel`: editar datos personales; el admin, además, alta de barberos y acceso a `/horarios`
- Reservar cita (servicio → barbero → fecha/hora → confirmación) solo con sesión;
  barbero y admin añaden un primer paso «¿para quién?» (cliente con cuenta o sin cuenta)
- `/mis-citas`: lista por fecha según el rol, con cancelar
- `/horarios` (solo admin): horario de cada barbero **día a día** (fecha + franjas)
  y listado de citas en conflicto
- Política de privacidad (`/privacidad`) y página 404
- Contacto: botones email y WhatsApp (sin formulario → sin backend de correo)

## Decisiones tomadas

- FigJam: una sección por página, con móvil/tableta/escritorio dentro, de menor a mayor.
- Stack cambiado de Angular + Spring Boot a React + NestJS (en fase 0, sin código).
- Frontend con Vite. 2 microservicios: `auth` y `citas`.
- Idioma: código, archivos y componentes en inglés; textos visibles en español.
  **Las rutas visibles (URL) y los `id` de anclas, en español.** La API, en inglés.
- Landing en móvil: bloques `header`, `hero`, `servicios`, `barberos`, `contacto`
  (horarios + ubicación + email/WhatsApp + mapa) y `footer`. Sin formulario de contacto.
- Menú hamburguesa = overlay, con dos estados. Sin sesión: Servicios, Barberos,
  Contacto, Login. Con sesión: Servicios, Barberos, Contacto, Reservar, Mis citas
  («Agenda» para barbero y admin), Mi cuenta, Logout. Las anclas usan ruta absoluta
  (`/#servicios`). Logout es una acción (`<button>`), no un enlace.
- Rutas: `/`, `/reservar`, `/acceso`, `/panel`, `/mis-citas`, `/horarios`,
  `/privacidad`, `*` (404). `/modificar-cita/:id` queda aplazada a v2: en v1 se cancela y se vuelve a reservar.
- `/panel` = cuenta (servicio `auth`). `/mis-citas` = citas (servicio `citas`).
- Login con correo y contraseña en v1. Google, en v2 (la tabla de usuarios guardará `provider`).
- Solo se reserva con sesión: sin sesión se redirige a `/acceso?volver=/reservar`
  y se vuelve al flujo. `volver` solo acepta rutas internas.
- Una sola vista `/mis-citas` para todos los roles; el backend devuelve más o menos
  citas según el rol. Los permisos (rol y propiedad) se comprueban en cada acción.
  Cita ajena o inexistente → 404 (no 403).
- Cancelar no borra: la cita pasa a `cancelled`, libera el hueco y se conserva.
- **Reglas de cancelación:** el cliente cancela hasta 6 h antes del inicio; pasado
  ese plazo debe contactar con la barbería. No se cancelan citas pasadas.
  Barbero y admin cancelan sin límite de antelación (propuesto, a confirmar).
- Horarios en el servicio `citas`. El admin los define **día a día** (fecha +
  franjas inicio/fin). Un día sin horario no ofrece huecos. Agilizarlo (copiar
  semanas, plantillas) queda para después.
- Cambiar un horario solo afecta a huecos futuros: las citas existentes se conservan.
  El sistema solo muestra el listado de citas en conflicto; el admin llama y cancela a mano.
- Clientes sin cuenta: la cita guarda `client_id` (opcional) o `guest_name` y
  `guest_phone`. Búsqueda de cliente por coincidencia exacta de teléfono o correo,
  solo barbero y admin. Los datos se eliminan cuando dejan de ser necesarios.
- El teléfono es obligatorio (registro y cliente sin cuenta).
- Cambio de correo exige la contraseña actual (sin verificación por correo en v1).
- Recuperación de contraseña: en v1 la resetea a mano el admin.
- Servicios y precios: editados por script en v1.

## Fases de entrega (modelo B: por fases, desplegando pronto)

| Fase | Contenido | Resultado visible |
|---|---|---|
| v1.0 | Landing, `/privacidad`, 404, despliegue completo | La web está online |
| v1.1 | `auth` (registro con teléfono, login, roles), `/acceso`, `/panel` (datos y alta de barberos), primer admin por script | Entras con tu rol, editas tus datos y el admin crea barberos |
| v1.2 | `citas`: servicios, `/horarios` (día a día + conflictos), `/reservar` (con clientes sin cuenta), `/mis-citas` con cancelar | Se reserva y cancela con horarios reales, incluidos clientes por teléfono |
| v1.3 | Mejoras y pulido | Pulido |

Cada entrada del menú se añade cuando existe su ruta. La v1.2 es el grueso del
proyecto: se partirá en tareas internas al planificar la construcción.

## Por decidir

- Antelación mínima para reservar (¿se puede reservar para dentro de 1 hora?) y
  hasta cuántos días vista se puede reservar.
- Intervalo de los huecos (cada 15 / 30 min).
- Si un barbero puede crear citas en la agenda de otro barbero.
- Baja de barberos (desactivar, no borrar, para conservar el historial).
- Cambio obligatorio de la contraseña inicial del barbero en su primer acceso.
- Agrupación de la lista de citas: por días o por semanas.
- Texto de la política de privacidad (revisión legal) y aviso de cookies
  (depende de si usamos analítica o el mapa incrustado).
- Verificación del correo al cambiarlo (requeriría envío de correo).
- Cómo habla el frontend con los 2 servicios (p. ej. Caddy enrutando /api/auth y /api/appointments).
- Cómo se entera `citas` de un barbero nuevo creado en `auth` (modelo de datos).

## Estado actual

- Fase: 0 (diseño y modelo de datos)
- Hecho: paleta, landing móvil con menú overlay en dos estados (paso 2 ✅),
  mapa del sitio y alcance de la v1 (paso 3 ✅).
- Lienzo de FigJam: 00 Marca, 01 Landing, 02 Reservar cita, 03 Mis citas,
  03.1 Modificar cita (aplazada a v2), 04 Mi cuenta (`/panel`), 05 Login / Registro,
  06 Horarios (admin), 07 Privacidad y 404, más `Mapa del Sitio`.
  Las secciones 02-07 aún pueden ser copias de la landing.
- Siguiente: paso 4 → flujo de reserva en móvil. Después, paso 5 → sistema de diseño en Figma.

## Convenciones

- Commits: Conventional Commits (feat:, fix:, docs:)
- Ramas: feature/nombre-corto, PR hacia main
- Código en inglés, comentarios y docs en español
