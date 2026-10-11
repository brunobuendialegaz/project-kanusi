# Mapa del sitio y rutas

> Estado: **cerrado en el paso 3** (fase 0). Las filas marcadas con 🟡 siguen pendientes de decisión.

## Convenciones

- Las rutas visibles (URL) están en **español**; componentes, archivos y variables, en **inglés**.
- Los `id` de las anclas van en minúscula y en español (`#servicios`), porque aparecen en la URL.
- Las rutas de la API (backend) no se rigen por esta tabla; se definirán aparte.

## Roles

| Rol | Descripción |
|---|---|
| Visitante | Sin sesión iniciada |
| Cliente | Usuario registrado |
| Barbero | Atiende citas que tiene asignadas |
| Barbero admin | Barbero con acceso a todas las citas y a la gestión del negocio |

> **Cliente sin cuenta:** no es un rol ni un usuario. Es una persona que reserva por teléfono; su cita guarda nombre y teléfono, y la crea un barbero o el admin.

## Páginas

| Ruta | Quién entra | Para qué sirve | Cómo se llega |
|---|---|---|---|
| `/` | Todos | Landing: servicios, barberos y contacto | URL directa; logo del header |
| `/#servicios`, `/#barberos`, `/#contacto` | Todos | Anclas dentro de la landing | Menú |
| `/reservar` | Solo con sesión | Flujo por pasos: (solo barbero y admin: ¿para quién?) → servicio → barbero → fecha/hora → confirmación | Botón RESERVAR; menú con sesión |
| `/acceso` | Sin sesión (con sesión redirige a `/mis-citas`) | Login y registro (dos estados) | Menú «Login» o redirección desde una ruta protegida |
| `/panel` | Cliente, barbero y barbero admin | Cuenta: editar nombre, correo y teléfono. Admin, además: alta de barberos y botón a `/horarios` | Menú con sesión («Mi cuenta») |
| `/mis-citas` | Cliente, barbero y barbero admin | Lista de citas ordenada por fecha, según el rol; permite cancelar | Menú con sesión («Mis citas» / «Agenda»); botón en `/panel` |
| `/horarios` | Solo barbero admin | Definir el horario de cada barbero día a día (fecha y franjas); tras guardar, listado de citas en conflicto | Botón en el bloque de gestión de `/panel` |
| `/privacidad` | Todos | Política de privacidad: qué datos se guardan, para qué y cómo ejercer los derechos (RGPD) | Enlace en el `footer`; casilla de aceptación en el registro de `/acceso` |
| `*` (404) | Todos | Página no encontrada, con enlace a la landing | URL mal escrita |

## Reglas de acceso

- **Rutas protegidas:** `/reservar`, `/panel`, `/mis-citas` y `/horarios`. Sin sesión, se redirige a `/acceso?volver=<ruta>` (la ruta va codificada).
- **Rutas por rol:** `/horarios` es solo para el barbero admin. Para otros roles, el backend rechaza la petición y el frontend muestra la pantalla 404.
- **Redirección tras el login:** se vuelve a la ruta indicada en `volver`; si no hay, a `/mis-citas`.
- **Seguridad de `volver`:** solo se aceptan rutas internas (empiezan por `/`), para evitar redirecciones a sitios externos.
- **Cita inexistente o ajena:** el backend responde 404 (no 403) y el frontend muestra la pantalla 404.
- **Registro:** exige marcar la casilla de aceptación de la política de privacidad (`/privacidad`), porque se guardan datos personales, y un teléfono. El registro público siempre crea un cliente.
- **Cambio de correo:** exige la contraseña actual (sin verificación por correo en v1).

## Menú de navegación (overlay en móvil)

| Estado | Entradas |
|---|---|
| Sin sesión | Servicios `/#servicios`, Barberos `/#barberos`, Contacto `/#contacto`, Login `/acceso` |
| Con sesión | Servicios, Barberos, Contacto, Reservar `/reservar`, Mis citas `/mis-citas` (barbero y admin: «Agenda»), Mi cuenta `/panel`, Logout |

- Las anclas usan ruta absoluta (`/#servicios`) para funcionar desde cualquier página.
- **Logout** es una acción, no un enlace: borra la sesión y redirige a `/`. En React, un `<button>` con `onClick`.
- Cada entrada se añade al menú cuando existe su ruta (según la fase).

## Reparto por servicio

| Pantalla | Servicio |
|---|---|
| `/acceso`, `/panel` | `auth` (usuarios, datos personales, roles, alta de barberos) |
| `/reservar`, `/mis-citas`, `/horarios` | `citas` (servicios, barberos, horarios, citas) |

