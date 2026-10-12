# Mapa del sitio y rutas

> Estado: **cerrado en el paso 3** (fase 0); flujo de `/reservar` ampliado en el **paso 4**. Las filas marcadas con 🟡 siguen pendientes de decisión.

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
| `/reservar` | Solo con sesión | Flujo de un módulo visible a la vez (ver «Flujo de `/reservar`»). Cliente: servicio → barbero → fecha/hora → confirmar. Barbero: ¿para quién? → servicio → fecha/hora → confirmar. Admin: ¿para quién? → servicio → barbero → fecha/hora → confirmar | Botón RESERVAR; menú con sesión |
| `/acceso` | Sin sesión (con sesión redirige a `/mis-citas`) | Login y registro (dos estados) | Menú «Login» o redirección desde una ruta protegida |
| `/panel` | Cliente, barbero y barbero admin | Cuenta: editar nombre, correo y teléfono. Admin, además: alta de barberos y botón a `/horarios` | Menú con sesión («Mi cuenta») |
| `/mis-citas` | Cliente, barbero y barbero admin | Lista de citas ordenada por fecha, según el rol; permite cancelar | Menú con sesión («Mis citas» / «Agenda»); botón en `/panel`; botones finales de `/reservar` |
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
- **Logout** es una acción, no un enlace: borra la sesión (y lo guardado de `/reservar`) y redirige a `/`. En React, un `<button>` con `onClick`.
- Cada entrada se añade al menú cuando existe su ruta (según la fase).
- `/reservar` **no muestra el menú**: arriba solo lleva su barra de progreso, y abajo el footer.

## Flujo de `/reservar`

Una sola ruta. Un componente guarda en el estado el paso y lo elegido, y muestra un único módulo a la vez.

### Pasos según rol

| Rol | Pasos |
|---|---|
| Cliente | 1 Servicio → 2 Barbero → 3 Fecha y hora → 4 Confirmar |
| Barbero | 1 ¿Para quién? → 2 Servicio → 3 Fecha y hora → 4 Confirmar (barbero fijo en él mismo, no se muestra) |
| Barbero admin | 1 ¿Para quién? → 2 Servicio → 3 Barbero → 4 Fecha y hora → 5 Confirmar |

### Comportamiento

- **Servicio y barbero avanzan solos** al tocar. **¿Para quién?** lleva botón «Continuar». Elegir hora lleva al resumen, que no reserva nada. Solo «Confirmar reserva» es irreversible.
- Lo elegido se conserva al volver; **cambiar una elección invalida lo que depende de ella** (p. ej. cambiar el servicio descarta la hora).
- **Barra de progreso:** móvil, un segmento por paso con el nombre solo en el actual y número en el resto (completados tocables, futuros no; áreas táctiles ≥ 44 px; «Paso X de N» en el `aria-label`); tableta y escritorio, etiquetas completas. La flecha/X de la izquierda lleva a `/` en el paso 1 y al paso anterior en los demás. Salir con la X no pregunta, salvo en el resumen.
- **Barbero:** tres tarjetas (barbero 1, barbero 2, «Cualquiera»). «Cualquiera» = «Barbero disponible»; el backend asigna un barbero libre y la confirmación muestra cuál.
- **Fecha y hora:** `<select>` con los 14 días (sin huecos: desactivado y marcado), primer día con huecos preseleccionado, horas en 2 columnas por filas, sin títulos de grupo; las ocupadas se muestran desactivadas. Con «Cualquiera», hay hueco si al menos un barbero está libre.
- **¿Para quién?:** búsqueda exacta por teléfono o correo (un solo campo; con `@` es correo). Con resultado: nombre y teléfono en línea y «Continuar»; si no es la persona, se busca de nuevo. Sin resultado: «Cliente no encontrado» y formulario con nombre y teléfono obligatorios. En la barra, el segmento se llama «Cliente».
- **Resumen:** aviso «solo por teléfono» cuando la cita empieza en menos de 6 h, debajo de los datos y encima del botón. Doble toque protegido («Reservando…»). Si la hora se ocupó entretanto, vuelve a fecha y hora con el mismo día seleccionado.
- **Confirmación (estado final, no es ruta):** cliente → «Ver mis citas» (`/mis-citas`) y «Volver al inicio» (`/`). Personal → «Reservar otra cita» (reinicio limpio en «¿Para quién?») y «Ver agenda» (`/mis-citas`).
- **Tableta y escritorio (v1):** el mismo módulo centrado con ancho máximo (~480-560 px).

### Persistencia

- `sessionStorage`: IDs de servicio, barbero, día, hora y paso actual. **Nunca** datos del cliente del personal.
- Se revalida al restaurar; se limpia al confirmar, al salir desde el paso 1 y en el logout.
- El personal que recarga repite «¿Para quién?» y salta después al paso donde estaba.

### Reglas de reserva (validadas en backend, hora del servidor, Europe/Madrid)

- Antelación mínima del cliente: **30 min**; barbero y admin, sin mínimo.
- Ventana: **14 días** vista.
- Huecos cada **30 min**; un hueco vale si el servicio entero cabe sin pisar otra cita. Duraciones: corte 30 min, corte+barba 45 min (guardadas en el servicio).
- Barbero crea solo en su agenda; admin, en cualquiera.
- Cita idéntica del mismo cliente a la misma hora: rechazada.

## Reparto por servicio

