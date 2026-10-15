# Barbería Kanusi — Contexto del proyecto

## Objetivo

Web real para una barbería de pueblo (estilo underground) y proyecto de aprendizaje
para mí (2º DAW). Debe estar online 365 días al año.

## Nombre

- Negocio: **Kanusi**. (Yayah es el nombre del barbero, no del negocio.)

## Stack

- Frontend: React + Vite + Tailwind, enrutado con React Router.
  `react-day-picker` para el calendario de `/horarios` (selección múltiple; revisar su documentación al instalar).
- Backend: Node.js + NestJS, 2 microservicios:
  - `auth`: usuarios, registro, login, roles (cliente / barbero / barbero admin),
    cambio de contraseña, alta de barberos y búsqueda exacta de clientes por teléfono o correo
  - `citas`: servicios y precios, barberos, horarios, citas y agenda
- BD: PostgreSQL (una base por servicio; estructura a definir en el modelo de datos)
- Despliegue: VPS con Docker Compose + Caddy (HTTPS), CI/CD con GitHub Actions
- Repo: monorepo (frontend/, services/ con auth y citas, docs/, docker-compose.yml)
- Mapa de páginas y rutas: `docs/routes.md` (fuente de verdad de rutas y permisos)

## Seguridad (regla de base)

- La seguridad vive en el backend; el frontend solo oculta cosas por comodidad.
- `RolesGuard` + decorador `@Roles(...)` en cada endpoint protegido, denegando por defecto.
- El rol sale siempre del token verificado o de la BD, nunca de un campo del body ni de `localStorage`.
- `ValidationPipe` con `whitelist: true`: el registro público ignora cualquier campo `role` y fija «cliente» en el código.
- Prueba mínima: llamar a un endpoint de admin con el token de un cliente y comprobar que responde 403.

## Diseño

- Herramientas: FigJam (wireframes) → Figma Design (diseño final con Variables,
  Auto Layout, componentes con variantes y Dev Mode/MCP para pasar a React + Tailwind)
- Paleta: #dccc8d (acento dorado), #b5b5b5, #ffffff, #616060, #0c0c0c
  - Texto secundario sobre #0c0c0c: usar #b5b5b5 (el #616060 tiene contraste flojo).
- Mobile-first. Frames de referencia: 390 (móvil) / 768 (tableta) / 1440 (escritorio)
- Lienzo de FigJam: sección `Mapa del Sitio` (tabla de rutas) + una sección por
  pantalla: 00 Marca, 01 Landing Page, 02 Login / Registro, 03 Reservar cita, 04 Panel,
  05 Mis citas, 07 Horarios, 08 Privacidad - 404. La 06 queda sin usar
  (reservada a «Modificar cita», aplazada a v2).
- Convención en wireframes: cada enlace del menú lleva su `href` al lado
  (sin flechas); las acciones (p. ej. Logout) se marcan como `action=`;
  anclas marcadas con su `id` a la izquierda del bloque.
- Jerarquía de títulos: `h1` = frase del hero (uno por página), `h2` = título de
  cada sección, `h3` = cada servicio / cada barbero.
- Patrón de selección (todo el proyecto): elemento elegido = borde dorado `#dccc8d`
  brillante con sombra (glow) del mismo color. En los wireframes de FigJam basta un borde
  dorado grueso con una nota «borde dorado + glow»; el glow real se hace en Figma Design.
- Teléfono de la barbería: un solo dato reutilizado (contacto, avisos, estados vacíos),
  siempre como enlace `tel:`. De momento marcador `XXXXXXXXX`; irá el real.
- Etiquetas siempre visibles encima de los campos, no solo dentro.
- Popups: ninguno, salvo `ConfirmDialog` (un solo componente, `<dialog>` nativo con `showModal()`)
  para confirmar acciones destructivas. Avisos, errores y mensajes de éxito van dentro del módulo.
- Ritmo de trabajo: en el wireframe solo decisiones estructurales (qué hay, qué hace, qué estados);
  los detalles visuales van a la lista del paso 5.

## Roles

- Visitante (sin sesión), Cliente, Barbero y Barbero admin.
- Cliente sin cuenta: no es un rol ni un usuario; es quien reserva por teléfono.
  Su cita guarda nombre y teléfono y la crea un barbero o el admin.