> La búsqueda de un cliente por teléfono o email (solo barbero y admin, coincidencia exacta) vive en `auth`, y `citas` la consulta.

## Visibilidad y permisos de citas

La vista `/mis-citas` es la misma para todos los roles, con componentes que varían según el rol. El backend decide qué citas devuelve:

| Rol | Citas devueltas | Puede crear | Puede cancelar |
|---|---|---|---|
| Cliente | Las suyas | Para sí mismo | Las suyas, hasta 6 h antes |
| Barbero | Las que tiene asignadas | Para cualquier cliente, con o sin cuenta 🟡 en su propia agenda | Las que tiene asignadas, sin límite de antelación 🟡 |
| Barbero admin | Todas | Para cualquier cliente y cualquier barbero | Todas, sin límite de antelación 🟡 |

- Los permisos (rol y propiedad de la cita) se comprueban en el backend en **cada acción**, no solo al listar.

## Reglas de cancelación

- **Cancelar no borra:** la cita pasa a estado `cancelled`, libera el hueco y se conserva en el historial. Se guarda quién la canceló y cuándo.
- **Cliente:** puede cancelar hasta **6 horas antes** del inicio de la cita. Pasado ese plazo, el botón aparece desactivado con un mensaje que invita a contactar con la barbería (WhatsApp o teléfono).
- **Barbero y admin:** pueden cancelar sin límite de antelación 🟡, porque son quienes resuelven imprevistos y conflictos de horario.
- **Citas pasadas:** no se pueden cancelar.
- **La regla se valida en el backend**, con la hora del servidor (zona Europe/Madrid). El frontend solo desactiva el botón por comodidad.
- Un cliente sin cuenta no cancela por la web: lo hace el barbero o el admin.

## Clientes sin cuenta

- La cita guarda `client_id` (opcional) o, si no hay cuenta, `guest_name` y `guest_phone`.
- El barbero o el admin busca primero por teléfono o email completos; si existe, asocia la cita a ese usuario; si no, introduce nombre y teléfono.
- Sus datos se eliminan cuando dejan de ser necesarios, y la política de privacidad lo recoge.

## Horarios y conflictos

- El admin define el horario de cada barbero **día a día** desde `/horarios`: elige barbero y fecha, y añade una o varias franjas (inicio y fin). Ejemplo: lunes 5, de 10:00 a 14:00 y de 16:00 a 20:00.
- **Un día sin horario no ofrece huecos.** Esto cubre también enfermedades, vacaciones y festivos: basta con no definir ese día.
- Un cambio de horario solo afecta a huecos futuros: **las citas ya reservadas se conservan**.
- Al guardar, se muestra el listado de citas en conflicto (fecha, hora, barbero, nombre y teléfono del cliente). El admin llama por teléfono y cancela a mano. El sistema no cancela ni avisa solo.
- Más adelante se podrá agilizar (copiar una semana, plantillas), pero no entra en la v1.2.

## Alta de barberos

- Solo el admin, desde `/panel`: nombre, correo, teléfono y contraseña inicial. El backend asigna el rol.
- El primer admin se crea con script.

## Pendiente de decidir

- 🟡 Antelación mínima para reservar y hasta cuántos días vista se puede reservar.
- 🟡 Intervalo de los huecos (15 o 30 minutos).
- 🟡 Si barbero y admin cancelan sin límite de antelación (propuesto).
- 🟡 Si un barbero puede crear citas en la agenda de otro barbero.
- 🟡 Baja de barberos (desactivar, no borrar, para conservar el historial).
- 🟡 Cambio obligatorio de la contraseña inicial del barbero en su primer acceso.
- 🟡 Agrupación de la lista: por días o por semanas (se decide al diseñar `/mis-citas`).
- 🟡 Redactar el texto de la política de privacidad (revisión legal) y decidir si hace falta aviso de cookies (depende de si usamos analítica o mapas incrustados).
- 🟡 Verificación del correo al cambiarlo (requeriría envío de correo).
- 🟡 Cómo se entera `citas` de un barbero nuevo creado en `auth` (modelo de datos).

## Aplazado a v2

- `/modificar-cita/:id`: en v1 se cancela la cita y se vuelve a reservar.
- Login con Google.
- Recuperación de contraseña (en v1 la resetea a mano el barbero admin).
- Edición de servicios y precios desde la web (en v1, por script).
- Agilizar la carga de horarios (copiar semanas, plantillas).