| Pantalla | Servicio |
|---|---|
| `/acceso`, `/panel` | `auth` (usuarios, datos personales, roles, alta de barberos) |
| `/reservar`, `/mis-citas`, `/horarios` | `citas` (servicios, barberos, horarios, citas) |

> La búsqueda de un cliente por teléfono o email (solo barbero y admin, coincidencia exacta) vive en `auth`, y `citas` la consulta. El resultado devuelve solo nombre y teléfono.

## Visibilidad y permisos de citas

La vista `/mis-citas` es la misma para todos los roles, con componentes que varían según el rol. El backend decide qué citas devuelve:

| Rol | Citas devueltas | Puede crear | Puede cancelar |
|---|---|---|---|
| Cliente | Las suyas | Para sí mismo (mínimo 30 min de antelación) | Las suyas, hasta 6 h antes |
| Barbero | Las que tiene asignadas | Para cualquier cliente, con o sin cuenta, **solo en su propia agenda**, sin mínimo de antelación | Las que tiene asignadas, sin límite de antelación 🟡 |
| Barbero admin | Todas | Para cualquier cliente y cualquier barbero, sin mínimo de antelación | Todas, sin límite de antelación 🟡 |

- Los permisos (rol y propiedad de la cita) se comprueban en el backend en **cada acción**, no solo al listar.

## Reglas de cancelación

- **Cancelar no borra:** la cita pasa a estado `cancelled`, libera el hueco y se conserva en el historial. Se guarda quién la canceló y cuándo.
- **Cliente:** puede cancelar hasta **6 horas antes** del inicio de la cita. Pasado ese plazo, el botón aparece desactivado con un mensaje que invita a contactar con la barbería (WhatsApp o teléfono). Una cita reservada con menos de 6 h de antelación nace ya sin cancelación web, y la confirmación lo avisa.
- **Barbero y admin:** pueden cancelar sin límite de antelación 🟡, porque son quienes resuelven imprevistos y conflictos de horario.
- **Citas pasadas:** no se pueden cancelar.
- **La regla se valida en el backend**, con la hora del servidor (zona Europe/Madrid). El frontend solo desactiva el botón por comodidad.
- Un cliente sin cuenta no cancela por la web: lo hace el barbero o el admin.

## Clientes sin cuenta

- La cita guarda `client_id` (opcional) o, si no hay cuenta, `guest_name` y `guest_phone`.
- El barbero o el admin busca primero por teléfono o email completos; si existe, confirma que es esa persona y asocia la cita a ese usuario; si no, introduce nombre y teléfono.
- Sus datos se eliminan cuando dejan de ser necesarios, y la política de privacidad lo recoge.

## Horarios y conflictos

- El admin define el horario de cada barbero **día a día** desde `/horarios`: elige barbero y fecha, y añade una o varias franjas (inicio y fin). Ejemplo: lunes 5, de 10:00 a 14:00 y de 16:00 a 20:00.
- **Un día sin horario no ofrece huecos.** Esto cubre también enfermedades, vacaciones y festivos: basta con no definir ese día.
- Como se reserva hasta 14 días vista, el admin debe tener cargados al menos los próximos 14 días.
- Un cambio de horario solo afecta a huecos futuros: **las citas ya reservadas se conservan**.
- Al guardar, se muestra el listado de citas en conflicto (fecha, hora, barbero, nombre y teléfono del cliente). El admin llama por teléfono y cancela a mano. El sistema no cancela ni avisa solo.
- **Turnos (v2):** un turno es un atajo que genera días individuales (p. ej. L-V, 10-14 y 17-21). Después cada día se modifica por separado. Escribe en la misma tabla que el día a día.

## Alta de barberos

- Solo el admin, desde `/panel`: nombre, correo, teléfono y contraseña inicial. El backend asigna el rol.
- El primer admin se crea con script.

## Pendiente de decidir

- 🟡 Regla definitiva de asignación con «Cualquiera» (provisional: el primero de la lista que esté libre).
- 🟡 Confirmar con Yayah la duración real de corte+barba (45 min).
- 🟡 Teléfono real de la barbería y fotos de los barberos (placeholders de momento).
- 🟡 Cómo piden las API la disponibilidad (días con huecos de los 14 y horas de un día).
- 🟡 Si barbero y admin cancelan sin límite de antelación (propuesto).
- 🟡 Baja de barberos (desactivar, no borrar, para conservar el historial).
- 🟡 Cambio obligatorio de la contraseña inicial del barbero en su primer acceso.
- 🟡 Agrupación de la lista: por días o por semanas (se decide al diseñar `/mis-citas`).
- 🟡 Redactar el texto de la política de privacidad (revisión legal) y decidir si hace falta aviso de cookies (depende de si usamos analítica o mapas incrustados).
- 🟡 Verificación del correo al cambiarlo (requeriría envío de correo).
- 🟡 Cómo se entera `citas` de un barbero nuevo creado en `auth` (modelo de datos).
- 🟡 (v2) Qué pasa al aplicar un turno sobre un día que ya tiene horario.

## Aplazado a v2

- `/modificar-cita/:id`: en v1 se cancela la cita y se vuelve a reservar.
- Login con Google.
- Recuperación de contraseña (en v1 la resetea a mano el barbero admin).
- Edición de servicios y precios desde la web (en v1, por script).
- Turnos/plantillas de horario.
- Botón «lo antes posible» dedicado.
- Añadir la cita al calendario del móvil (`.ics`).
- Composiciones propias para tableta y escritorio en `/reservar`.
- Que un barbero cree citas en la agenda de otro.
- Fotos reales de los barberos.