- El registro público siempre crea un cliente. El primer admin se crea con script;
  los barberos los da de alta el admin desde `/panel`.

## Funcionalidades v1 (cerradas en el paso 3; detalle de pantallas en el paso 4)

- Ver servicios y precios (con duración)
- Registro (nombre, correo, teléfono obligatorio, contraseña y casilla obligatoria de aceptación de la política de privacidad) y login
- `/panel`: editar nombre y teléfono, cambiar contraseña; el admin, además, alta de barberos y acceso a `/horarios`
- Reservar cita solo con sesión, en un flujo de un módulo visible a la vez
  (ver «Flujo de reserva»); barbero y admin añaden un primer paso «¿Para quién?»
- `/mis-citas`: lista según el rol, con filtro «Pendientes» / «Todas» y cancelar
- `/horarios` (solo admin): horario de cada barbero con selección múltiple de días y listado de citas en conflicto
- Política de privacidad (`/privacidad`) y página 404
- Contacto: botones email y WhatsApp (sin formulario → sin backend de correo)

## Pantalla `/acceso` (cerrada en el paso 4)

- Una sola ruta con **dos módulos** (no pestañas): login (estado inicial) y registro. Se cambia con un enlace
  al pie de cada uno; al cambiar, el foco va al primer campo del módulo nuevo.
- **Login:** Correo, Contraseña, botón «Entrar», enlace «¿No tienes cuenta? **Regístrate**» y, como texto fijo,
  «¿Olvidaste tu contraseña? Llama al [teléfono]» (`tel:`).
- **Registro:** Nombre, Correo, Teléfono, Contraseña, Repetir contraseña, casilla «He leído y acepto la
  **política de privacidad**» (enlace a `/privacidad` en pestaña nueva, justo encima del botón), botón
  «Crear cuenta» y enlace «¿Ya tienes cuenta? **Entra**».
- **Errores:** bajo cada campo, al pulsar el botón (no mientras se escribe); el foco va al primer campo con
  error. En el login, un solo mensaje sobre el botón («Correo o contraseña incorrectos») sin decir cuál falló.
  Correo ya registrado: error bajo Correo. Fallo de red: mensaje sobre el botón y el botón vuelve a estar activo.
- **Tras registrarse:** entra con la sesión iniciada y va a `volver` o, si no hay, a `/mis-citas`.
- **Estados:** «Entrando…» y «Creando cuenta…» (botón desactivado, evita el doble toque).

## Pantalla `/panel` (cerrada en el paso 4)

- Una sola página; los bloques cambian según el rol. Cliente y barbero ven lo mismo; el admin, además, «Gestión».
  No lleva «Cerrar sesión» (ya está en el menú).
- **Mis datos:** formulario con nombre y teléfono ya rellenos, editables, y botón «Guardar» («Guardando…» y
  «Datos guardados» dentro del módulo). Errores bajo cada campo.
- **Correo:** solo lectura, como texto plano, con la nota «Para cambiarlo, llama al [teléfono]» (`tel:`).
- **Cambiar contraseña** (todos los roles): módulo con contraseña actual, nueva y repetir; estados «Cambiando
  contraseña…», «Contraseña cambiada» y error «La contraseña actual no es correcta».
- **Gestión (solo admin):** botones «Dar de alta barbero» (abre el formulario dentro del mismo módulo, que crece)
  y «Horarios» (a `/horarios`). Formulario: nombre, correo, teléfono y contraseña inicial, todos obligatorios, con
  error bajo cada campo; botón «Crear barbero» («Creando…»); tras crearlo, «Barbero creado» y el formulario se vacía.
  Correo existente: error bajo Correo.

## Pantalla `/mis-citas` (cerrada en el paso 4)

- **Filtro** de una sola opción arriba (dos botones, el elegido con el patrón de selección): «Pendientes» (por
  defecto; citas futuras no canceladas, **la más próxima primero**) y «Todas» (pasadas, futuras y canceladas con su
  estado visible, **la más reciente primero**). La carga de «Todas» se limitará al diseñar la API.
- **Tarjeta común, con líneas según el rol:**

| Rol | Línea 1 | Línea 2 |
|---|---|---|
| Cliente | Fecha, Hora | Barbero, Servicio |
| Barbero | Fecha, Hora | Cliente · teléfono, Servicio |
| Admin | Fecha, Hora, Barbero | Cliente · teléfono, Servicio |

  El teléfono es un enlace `tel:`. Las citas de clientes sin cuenta se muestran igual (`guest_name`, `guest_phone`).
  En «Todas», las canceladas y pasadas llevan el texto «Cancelada» o «Realizada» y no llevan botón de cancelar.
