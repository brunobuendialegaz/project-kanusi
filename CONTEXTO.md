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
  - Texto secundario sobre #0c0c0c: usar #b5b5b5 (el #616060 tiene contraste flojo).
- Mobile-first. Frames de referencia: 390 (móvil) / 768 (tableta) / 1440 (escritorio)
- Lienzo de FigJam: sección `Mapa del Sitio` (tabla de rutas) + una sección por
  pantalla (ver "Estado actual" para la numeración vigente)
- Convención en wireframes: cada enlace del menú lleva su `href` al lado
  (sin flechas); las acciones (p. ej. Logout) se marcan como `action=`;
  anclas marcadas con su `id` a la izquierda del bloque.
- Jerarquía de títulos: `h1` = frase del hero (uno por página), `h2` = título de
  cada sección, `h3` = cada servicio / cada barbero.
- Patrón de selección (todo el proyecto): elemento elegido = borde dorado `#dccc8d`
  brillante con sombra (glow) del mismo color.
- Teléfono de la barbería: un solo dato reutilizado (contacto, avisos, estados vacíos),
  siempre como enlace `tel:`. De momento marcador `XXXXXXXXX`; irá el real.

## Roles

- Visitante (sin sesión), Cliente, Barbero y Barbero admin.
- Cliente sin cuenta: no es un rol ni un usuario; es quien reserva por teléfono.
  Su cita guarda nombre y teléfono y la crea un barbero o el admin.
- El registro público siempre crea un cliente. El primer admin se crea con script;
  los barberos los da de alta el admin desde `/panel`.

## Funcionalidades v1 (cerradas en el paso 3; flujo de reserva detallado en el paso 4)

- Ver servicios y precios (con duración)
- Registro (con teléfono obligatorio y casilla obligatoria de aceptación de la política de privacidad) y login
- `/panel`: editar datos personales; el admin, además, alta de barberos y acceso a `/horarios`
- Reservar cita solo con sesión, en un flujo de un módulo visible a la vez
  (ver "Flujo de reserva"); barbero y admin añaden un primer paso «¿Para quién?»
- `/mis-citas`: lista por fecha según el rol, con cancelar
- `/horarios` (solo admin): horario de cada barbero **día a día** (fecha + franjas)
  y listado de citas en conflicto
- Política de privacidad (`/privacidad`) y página 404
- Contacto: botones email y WhatsApp (sin formulario → sin backend de correo)

## Flujo de reserva `/reservar` (cerrado en el paso 4)

**Estructura**
- Una sola ruta `/reservar`; un componente guarda en el estado el paso actual y lo elegido,
  y muestra **un solo módulo** a la vez. Sin navbar: arriba, solo la barra de progreso;
  abajo, el footer; solo el módulo central hace scroll.
- Pasos según rol:
  - Cliente (4): Servicio → Barbero → Fecha y hora → Confirmar.
  - Barbero (4): ¿Para quién? → Servicio → Fecha y hora → Confirmar (el barbero va fijo
    en él mismo; el paso no se muestra).
  - Admin (5): ¿Para quién? → Servicio → Barbero → Fecha y hora → Confirmar.
- Servicio y barbero **avanzan solos** al tocar. Elegir hora lleva al resumen (no reserva).
  Solo el botón «Confirmar reserva» es irreversible. «¿Para quién?» lleva botón «Continuar».
- Lo elegido se conserva al volver; **cambiar una elección invalida lo que depende de ella**
  (p. ej. cambiar el servicio descarta la hora).

**Barra de progreso**
- Móvil (390): un segmento por paso (actual dorado, completados dorado atenuado y tocables,
  futuros grises y no tocables) y debajo «Paso 2 de 4 · Barbero». Área táctil ≥ 44 px.
- Tableta y escritorio: etiquetas completas de cada paso (misma lógica, otra presentación).
- A la izquierda, flecha/X: en el paso 1 lleva a `/`; en los demás vuelve al paso anterior.
  Salir con la X no pregunta, salvo en el resumen, donde pide confirmación.

**Módulos**
- **Servicio:** tarjeta mínima (nombre, duración, precio), una por fila a todo el ancho.
  Lista vacía o fallo de la petición: mensaje con teléfono dentro del módulo (no 404). Estado «cargando».
