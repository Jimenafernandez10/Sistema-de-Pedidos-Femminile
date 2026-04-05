# Femminile — Guía de deploy

## Archivos del proyecto

```
femminile-app/
├── index.html          ← app completa (HTML + CSS + JS)
├── vercel.json         ← configuración de deploy
├── supabase_schema.sql ← schema de base de datos
└── README.md           ← esta guía
```

---

## Paso 1 — Crear proyecto en Supabase

1. Entrá a https://supabase.com y creá una cuenta (es gratis)
2. Hacé click en **New Project**
3. Elegí un nombre: `femminile`
4. Elegí región más cercana (por ej. South America / São Paulo)
5. Creá una contraseña segura y guardala
6. Esperá ~2 minutos mientras se provisiona

---

## Paso 2 — Ejecutar el Schema SQL

1. En tu proyecto Supabase, andá a **SQL Editor** (ícono de base de datos)
2. Hacé click en **New Query**
3. Copiá y pegá todo el contenido de `supabase_schema.sql`
4. Hacé click en **Run** (o `Ctrl+Enter`)
5. Deberías ver: `Success. No rows returned`

Esto crea:
- Tabla `pedidos` con columnas calculadas automáticas
- Tabla `cobros`
- Row Level Security habilitado
- Datos de ejemplo (opcional, podés borrar esa sección del SQL)

---

## Paso 3 — Obtener las credenciales

1. En tu proyecto Supabase, andá a **Settings → API**
2. Copiá estos dos valores:
   - **Project URL** → algo como `https://abcdefgh.supabase.co`
   - **anon public key** → un JWT largo

---

## Paso 4 — Pegar las credenciales en index.html

Abrí `index.html` y buscá estas dos líneas (están juntas, cerca del final):

```javascript
const SUPABASE_URL  = 'https://TU_PROJECT_ID.supabase.co';  // <-- reemplazá
const SUPABASE_ANON = 'TU_ANON_PUBLIC_KEY';                 // <-- reemplazá
```

Reemplazalas con tus valores reales:

```javascript
const SUPABASE_URL  = 'https://abcdefgh.supabase.co';
const SUPABASE_ANON = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...';
```

Guardá el archivo.

---

## Paso 5 — Deploy en Vercel

### Opción A — Vercel CLI (recomendada)

```bash
# 1. Instalar Vercel CLI (una sola vez)
npm install -g vercel

# 2. Entrar a la carpeta del proyecto
cd femminile-app

# 3. Deploy
vercel

# Seguir los pasos:
# - ¿Linking to existing project? → No
# - Project name → femminile
# - In which directory? → ./
# - Want to override settings? → No

# 4. Para producción
vercel --prod
```

### Opción B — Vercel Web (más fácil)

1. Subí la carpeta `femminile-app` a un repo de GitHub
2. Entrá a https://vercel.com y logueate
3. Click en **Add New Project**
4. Importá el repo de GitHub
5. Sin configurar nada extra, click en **Deploy**
6. En ~30 segundos tenés tu URL pública

---

## Paso 6 — Verificar que funciona

Al abrir la app debería aparecer:
- Toast verde: **"Base de datos conectada"**
- Punto verde en el sidebar: **"Conectado · Supabase"**
- Los datos de ejemplo cargados en el dashboard

Si aparece toast rojo, revisá que las credenciales en `index.html` sean correctas.

---

## Operaciones disponibles

| Acción | Tabla | Método |
|--------|-------|--------|
| Ver pedidos | `pedidos` | SELECT |
| Crear pedido | `pedidos` | INSERT |
| Editar pedido | `pedidos` | UPDATE |
| Eliminar pedido | `pedidos` | DELETE |
| Ver cobros | `cobros` | SELECT |
| Crear cobro | `cobros` | INSERT |
| Eliminar cobro | `cobros` | DELETE |

---

## Columnas calculadas en Supabase

Las columnas `total`, `ganancia` y `saldo` se calculan automáticamente en la base de datos:

```sql
total    = cant * precio
ganancia = cant * (precio - costo)
saldo    = MAX(0, total - seña)
```

Esto significa que **nunca podés quedar con datos inconsistentes**, incluso si editás directamente desde el panel de Supabase.

---

## Seguridad (para el futuro)

Actualmente la app usa Row Level Security con política abierta (cualquiera puede leer/escribir si tiene la URL). Cuando quieras agregar login:

1. Activá **Auth → Email** en Supabase
2. Reemplazá la política RLS por:

```sql
-- Solo el usuario autenticado puede ver sus datos
create policy "solo autenticada" on public.pedidos
  for all using (auth.uid() is not null);
```

3. Agregá `await db.auth.signInWithPassword({ email, password })` en el JS

---

## Soporte

Si algo no funciona, revisá:
1. Las credenciales en `index.html` (URL y ANON KEY)
2. Que el SQL se haya ejecutado sin errores
3. Que las tablas `pedidos` y `cobros` existan en **Table Editor**
