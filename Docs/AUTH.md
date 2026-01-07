# AUTH.md

Documento de referencia para el sistema de autenticación. Describe flujos, responsabilidades y decisiones.

---

## Alcance

Implementa:

- Registro de usuario
- Login
- Logout
- Reset password (fase futura)

No implementa:

- OAuth
- Multi-tenant
- Roles/permisos

---

## Principios

- Auth **server-first**.
- Ningún secreto o lógica sensible en el cliente.
- Validación estricta con Zod.
- Sesiones controladas vía cookies httpOnly.

---

## Modelo de usuario (conceptual)

Campos mínimos:

- id (UUID)
- email (único)
- passwordHash
- createdAt
- updatedAt

El password **nunca** se almacena en texto plano.

---

## Registro (Register)

### Flujo

1. Usuario envía email + password.
2. Validación Zod (formato, longitud, fuerza mínima).
3. Normalización de email (lowercase).
4. Hash del password con `argon2id`.
5. Persistencia en DB vía Prisma.
6. Creación de sesión.
7. Set de cookie httpOnly.

### Reglas

- Email único.
- Password mínimo (longitud + entropía).
- Mensajes de error genéricos.

---

## Login

### Flujo

1. Usuario envía email + password.
2. Validación Zod.
3. Búsqueda de usuario por email.
4. Verificación del hash (`argon2id`).
5. Creación/actualización de sesión.
6. Set de cookie httpOnly.

### Reglas

- No revelar si el email existe.
- Rate limit activo.
- Logs sin datos sensibles.

---

## Logout

### Flujo

1. Invalidar sesión server-side.
2. Borrar cookie.

---

## Sesión

- Cookie httpOnly, `secure`, `sameSite=lax|strict`.
- TTL definido (ej: 7–30 días).
- Manejo mediante `iron-session`.

No se usa JWT para sesiones.

---

## Reset Password (futuro)

### Flujo previsto

1. Usuario solicita reset (email).
2. Generación de token de un solo uso.
3. Token con expiración corta.
4. Envío por email.
5. Validación token.
6. Nuevo password → nuevo hash.
7. Invalidar sesiones activas.

### Reglas

- Token no reutilizable.
- No indicar si el email existe.
- Rate limit estricto.

---

## Seguridad adicional

- Headers de seguridad activos.
- Protección CSRF implícita (cookies + POST).
- Rate limit en login y reset.

---

## Errores y UX

- Mensajes genéricos.
- No filtrar información sensible.
- Errores técnicos solo en logs.

---

## Regla final

Si el flujo de auth cambia, este documento **debe** actualizarse.
Código y documentación deben permanecer alineados.