- **Barbero:** tres tarjetas iguales (barbero 1, barbero 2, «Cualquiera»). «Cualquiera» lleva la
  línea «Quien esté libre a tu hora». Sin foto: inicial en círculo dorado (placeholder; irán fotos reales);
  «Cualquiera» con icono genérico.
- **Fecha y hora:** `<select>` con los 14 días («Hoy · dom 11 oct»…); días sin huecos desactivados y
  marcados «· sin huecos»; primer día con huecos preseleccionado. Debajo, horas en cuadrícula de
  2 columnas (orden por filas: 1 2 / 3 4), agrupadas en «Mañana» y «Tarde», solo «10:00», «10:30».
  Con «Cualquiera», un hueco existe si al menos un barbero está libre. Estados: día sin huecos
  («Sin huecos este día»), ningún día con huecos («No hay huecos en los próximos 14 días. Llama al …»), cargando.
- **¿Para quién? (barbero y admin):** un campo «Teléfono o correo del cliente» (si contiene `@`, es correo) y «Buscar».
  Con resultado: tarjeta con nombre y teléfono (nunca correo ni citas), «¿Es esta persona?» y «Cambiar cliente».
  Sin resultado: «No encontrado. Introduce sus datos» con nombre y teléfono (teléfono ya relleno si se buscó por teléfono).
  Nombre y teléfono obligatorios, con error bajo cada campo.
- **Resumen:** tarjeta con servicio, barbero, día y hora (y cliente para el personal). Con «Cualquiera»:
  «Se te asignará al confirmar». Aviso dorado **debajo de los datos y encima del botón**, solo si la cita
  empieza en menos de 6 h: solo se puede cambiar o cancelar por teléfono (enlace `tel:`).
  Botón fijo abajo; al pulsarlo pasa a «Reservando…» y se desactiva. Si la hora se ocupó entretanto, el
  mensaje lleva al paso de fecha y hora con el mismo día seleccionado, conservando lo demás.
- **Confirmación (estado final, no es una ruta):** icono de éxito, «Reserva confirmada», tarjeta con servicio,
  barbero **ya asignado**, día y hora, y el aviso del teléfono si aplica. Sin barra de progreso.
  Cliente: «Ver mis citas» (principal) y «Volver al inicio». Personal: «Reservar otra cita» (principal, arranca
  limpio en «¿Para quién?») y «Ver agenda».

**Persistencia**
- `sessionStorage` con IDs de servicio, barbero, día, hora y paso actual (no nombres ni precios).
  **No se guardan** los datos del cliente del personal: tras recargar, el personal repite «¿Para quién?»
  y después salta al punto donde estaba.
- Al restaurar se revalida siempre (servicio existente, hora libre y no pasada).
- Se limpia al confirmar con éxito, al salir con la X desde el paso 1 y al hacer logout.
- Sesión caducada: `/acceso?volver=/reservar`, y al volver se recupera lo elegido.

**Reglas de negocio (validadas en el backend con hora del servidor, Europe/Madrid)**
- Antelación mínima del cliente: **30 min**. Barbero y admin: sin mínimo.
- Ventana de reserva: **14 días** vista (el frontend solo oculta lo que queda fuera).
- Huecos cada **30 min** (parámetro único de configuración). Un hueco es válido si el servicio entero
  cabe sin pisar otra cita. Duración real guardada por servicio: corte 30 min, corte+barba 45 min.
- Barbero: crea citas solo en su propia agenda. Admin: en cualquiera.
- «Cualquiera»: el backend asigna un barbero libre (regla provisional: el primero de la lista que esté libre).
- Una cita idéntica del mismo cliente a la misma hora se rechaza (protege del doble toque).

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
  franjas inicio/fin). Un día sin horario no ofrece huecos.
- Los turnos/plantillas (v2) serán un atajo que **genera días individuales** en la misma tabla,
  no una regla viva; cada día se modifica después por separado. El modelo de v1 no cambia.
- Cambiar un horario solo afecta a huecos futuros: las citas existentes se conservan.
  El sistema solo muestra el listado de citas en conflicto; el admin llama y cancela a mano.
- Clientes sin cuenta: la cita guarda `client_id` (opcional) o `guest_name` y
  `guest_phone`. Búsqueda de cliente por coincidencia exacta de teléfono o correo,
  solo barbero y admin. Los datos se eliminan cuando dejan de ser necesarios.
