# AGENT.md

Documento de referencia técnica y operativa del proyecto. Fuente de verdad para humanos y agentes de IA.

---

## Stack base

- OS dev: Ubuntu 22.04
- Node.js: v22.x
- Package manager: **pnpm**
- Framework: **Next.js (App Router)**
- ORM: **Prisma**
- DB: **MariaDB**
- UI: **shadcn/ui**
- Estilos: **Tailwind CSS + SCSS**

---

## Arquitectura Next.js

- Se usa **App Router exclusivamente**
- Server Components por defecto
- Client Components solo cuando:
  - Hay interacción directa (forms, modals)
  - Uso explícito de hooks (`useState`, `useEffect`)
- Data fetching y lógica sensible **solo en server** (auth, hashing, sesiones).

NO usar:

- Pages Router
- getServerSideProps / getStaticProps

---

## Autenticación y seguridad

### Alcance actual

- Register
- Login
- Logout
- (Reset password definido pero no implementado aún)

### Autenticación

- Tipo: **Auth propia (credentials)**
- No OAuth por ahora
- No JWT para sesiones

### Hash de passwords

- Librería: `@node-rs/argon2`
- Algoritmo: **argon2id**
- Nunca guardar passwords en texto plano
- Nunca exponer lógica de hash al cliente

### Sesión

- Tipo: **cookie httpOnly**
- Librería: `iron-session`
- Cookies:
  - httpOnly: true
  - secure: true (en producción)
  - sameSite: `lax` o `strict`

Logout = invalidación de sesión server-side.

---

## Validación

- Librería: **Zod**
- Validar:
  - Inputs de login
  - Inputs de register
  - Inputs de reset password
- No usar class-validator
- No confiar en validación del cliente

---

## Seguridad adicional

- Rate limit:
  - Login
  - Register
  - Reset password
- Headers de seguridad:
  - Content-Security-Policy
  - X-Frame-Options
  - Referrer-Policy
  - Strict-Transport-Security
- No exponer secretos al cliente.
- No usar librerías legacy (Passport, csurf).
- No usar Helmet (pensado para Express)

---

## Estilos

- Tailwind:
  - Layout
  - Spacing
  - Tokens
  - Estados
- SCSS:
  - Mixins
  - Animaciones complejas
  - Overrides puntuales
- shadcn/ui sin modificar su API base.

Reglas:

- No replicar utilidades Tailwind en SCSS
- SCSS global en `app/globals.scss`

---

## Código y calidad (Linting y formato)

- ESLint:
  - eslint-config-next
  - @typescript-eslint (strict)
- Prettier obligatorio
- El código debe pasar lint antes de commit

---

## Git y DX

- Husky habilitado
- Hooks:
  - pre-commit → lint-staged
  - commit-msg → conventional commits
- lint-staged:
  - Ejecutar eslint + prettier solo en archivos staged
- Conventional Commits:
  - feat:
  - fix:
  - chore:
  - refactor:
  - docs:
  - test:

Commits que no cumplen convención → rechazados.

---

## Variables de entorno

- Variables de entorno gestionadas con `dotenv-safe`
- `.env.example` obligatorio
- No hardcodear secretos
- No subir `.env` reales al repo

---

## Fuera de alcance (por ahora)

- OAuth (Google, GitHub, etc.)
- Multi-tenant
- Roles y permisos avanzados
- Microservicios
- JWT stateless

---

## Convenciones

- No lógica de negocio en componentes UI.
- No validación manual sin Zod.
- No acceso directo a cookies fuera del server.
- No introducir nuevas librerías sin justificar.

---

## Evolución prevista

- Reset password.
- OAuth (Google/GitHub) como capa adicional.
- Escalado horizontal manteniendo sesiones controladas.

---

## Regla final

Si una decisión **no está documentada aquí**,  
**NO se asume**.  
Se discute y se agrega a este archivo.
Cambios arquitectónicos requieren actualizar este archivo.
