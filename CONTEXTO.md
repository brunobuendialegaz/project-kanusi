# Barbería YAYAH — Contexto del proyecto

## Objetivo

Web real para una barbería de pueblo (estilo underground) y proyecto de aprendizaje
para mí (2º DAW). Debe estar online 365 días al año.

## Stack

- Frontend: Angular + Tailwind
- Backend: Spring Boot (microservicios, empezando por uno solo)
- BD: PostgreSQL
- Despliegue: VPS con Docker Compose + Caddy (HTTPS), CI/CD con GitHub Actions
- Repo: monorepo (frontend/, services/, docs/, docker-compose.yml)

## Diseño

- Herramientas: FigJam (wireframes) → Figma Design (diseño final)
- Paleta: #dccc8d (acento dorado), #b5b5b5, #ffffff, #616060, #0c0c0c
- Mobile-first: móvil → tableta → escritorio
- Páginas previstas: Landing, Reservar cita, Login/Registro, Panel del barbero

## Funcionalidades v1 (pendiente de cerrar)

- [ ] Ver servicios y precios
- [ ] Reservar cita (servicio → barbero → fecha/hora → confirmación)
- [ ] Login de clientes
- [ ] Panel del barbero (ver agenda)

## Decisiones tomadas

- (ir añadiendo aquí, una línea por decisión)

## Estado actual

- Fase: 0 (diseño y modelo de datos)
- Hecho: paleta y wireframe de la landing en 3 tamaños
- Siguiente: flujo de reserva en móvil

## Convenciones

- Commits: Conventional Commits (feat:, fix:, docs:)
- Ramas: feature/nombre-corto, PR hacia main
- Código en inglés, comentarios y docs en español