- **Cancelar:** botón «X» en la tarjeta (área táctil de 44 px y `aria-label` «Cancelar cita del [fecha]»; no cabe
  el texto). Abre el `ConfirmDialog`: «¿Cancelar esta cita?», fecha y hora, y dos botones a ancho completo, uno sobre
  otro: «Mantener cita» (principal) y «Sí, cancelar» («Cancelando…» mientras se envía). Tras confirmar, la cita pasa
  a «Cancelada» y sale de «Pendientes».
- **Cliente con la cita a menos de 6 h:** botón desactivado y texto «Para cancelar, llama al [teléfono]» (`tel:`).
- **Estados:** lista vacía (Pendientes: «No hay citas pendientes» y, solo para el cliente, botón «Reservar»; Todas:
  «Aún no hay citas»), cargando y error de carga con «Reintentar».

## Pantalla `/horarios` (cerrada en el paso 4)

- **Selector de barbero** (desplegable) arriba. Con otro barbero elegido, la lista abierta se cierra.
- **Calendario** de selección múltiple (`react-day-picker`): se marcan los días que haga falta; los días que ya
  tienen horario aparecen marcados.
- **Franjas:** Inicio / Fin (mañana) e Inicio tarde / Fin tarde, en saltos de 30 min. Ambas franjas son opcionales
  pero debe haber al menos una completa. Errores bajo el campo (p. ej. «El fin debe ser posterior al inicio»; la
  tarde empieza después de la mañana; «Selecciona al menos un día»).
- **«Añadir»:** guarda las franjas en cada día elegido y **sobrescribe** los días que ya tenían horario («Añadiendo…»
  y «Horario guardado»). Los días siguen siendo entradas individuales en la misma tabla.
- **«Listar horarios»:** debajo del formulario, tarjetas de los días futuros del barbero elegido (hoy incluido),
  fecha ascendente, una línea por día (`Lun 19 oct · 10:00-14:00 y 17:00-21:00`), cada una con «X» para quitar.
  Estados: vacía («No hay horarios establecidos»), cargando y error de carga con «Recargar».
- **Quitar un día:** `ConfirmDialog` «¿Quitar el horario de [barbero] del [fecha]?», con «Mantener» (principal) y
  «Sí, quitar» («Quitando…»). Si ese día tiene citas, el modal lo avisa («3 citas agendadas»).
- **Conflictos:** solo existen cuando una **cita** queda fuera del horario nuevo (al sobrescribir con un horario más
  corto o al quitar un día). Sobrescribir un día no es un conflicto. Tras guardar o quitar, aparece en el mismo módulo
  el aviso «N citas quedan fuera del horario» y una tarjeta por cita (`Fecha · Hora · Barbero` / `Cliente · teléfono ·
  Servicio`, con «Fuera de horario» en rojo y «X» para cancelar con el `ConfirmDialog` de cita). La cita **conserva su
  barbero**; el admin llama al cliente y cancela a mano. Al cancelar, la tarjeta desaparece.

## Pantallas `/privacidad` y 404 (cerradas en el paso 4)

- **`/privacidad`:** una página con `h1` «Política de privacidad» y seis `h2` seguidos (sin acordeón): Responsable
  (nombre o razón social, NIF, dirección y correo del titular), Qué datos guardamos, Para qué los usamos, Cuánto
  tiempo los conservamos, Tus derechos y cómo ejercerlos, Contacto.
- **404:** cabecera y footer normales, `h1` «Página no encontrada», una línea de texto y botón «Volver al inicio»
  (`/`). Es la misma pantalla para una URL mal escrita, una cita ajena o `/horarios` sin ser admin.

## Flujo de reserva `/reservar` (cerrado en el paso 4)

**Estructura**
- Una sola ruta `/reservar`; un componente guarda en el estado el paso actual y lo elegido,
  y muestra **un solo módulo** a la vez. Sin navbar: arriba, solo la barra de progreso;
  abajo, el footer; solo el módulo central hace scroll.