- El teléfono es obligatorio (registro y cliente sin cuenta).
- Cambio de correo exige la contraseña actual (sin verificación por correo en v1).
- Recuperación de contraseña: en v1 la resetea a mano el admin.
- Servicios y precios: editados por script en v1.
- Tableta y escritorio en v1: el mismo módulo de móvil, centrado con ancho máximo (~480-560 px).
  Composiciones más complejas, en v2.

## Fases de entrega (modelo B: por fases, desplegando pronto)

| Fase | Contenido | Resultado visible |
|---|---|---|
| v1.0 | Landing, `/privacidad`, 404, despliegue completo | La web está online |
| v1.1 | `auth` (registro con teléfono, login, roles), `/acceso`, `/panel` (datos y alta de barberos), primer admin por script | Entras con tu rol, editas tus datos y el admin crea barberos |
| v1.2 | `citas`: servicios, `/horarios` (día a día + conflictos), `/reservar` (con clientes sin cuenta), `/mis-citas` con cancelar | Se reserva y cancela con horarios reales, incluidos clientes por teléfono |
| v1.3 | Mejoras y pulido | Pulido |

Cada entrada del menú se añade cuando existe su ruta. La v1.2 es el grueso del
proyecto: se partirá en tareas internas al planificar la construcción.

## Aplazado a v2

- Turnos/plantillas de horario (p. ej. «semanal L-V 10-14 y 17-21», «solo tardes»).
- `/modificar-cita/:id`, login con Google, recuperación de contraseña, edición de servicios desde la web.
- Botón «lo antes posible» dedicado (en v1 lo cubre «Cualquiera» + primer día con huecos preseleccionado).
- Añadir la cita al calendario del móvil (`.ics`).
- Composiciones propias para tableta y escritorio en `/reservar`.
- Que un barbero cree citas en la agenda de otro (abrirlo es quitar una restricción).
- Fotos reales de los barberos (en v1, inicial en círculo).

## Por decidir

- Qué pasa al aplicar un turno sobre un día que ya tiene horario (sobrescribir, saltar o preguntar). v2.
- Regla definitiva de asignación con «Cualquiera» (provisional: el primero de la lista libre).
- Confirmar con Yayah la duración real de corte+barba (45 min) y si le encaja perder 15 min por cita.
- Teléfono real de la barbería y fotos reales de los barberos.
- Cómo piden las API la disponibilidad: días con huecos de los 14 y horas de un día (se cierra al diseñar la API).
- Baja de barberos (desactivar, no borrar, para conservar el historial).
- Cambio obligatorio de la contraseña inicial del barbero en su primer acceso.
- Agrupación de la lista de citas: por días o por semanas.
- Texto de la política de privacidad (revisión legal) y aviso de cookies
  (depende de si usamos analítica o el mapa incrustado).
- Verificación del correo al cambiarlo (requeriría envío de correo).
- Cómo habla el frontend con los 2 servicios (p. ej. Caddy enrutando /api/auth y /api/appointments).
- Cómo se entera `citas` de un barbero nuevo creado en `auth` (modelo de datos).
- Si barbero y admin cancelan sin límite de antelación (propuesto).

## Estado actual

- Fase: 0 (diseño y modelo de datos)
- Hecho: paleta, landing móvil con menú overlay en dos estados (paso 2 ✅),
  mapa del sitio y alcance de la v1 (paso 3 ✅), decisiones de diseño del flujo
  de reserva (paso 4, parte de decisiones ✅).
- Lienzo de FigJam: 00 Marca, 01 Landing, 02 Reservar cita, 03 Mis citas,
  03.1 Modificar cita (aplazada a v2), 04 Mi cuenta (`/panel`), 05 Login / Registro,
  06 Horarios (admin), 07 Privacidad y 404, más `Mapa del Sitio`.
  Las secciones 03-07 aún pueden ser copias de la landing.
- Siguiente: paso 4 (parte de dibujo) → dibujar `/reservar` en FigJam, módulo a módulo.
  Después, paso 5 → sistema de diseño en Figma.

## Convenciones

- Commits: Conventional Commits (feat:, fix:, docs:)
- Ramas: feature/nombre-corto, PR hacia main
- Código en inglés, comentarios y docs en español
