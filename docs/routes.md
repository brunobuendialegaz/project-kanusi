# Mapa del sitio y rutas

> Estado: **borrador**, paso 3 (fase 0). Las filas marcadas con 🟡 siguen pendientes de decisión.

## Convenciones

- Las rutas visibles (URL) están en **español**; componentes, archivos y variables, en **inglés**.
- Los `id` de las anclas van en minúscula y en español (`#servicios`), porque aparecen en la URL.
- Las rutas de la API (backend) no se rigen por esta tabla; se definirán aparte.

## Roles

| Rol           | Descripción                                                     |
| ------------- | --------------------------------------------------------------- |
| Visitante     | Sin sesión iniciada                                             |
| Cliente       | Usuario registrado                                              |
| Barbero       | Atiende citas que tiene asignadas                               |
| Barbero admin | Barbero con acceso a todas las citas y a la gestión del negocio |

## Páginas

| Ruta                                      | Quién entra                                                              | Para qué sirve                                                                            | Cómo se llega                                               |
| ----------------------------------------- | ------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------- | ----------------------------------------------------------- |
| `/`                                       | Todos                                                                    | Landing: servicios, barberos y contacto                                                   | URL directa; logo del header                                |
| `/#servicios`, `/#barberos`, `/#contacto` | Todos                                                                    | Anclas dentro de la landing                                                               | Menú                                                        |
| `/reservar`                               | Solo con sesión                                                          | Flujo por pasos: servicio → barbero → fecha/hora → confirmación                           | Botón RESERVAR; menú con sesión                             |
| `/acceso`                                 | Sin sesión (con sesión redirige a `/mis-citas`)                          | Login y registro (dos estados)                                                            | Menú «Login» o redirección desde una ruta protegida         |
| `/panel`                                  | Cliente, barbero y barbero admin                                         | Cuenta: editar nombre, correo y teléfono; el admin, además, ajustes de gestión (a futuro) | Menú con sesión («Mi cuenta»)                               |
| `/mis-citas`                              | Cliente, barbero y barbero admin                                         | Lista de citas ordenada por fecha, según el rol                                           | Menú con sesión («Mis citas» / «Agenda»); botón en `/panel` |
| `/modificar-cita/:id`                     | Cliente, barbero y barbero admin (solo si tienen permiso sobre esa cita) | Cambiar o cancelar una cita                                                               | Botón en cada cita de `/mis-citas`                          |
| `*` (404)                                 | Todos                                                                    | Página no encontrada, con enlace a la landing                                             | URL mal escrita                                             |

## Reglas de acceso

- **Rutas protegidas:** `/reservar`, `/panel`, `/mis-citas` y `/modificar-cita/:id`. Sin sesión, se redirige a `/acceso?volver=<ruta>` (la ruta va codificada).
- **Redirección tras el login:** se vuelve a la ruta indicada en `volver`; si no hay, a `/mis-citas`.
- **Seguridad de `volver`:** solo se aceptan rutas internas (empiezan por `/`), para evitar redirecciones a sitios externos.
- **Cita inexistente o ajena:** el backend responde 404 (no 403) y el frontend muestra la pantalla 404.
- **Cambio de correo:** exige la contraseña actual (sin verificación por correo en v1).

## Menú de navegación (overlay en móvil)

| Estado     | Entradas                                                                                                                            |
| ---------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| Sin sesión | Servicios `/#servicios`, Barberos `/#barberos`, Contacto `/#contacto`, Login `/acceso`                                              |
| Con sesión | Servicios, Barberos, Contacto, Reservar `/reservar`, Mis citas `/mis-citas` (barbero y admin: «Agenda»), Mi cuenta `/panel`, Logout |

- Las anclas usan ruta absoluta (`/#servicios`) para funcionar desde cualquier página.
- **Logout** es una acción, no un enlace: borra la sesión y redirige a `/`. En React, un `<button>` con `onClick`.

## Reparto por servicio

| Pantalla                                         | Servicio                                   |
| ------------------------------------------------ | ------------------------------------------ |
| `/acceso`, `/panel`                              | `auth` (usuarios, datos personales, roles) |
| `/reservar`, `/mis-citas`, `/modificar-cita/:id` | `citas` (servicios, barberos, citas)       |

## Visibilidad y permisos de citas

La vista `/mis-citas` es la misma para todos los roles, con componentes que varían según el rol. El backend decide qué citas devuelve:

| Rol           | Citas devueltas         | Puede modificar o cancelar |
| ------------- | ----------------------- | -------------------------- |
| Cliente       | Las suyas               | Las suyas                  |
| Barbero       | Las que tiene asignadas | Las que tiene asignadas    |
| Barbero admin | Todas                   | Todas                      |

> Los permisos (rol y propiedad de la cita) se comprueban en el backend en **cada acción**, no solo al listar.

## Pendiente de decidir

- 🟡 Página de política de privacidad (`/privacidad`), por RGPD.
- 🟡 Reglas de cancelación (antelación mínima, citas pasadas).
- 🟡 Agrupación de la lista: por días o por semanas (se decide al diseñar `/mis-citas`).
- 🟡 Verificación del correo al cambiarlo (requeriría envío de correo).

## Aplazado a v2

- Login con Google.
- Recuperación de contraseña (en v1 la resetea a mano el barbero admin).
- Ajustes de gestión del admin en `/panel` (servicios, barberos...).