- Pasos según rol:
  - Cliente (4): Servicio → Barbero → Fecha y hora → Resumen.
  - Barbero (4): ¿Para quién? → Servicio → Fecha y hora → Resumen (el barbero va fijo
    en él mismo; el paso no se muestra).
  - Admin (5): ¿Para quién? → Servicio → Barbero → Fecha y hora → Resumen.
  - En la barra el segmento del primer paso del personal se llama «Cliente» y el último, «Resumen»
    (el botón de ese paso sigue diciendo «Confirmar reserva»).
- Servicio y barbero **avanzan solos** al tocar. Elegir hora lleva al resumen (no reserva).
  Solo el botón «Confirmar reserva» es irreversible. «¿Para quién?» lleva botón «Continuar».
- Lo elegido se conserva al volver; **cambiar una elección invalida lo que depende de ella**
  (p. ej. cambiar el servicio descarta la hora).

**Barra de progreso**
- Móvil (390): un segmento por paso; el actual (dorado) muestra el nombre del paso, el resto solo
  su número (completados dorado atenuado y tocables, futuros grises y no tocables). Sin línea
  «Paso X de N» debajo: ese texto va en el `aria-label`. Área táctil ≥ 44 px.
- Tableta y escritorio: etiquetas completas de cada paso (misma lógica, otra presentación).
- A la izquierda, flecha/X: en el paso 1 lleva a `/`; en los demás vuelve al paso anterior.
  La flecha nunca pregunta: al volver se conserva lo elegido. Sin popups en el flujo.

**Módulos**
- **Servicio:** tarjeta mínima (nombre, duración, precio), una por fila a todo el ancho.
  Lista vacía o fallo de la petición: mensaje con teléfono dentro del módulo (no 404). Estado «cargando».
- **Barbero:** tres tarjetas iguales (barbero 1, barbero 2, «Cualquiera»). «Cualquiera» lleva la
  línea «Barbero disponible». Sin foto: inicial en círculo dorado (placeholder; irán fotos reales);
  «Cualquiera» con icono genérico.
- **Fecha y hora:** `<select>` con los 14 días («Hoy · dom 11 oct»…); días sin huecos desactivados y
  marcados «· sin huecos»; primer día con huecos preseleccionado. Debajo, horas en cuadrícula de
  2 columnas (orden por filas: 1 2 / 3 4), sin títulos de grupo (el salto de horas ya separa mañana
  y tarde), solo «10:00», «10:30». Las horas ocupadas se muestran desactivadas (sin borde, texto atenuado).
  Con «Cualquiera», un hueco existe si al menos un barbero está libre. Estados: día sin huecos
  («Sin huecos este día»), ningún día con huecos («No hay huecos en los próximos 14 días. Llama al …»), cargando.
- **¿Para quién? (barbero y admin):** en la barra el segmento se llama «Cliente». Un campo «Cliente:»
  (teléfono o correo; si contiene `@`, es correo) y «Buscar». Con resultado: nombre y teléfono en línea,
  bajo el campo (nunca correo ni citas), y «Continuar»; si no es la persona, se busca de nuevo (editar el
  campo invalida el resultado). Sin resultado: «Cliente no encontrado» y formulario «Nuevo cliente» con
  nombre y teléfono (teléfono ya relleno si se buscó por teléfono). Nombre y teléfono obligatorios, con
  error bajo cada campo. Sin popups ni «Cambiar cliente». El botón «Continuar» va dentro del contenido
  (no fijo): el módulo crece y decrece según el estado.
- **Resumen:** tarjeta con servicio, barbero, día y hora (y cliente para el personal). Con «Cualquiera»:
  «Se te asignará al confirmar». Aviso dorado **debajo de los datos y encima del botón**, solo si la cita
  empieza en menos de 6 h: solo se puede cambiar o cancelar por teléfono (enlace `tel:`).
  Botón «Confirmar reserva» dentro de la tarjeta (no fijo); al pulsarlo pasa a «Reservando…» y se desactiva.
  Sin popups: la tarjeta es un único componente que cambia de contenido según el estado (normal, con aviso,
  reservando, fallo con «Reintentar», hora ocupada con «Elegir otra hora» y, al final, confirmada). Los avisos y
  errores sustituyen la zona del botón. Si la hora se ocupó entretanto, «Elegir otra hora» lleva al paso de
  fecha y hora con el mismo día seleccionado, conservando lo demás.
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

