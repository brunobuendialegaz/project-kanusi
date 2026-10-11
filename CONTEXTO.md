# Barbería Kanusi — Contexto del proyecto

## Objetivo

Web real para una barbería de pueblo (estilo underground) y proyecto de aprendizaje
para mí (2º DAW). Debe estar online 365 días al año.

## Nombre

- Negocio: **Kanusi**. (Yayah es el nombre del barbero, no del negocio.)

## Stack

- Frontend: React + Vite + Tailwind (enrutado con React Router: propuesto)
- Backend: Node.js + NestJS, 2 microservicios:
  - `auth`: usuarios, registro, login y roles (cliente / barbero)
  - `citas`: servicios y precios, barberos, citas y agenda
- BD: PostgreSQL (una base por servicio; a confirmar en el modelo de datos)
- Despliegue: VPS con Docker Compose + Caddy (HTTPS), CI/CD con GitHub Actions
- Repo: monorepo (frontend/, services/ con auth y citas, docs/, docker-compose.yml)

## Diseño

- Herramientas: FigJam (wireframes) → Figma Design (diseño final con Variables,
  Auto Layout, componentes con variantes y Dev Mode/MCP para pasar a React + Tailwind)
- Paleta: #dccc8d (acento dorado), #b5b5b5, #ffffff, #616060, #0c0c0c
- Mobile-first. Frames de referencia: 390 (móvil) / 768 (tableta) / 1440 (escritorio)
- Lienzo de FigJam: secciones 00 Marca, 01 Landing, 02 Reservar cita, 03 Mis citas,
  03.1 Modificar cita, 05 Login / Registro, 06 Panel del barbero
  (03, 03.1 y 06 pueden reorganizarse tras el paso 3, porque `/panel` es una sola ruta)
- Convención en wireframes: cada enlace del menú lleva su `href` al lado
  (sin flechas); anclas marcadas con su `id` a la izquierda del bloque.
- Jerarquía de títulos: `h1` = frase del hero (uno por página), `h2` = título de
  cada sección, `h3` = cada servicio / cada barbero.

## Funcionalidades v1 (pendiente de cerrar en el paso 3)

- [ ] Ver servicios y precios (con duración)
- [ ] Reservar cita (servicio → barbero → fecha/hora → confirmación)
- [ ] Registro y login de clientes
- [ ] Panel del cliente: ver sus citas y editar sus datos (correo, teléfono, etc.)
- [ ] Modificar/cancelar cita (cliente y barbero)
- [ ] Panel del barbero (ver agenda)
- Contacto: botones email y WhatsApp (sin formulario → sin backend de correo)

## Decisiones tomadas

- FigJam: una sección por página (numerada 00-06), con móvil/tableta/escritorio
  dentro, de menor a mayor.
- Stack cambiado de Angular + Spring Boot a React + NestJS (en fase 0, sin código).
- Frontend con Vite.
- 2 microservicios: `auth` y `citas`.
- "Modificar cita" es una única pantalla compartida: el cliente entra desde
  su panel y el barbero desde el suyo. Los permisos se resuelven en backend.
- Landing en móvil (paso 2 cerrado): bloques `header`, `hero`, `services`, `barbers`,
  `contact` (horarios + ubicación + email/WhatsApp + mapa, unificados) y `footer`.
  Los `id` van en minúscula.
- Sin formulario de contacto en v1.
- Menú hamburguesa = overlay. Sin sesión: Servicios (`#services`), Barberos (`#barbers`),
  Contacto (`#contact`), Login. Con sesión: Panel y Reservar.
- Idioma: código, archivos y componentes en inglés; textos visibles de la web en español.
- `/panel` es una única ruta protegida que muestra contenido según el rol
  (renderizado condicional): el cliente ve sus citas y puede cambiar sus datos;
  el barbero ve su panel con la agenda. El backend manda en los permisos.

## Propuesta pendiente de confirmar (paso 3): páginas y rutas

Criterio: ruta propia si es un flujo de varios pasos, necesita enlace/botón atrás
o protección por login/rol. Panel (modal/drawer) si es una acción corta sobre algo
que ya se está viendo.

- `/` Landing: una sola página con secciones. El menú enlaza a ellas.
- `/reservar`: flujo por pasos, ruta propia (el botón RESERVAR de la landing lleva aquí).
- `/acceso`: login y registro en una misma página con dos estados.
- `/panel`: protegida (cliente y barbero, contenido distinto por rol).
- Modificar cita: panel lateral/modal reutilizable (pantalla completa en móvil),
  sin ruta propia (a confirmar).

## Por decidir

- Idioma de las rutas (propuesta: español; el wireframe aún usa `/login`).
- ¿"Mis citas" es una pestaña dentro de `/panel` o ruta aparte?
- Reservar sin sesión: ¿login antes de empezar o al final del flujo?
- Cómo habla el frontend con los 2 servicios (p. ej. Caddy enrutando /api/auth y /api/citas).
- Dónde viven los barberos: usuario en `auth`, perfil y servicios en `citas`.

## Estado actual

- Fase: 0 (diseño y modelo de datos)
- Hecho: paleta, landing móvil completa con menú overlay y jerarquía de títulos
  (paso 2 ✅), lienzo organizado en secciones 00-06 (02-06 aún son copias de la landing)
- Siguiente: paso 3 → páginas, flujos y alcance de la v1

## Convenciones

- Commits: Conventional Commits (feat:, fix:, docs:)
- Ramas: feature/nombre-corto, PR hacia main
- Código en inglés, comentarios y docs en español
