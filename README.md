<div align="center">

# ✂️ Kanusi

**Web y sistema de citas para una barbería de pueblo con estilo underground.**

*Lo que no se negocia: el estilo.*

![Estado](https://img.shields.io/badge/estado-en%20dise%C3%B1o-dccc8d?style=flat-square&labelColor=0c0c0c)
![Fase](https://img.shields.io/badge/fase-0%20%C2%B7%20dise%C3%B1o%20y%20modelo%20de%20datos-616060?style=flat-square&labelColor=0c0c0c)
![Licencia](https://img.shields.io/badge/licencia-por%20definir-b5b5b5?style=flat-square&labelColor=0c0c0c)

![React](https://img.shields.io/badge/React-0c0c0c?style=flat-square&logo=react&logoColor=61DAFB)
![Vite](https://img.shields.io/badge/Vite-0c0c0c?style=flat-square&logo=vite&logoColor=646CFF)
![Tailwind CSS](https://img.shields.io/badge/Tailwind-0c0c0c?style=flat-square&logo=tailwindcss&logoColor=06B6D4)
![NestJS](https://img.shields.io/badge/NestJS-0c0c0c?style=flat-square&logo=nestjs&logoColor=E0234E)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-0c0c0c?style=flat-square&logo=postgresql&logoColor=4169E1)
![Docker](https://img.shields.io/badge/Docker-0c0c0c?style=flat-square&logo=docker&logoColor=2496ED)
![Caddy](https://img.shields.io/badge/Caddy-0c0c0c?style=flat-square&logo=caddy&logoColor=1F88C0)
![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-0c0c0c?style=flat-square&logo=githubactions&logoColor=2088FF)

</div>

---

## Sobre el proyecto

Kanusi es una web real para una barbería de pueblo, pensada para estar **online los 365 días del año**. Permite consultar servicios y precios, reservar cita y gestionar la agenda, y a la vez es un proyecto de aprendizaje de desarrollo full stack construido de principio a fin por una sola persona.

El diseño parte de dos ideas:

- **Mobile-first.** La mayoría de clientes entrará desde el móvil.
- **Nadie se queda fuera.** Quien prefiere llamar por teléfono también tiene su cita en la agenda: el barbero la crea por él.

## Funcionalidades (v1)

- Landing con servicios, precios, barberos, horarios y ubicación.
- Registro y login con correo y contraseña.
- Reserva de cita por pasos: servicio → barbero → fecha y hora → confirmación.
- Citas para **clientes sin cuenta**, creadas por el barbero o el administrador.
- Lista de citas según el rol, con cancelación (el cliente puede cancelar hasta 6 h antes).
- Gestión de horarios día a día por el administrador, con listado de citas en conflicto.
- Alta de barberos desde el panel del administrador.
- Política de privacidad y aceptación explícita en el registro (RGPD).

## Roles

| Rol | Qué puede hacer |
|---|---|
| **Visitante** | Ver la landing y la política de privacidad |
| **Cliente** | Reservar, ver y cancelar sus citas, editar sus datos |
| **Barbero** | Gestionar las citas que tiene asignadas y crear citas para otros clientes |
| **Barbero admin** | Todo lo anterior, ver todas las citas, definir horarios y dar de alta barberos |

## Arquitectura

```mermaid
flowchart LR
    U([Navegador]) --> C[Caddy<br/>HTTPS]
    C --> F[Frontend<br/>React + Vite]
    C --> A[Servicio auth<br/>NestJS]
    C --> P[Servicio citas<br/>NestJS]
    A --> DA[(PostgreSQL<br/>auth)]
    P --> DP[(PostgreSQL<br/>citas)]
    P -. consulta clientes .-> A
```

| Servicio | Responsabilidad |
|---|---|
| `auth` | Usuarios, registro, login, roles, alta de barberos y búsqueda exacta de clientes |
| `citas` | Servicios y precios, barberos, horarios, citas y agenda |

Cada servicio tiene su propia base de datos. Todo se despliega en un VPS con Docker Compose, con HTTPS gestionado por Caddy y CI/CD con GitHub Actions.

## Hoja de ruta

| Fase | Contenido | Estado |
|---|---|---|
| **v0** | Diseño, mapa del sitio y modelo de datos | 🟡 En curso |
| **v1.0** | Landing, política de privacidad, 404 y despliegue completo | ⬜ Pendiente |
| **v1.1** | `auth`: registro, login, roles, `/panel` y alta de barberos | ⬜ Pendiente |
| **v1.2** | `citas`: servicios, horarios, reserva y cancelación, clientes sin cuenta | ⬜ Pendiente |
| **v1.3** | Mejoras y pulido | ⬜ Pendiente |
| **v2** | Login con Google, modificar cita, recuperar contraseña, edición de servicios desde la web | 💡 Ideas |

## Estructura del repositorio

```text
kanusi/
├── frontend/              # React + Vite + Tailwind
├── services/
│   ├── auth/              # Microservicio de usuarios (NestJS)
│   └── citas/             # Microservicio de citas (NestJS)
├── docs/
│   └── routes.md          # Mapa del sitio, rutas y permisos
├── docker-compose.yml
└── README.md
```

> La estructura es la prevista; las carpetas se irán creando según avancen las fases.

## Documentación

- 🗺️ [Mapa del sitio, rutas y permisos](docs/routes.md): fuente de verdad de las páginas y de quién accede a cada una.

## Diseño

- **Herramientas:** FigJam para wireframes y Figma para el diseño final.
- **Paleta:** `#dccc8d` (acento dorado) · `#b5b5b5` · `#ffffff` · `#616060` · `#0c0c0c`
- **Frames de referencia:** 390 px (móvil), 768 px (tableta) y 1440 px (escritorio).

## Convenciones

- **Idioma:** código, archivos y componentes en inglés; textos visibles y rutas (URL) en español; comentarios y documentación en español.
- **Commits:** [Conventional Commits](https://www.conventionalcommits.org/es/v1.0.0/) (`feat:`, `fix:`, `docs:`).
- **Ramas:** `feature/nombre-corto`, con pull request hacia `main`.

## Cómo ejecutarlo

Todavía no hay código que ejecutar: el proyecto está en la fase de diseño. Las instrucciones de instalación y despliegue se añadirán con la v1.0.

## Autor

Desarrollado por **brunobuendiadev**, estudiante de 2º de Desarrollo de Aplicaciones Web.

---

<div align="center">

© 2026 Kanusi · Hecho con cariño en un pueblo

</div>