- FigJam: una sección por página, **solo con el frame móvil (390)** y sus componentes/estados. Tableta (768)
  y escritorio (1440) no se dibujan en wireframe: se resuelven en Figma Design (paso 5) con Auto Layout y
  ancho máximo. Los wireframes son esquemas de baja fidelidad: sin glow ni colores finales.
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
- Login con correo y contraseña en v1. Google, en v2 (la tabla de usuarios guardará `provider`; con Google el
  correo llegaría verificado, y haría falta una pantalla extra para pedir teléfono y aceptar la privacidad).
- Solo se reserva con sesión: sin sesión se redirige a `/acceso?volver=/reservar`
  y se vuelve al flujo. `volver` solo acepta rutas internas.
- Una sola vista `/mis-citas` para todos los roles; el backend devuelve más o menos
  citas según el rol. Los permisos (rol y propiedad) se comprueban en cada acción.
  Cita ajena o inexistente → 404 (no 403).
- Cancelar no borra: la cita pasa a `cancelled`, libera el hueco y se conserva.
- **Reglas de cancelación:** el cliente cancela hasta 6 h antes del inicio; pasado
  ese plazo debe contactar con la barbería. No se cancelan citas pasadas.
  Barbero y admin cancelan sin límite de antelación (propuesto, a confirmar).
- Horarios en el servicio `citas`. El admin los define con **selección múltiple de días** (calendario) y una o dos
  franjas; cada día se guarda como entrada individual y los días que ya tenían horario se sobrescriben. Un día sin
  horario no ofrece huecos. Tiene cargados al menos los próximos 14 días.
- Los turnos/plantillas (v2) serán un atajo que **genera días individuales** en la misma tabla,
  no una regla viva; cada día se modifica después por separado. El modelo de v1 no cambia.
- Cambiar un horario solo afecta a huecos futuros: las citas existentes se conservan.
  El sistema solo muestra el listado de citas en conflicto; el admin llama y cancela a mano.
- Clientes sin cuenta: la cita guarda `client_id` (opcional) o `guest_name` y
  `guest_phone`. Búsqueda de cliente por coincidencia exacta de teléfono o correo,
  solo barbero y admin. Los datos se eliminan cuando dejan de ser necesarios.
- El teléfono es obligatorio (registro y cliente sin cuenta).
- **El correo no es editable en v1:** se cambia llamando a la barbería (lo hace el admin).
- Cambio de contraseña desde `/panel` para todos los roles (pide la contraseña actual).
- Recuperación de contraseña: en v1 la resetea a mano el admin; el login lo indica con el teléfono.
- Servicios y precios: editados por script en v1.
- Tableta y escritorio en v1: el mismo módulo de móvil, centrado con ancho máximo (~480-560 px).
  Composiciones más complejas, en v2.
- **Protección de datos:** el responsable del tratamiento es el titular del negocio; el desarrollador actúa como
  encargado, con contrato de encargo (art. 28 RGPD). Orientación, no asesoramiento legal.

## Fases de entrega (modelo B: por fases, desplegando pronto)

| Fase | Contenido | Resultado visible |
|---|---|---|
| v1.0 | Landing, `/privacidad`, 404, despliegue completo | La web está online |
| v1.1 | `auth` (registro con teléfono, login, roles, cambio de contraseña), `/acceso`, `/panel` (datos y alta de barberos), primer admin por script | Entras con tu rol, editas tus datos y el admin crea barberos |
| v1.2 | `citas`: servicios, `/horarios` (calendario + conflictos), `/reservar` (con clientes sin cuenta), `/mis-citas` con cancelar | Se reserva y cancela con horarios reales, incluidos clientes por teléfono |
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
- Reasignar una cita en conflicto a otro barbero desde `/horarios` (en v1 solo se cancela).
- Cambio de correo por el propio usuario (requeriría verificación por correo).
- Calendarios de agenda completos (vista semana/mes) en `/mis-citas`.

## Por decidir

- Regla definitiva de asignación con «Cualquiera» (provisional: el primero de la lista libre).
- Confirmar con Yayah la duración real de corte+barba (45 min) y si le encaja perder 15 min por cita.
- Teléfono real de la barbería y fotos reales de los barberos.
- Cómo piden las API la disponibilidad: días con huecos de los 14 y horas de un día (se cierra al diseñar la API).
- Baja de barberos (desactivar, no borrar, para conservar el historial).
- Cambio obligatorio de la contraseña inicial del barbero en su primer acceso (propuesto: no obligatorio en v1;
  el admin la entrega en persona y se cambia desde `/panel`).
