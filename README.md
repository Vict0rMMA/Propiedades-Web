# API REST — Propiedades

API REST en **Node.js** y **Express 5** para gestionar un catálogo inmobiliario. Incluye listados con filtros y paginación, operaciones CRUD protegidas con **JWT** y subida de imágenes (con optimización opcional a WebP si está disponible **Sharp**).

**Stack:** Express 5, Prisma, PostgreSQL (`pg` + `@prisma/adapter-pg`), validación con Zod, seguridad con Helmet y logging con Morgan.

## Arquitectura rápida (lo importante)

La app está separada en dos niveles:

- **`src/server.js`**: entrypoint del proceso. Carga `.env`, conecta a la base de datos y levanta el servidor HTTP.
- **`src/app.js`**: configuración de Express. Aquí viven middlewares globales, rutas y manejo centralizado de errores.

### `src/server.js`

- Carga variables con `dotenv`.
- Conecta Prisma con `prisma.$connect()`; si falla, se corta el proceso (evita “API levantada sin DB”).
- Arranca Express en `PORT` (default `3000`).

### `src/app.js`

Orden de middlewares (de arriba hacia abajo):

1. **Helmet** (CSP opcional si `DISABLE_CSP=1`).
2. **CORS**.
3. **Morgan**.
4. **JSON parser** (`express.json()`).

Rutas montadas bajo **`/api/v1`**:

- `GET /api/v1/health`
- `POST /api/v1/auth/login`
- `GET|POST|PUT|DELETE /api/v1/properties`
- `POST /api/v1/upload/property-image` (JWT, multipart `image`)

> Nota: en este repo se eliminó la carpeta `public/`. Las imágenes se publican bajo `/uploads`, pero la carpeta donde se guardan (y la estrategia de persistencia) depende del entorno; en producción conviene usar Storage (S3/Supabase Storage) o un volumen.

## Requisitos

- Node.js (según `package-lock.json`).
- PostgreSQL accesible (local o cloud).

## Instalación local

```bash
npm install
cp .env.example .env
```

Si usas Supabase: normalmente `DATABASE_URL` apunta al pooler (6543) para runtime y `DIRECT_URL` a conexión directa (5432) para migraciones.

```bash
npm run prisma:generate
npm run prisma:migrate
npm run dev
```

Salud: `GET /api/v1/health`.

## Variables de entorno

| Variable | Uso |
|----------|-----|
| `DATABASE_URL` | Conexión runtime (en Supabase suele ser el pooler 6543). |
| `DIRECT_URL` | Conexión directa 5432 para migraciones. |
| `DATABASE_SSL` | `1` activa TLS (cloud); `0` local. |
| `JWT_SECRET` | Secreto JWT (obligatorio). |
| `ADMIN_EMAIL` / `ADMIN_PASSWORD` | Credenciales del login. |
| `PORT` | Puerto HTTP (default 3000). |
| `DISABLE_CSP` | `1` desactiva CSP de Helmet. |

## Scripts

| Comando | Descripción |
|---------|-------------|
| `npm run dev` | Desarrollo con nodemon. |
| `npm start` | Producción con node. |
| `npm run prisma:generate` | Genera cliente Prisma. |
| `npm run prisma:migrate` | Migraciones en desarrollo. |
| `npm run prisma:deploy` | Migraciones en despliegue. |
| `npm run prisma:studio` | Prisma Studio. |
| `npm run check:env` | Valida variables (archivo puede estar vacío por decisión del equipo). |

## Git (qué se sube y qué no)

- **No se suben**: `.env`, `node_modules/`, `src/generated/prisma/`, `public/`.
- **Sí se sube**: `.env.example`, migraciones, código fuente y configuración.

## Licencia

ISC — ver `package.json`.