- Titular del negocio (autónomo o sociedad) y sus datos para la sección «Responsable»; hablarlo con el dueño
  y firmar el contrato de encargo antes de publicar.
- Texto de la política de privacidad (revisión legal; herramienta gratuita de la AEPD: Facilita RGPD) y aviso de
  cookies (depende de si usamos analítica o el mapa incrustado).
- Cómo habla el frontend con los 2 servicios (p. ej. Caddy enrutando /api/auth y /api/appointments).
- Cómo se entera `citas` de un barbero nuevo creado en `auth` (modelo de datos).
- Si barbero y admin cancelan sin límite de antelación (propuesto).
- Límite de citas cargadas en «Todas» de `/mis-citas` (se cierra al diseñar la API).
- Librería de fechas en el backend para Europe/Madrid (`date-fns-tz` o Luxon; el cálculo de huecos es código propio).

## Estado actual

- Fase: 0 (diseño y modelo de datos)
- Hecho: paleta, landing móvil con menú overlay en dos estados (paso 2 ✅), mapa del sitio y alcance de la v1
  (paso 3 ✅), **paso 4 ✅ (wireframes móvil en FigJam de todas las pantallas de la v1)**: `/reservar`, `/acceso`,
  `/panel`, `/mis-citas`, `/horarios`, `/privacidad` y 404.
- Lienzo de FigJam: 00 Marca, 01 Landing Page, 02 Login / Registro, 03 Reservar cita, 04 Panel, 05 Mis citas,
  07 Horarios, 08 Privacidad - 404, más `Mapa del Sitio`. La 06 queda sin usar (Modificar cita, v2).
- Pendiente de pulir al pasar a Figma (no bloquea):
  - `/reservar`: botón «Confirmar reserva» (en el wireframe aún «Confirmar»); confirmación con icono, «Reserva
    confirmada», «Ver mis citas» / «Volver al inicio» (personal: «Reservar otra cita» / «Ver agenda»); «Reintentar»
    y «Elegir otra hora» dentro de la tarjeta; estados «cargando» de Servicio y Fecha y hora; selector de día
    desplegado con un día «· sin huecos»; tildes («Día», «está»); tarjetas y botones a ancho completo con margen 16 px.
  - `/acceso`: casilla con el texto «He leído y acepto la política de privacidad» (enlace); campo Teléfono en el
    registro; estados «Entrando…» y «Creando cuenta…»; ver/ocultar contraseña; anchos de campo.
  - `/panel`: botón «Guardar» también en el panel de admin (aún dice «Modificar»); correo como texto plano sin caja;
    fragmentos «Contraseña cambiada» y «Contraseña actual incorrecta».
  - `/mis-citas`: fragmento del botón de cancelar desactivado (cita a menos de 6 h, con teléfono); estado «cargando»;
    botones del modal («Mantener cita» / «Sí, cancelar»); «Cancelada» / «Realizada».
  - `/horarios`: botones de los modales de quitar («Sí, quitar», «Quitando…»), barbero en el texto del modal;
    título del fragmento de conflictos («Conflictos tras guardar o quitar») con su línea de aviso; barbero en
    lugar de «Vacío» y «Fuera de horario»; quitar el botón «Panel Admin» de la lista vacía; un solo listado por día.
  - `/privacidad` y 404: frames más grandes (unificados en una sola sección); textos y datos del responsable.
- Pendiente de decidir en Figma: colores de estado (éxito, error, aviso; el verde y el rojo del wireframe
  no están en la paleta; el error no debe depender solo del color), glow real del elemento seleccionado,
  tipografías, estilado del calendario `react-day-picker`, y la barra de 5 pasos en 768.
- Siguiente: paso 5 → sistema de diseño en Figma Design (Variables, Auto Layout, componentes con variantes),
  empezando por Variables de color y tipografía; luego componentes (incluido `ConfirmDialog`) y frames de 768/1440.

## Convenciones

- Commits: Conventional Commits (feat:, fix:, docs:)
- Ramas: feature/nombre-corto, PR hacia main
- Código en inglés, comentarios y docs en español
