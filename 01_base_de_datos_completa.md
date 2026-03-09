# Base de Datos Completa de FinancePro

## 1) Vision general

La base de datos fue redisenada para cubrir **todos los apartados reales de la aplicacion**:

- autenticacion y cuenta de usuario
- perfil personal
- configuracion/preferencias
- movimientos financieros
- inversiones y portafolio
- noticias financieras
- datos de mercado
- notificaciones
- auditoria de eventos

El motor usado es SQLite (`financepro.db`) por simplicidad, portabilidad y facilidad para desarrollo local.

---

## 2) Modelo de datos por modulo

### 2.1 Usuarios y autenticacion

#### Tabla: `users`

Proposito: almacenar la identidad principal de acceso.

Campos clave:

- `email` (PK): identificador unico del usuario.
- `password`: contrasena (actualmente en texto; recomendacion de mejora: hash bcrypt).
- `name`: nombre completo mostrado en UI.
- `is_new`: bandera para experiencia de usuario nuevo.
- `status`: estado de cuenta (`active` / `disabled`).
- `created_at`, `updated_at`, `last_login_at`: trazabilidad temporal.

Relacion principal:

- `users.email` se conecta con casi todos los modulos por llave foranea.

---

### 2.2 Perfil de usuario

#### Tabla: `user_profiles`

Proposito: separar datos de perfil de los datos de autenticacion.

Campos:

- `user_email` (PK + FK -> `users.email`)
- `first_name`, `last_name`
- `bio`
- `avatar_url`
- `membership_level`
- `joined_at`, `created_at`, `updated_at`
- `country`, `timezone`, `language`

Beneficio de diseno:

- evita sobrecargar la tabla `users`
- permite evolucionar perfil sin afectar login

---

### 2.3 Configuracion del sistema

#### Tabla: `user_settings`

Proposito: persistir la configuracion visual y de privacidad.

Campos:

- `user_email` (PK + FK)
- `currency` (USD, EUR, COP, MXN, etc.)
- `dynamic_dark_mode`
- `two_factor_enabled`
- `market_alerts_enabled`
- `weekly_summary_enabled`
- `crypto_news_enabled`
- `new_device_alerts_enabled`
- `created_at`, `updated_at`

Mapeo con UI:

- pagina `settings.html`
- selector de moneda
- toggles de experiencia, seguridad y notificaciones

---

### 2.4 Movimientos financieros

#### Tabla: `movements`

Proposito: registrar ingresos y gastos.

Campos:

- `id` (PK autoincremental)
- `user_email` (FK)
- `type` (`income` / `expense`)
- `concept`
- `amount`
- `date`
- `category`
- `source` (manual / legacy-json)
- `created_at`, `updated_at`

Indices:

- `idx_movements_user_date` para consultas rapidas por usuario y fecha.

Mapeo con UI y API:

- `movements.html`
- `GET /api/movements`
- `POST /api/add-movement`
- `GET /api/export-expenses`

---

### 2.5 Inversiones y portafolio

#### Tabla: `investments`

Proposito: almacenar activos del usuario.

Campos:

- `id` (PK)
- `user_email` (FK)
- `symbol`
- `name`
- `investment` (capital inicial)
- `current` (valor actual)
- `pnl` (rendimiento)
- `type` (`crypto`, `stock`, `other`)
- `created_at`, `updated_at`

Indice:

- `idx_investments_user`

Mapeo:

- `investments.html`
- `portfolio.html`
- `GET /api/investments`
- `POST /api/add-investment`

#### Tabla: `portfolio_snapshots`

Proposito: historico de valor neto para graficas reales de evolucion.

Campos:

- `id` (PK)
- `user_email` (FK)
- `snapshot_date`
- `net_worth`
- `liquid_balance`
- `invested_value`
- `daily_change_percent`
- `created_at`

Regla:

- `UNIQUE (user_email, snapshot_date)` evita duplicar snapshot diario.

---

### 2.6 Noticias financieras

#### Tabla: `news_articles`

Proposito: almacenar noticias para la seccion de inteligencia de mercado.

Campos:

- `id` (PK)
- `category`
- `source`
- `title`
- `summary`
- `image_url`
- `article_url` (UNIQUE)
- `published_at`
- `created_at`

Indice:

- `idx_news_category_date`

Mapeo:

- `news.html`

---

### 2.7 Datos de mercado

#### Tabla: `market_assets`

Proposito: soportar widgets de precios (BTC, ETH, SPX, AAPL).

Campos:

- `symbol` (PK)
- `display_name`
- `asset_class`
- `price`
- `change_percent`
- `updated_at`

Mapeo:

- `GET /api/market-data` (actualmente mock en backend; esta tabla deja lista la persistencia real)

---

### 2.8 Notificaciones

#### Tabla: `notifications`

Proposito: historial de alertas del usuario.

Campos:

- `id` (PK)
- `user_email` (FK)
- `type`
- `title`
- `message`
- `is_read`
- `created_at`

Indice:

- `idx_notifications_user_read`

Mapeo:

- dropdown de notificaciones en `layout.html`.

---

### 2.9 Auditoria

#### Tabla: `audit_logs`

Proposito: trazabilidad de eventos de seguridad y acciones.

Campos:

- `id` (PK)
- `user_email` (FK nullable)
- `event_type`
- `event_detail`
- `ip_address`
- `user_agent`
- `created_at`

Uso recomendado:

- login exitoso/fallido
- cambio de password
- desactivacion de cuenta
- alta de movimientos/inversiones

---

## 3) Relacion entre tablas (resumen)

- `users` es la entidad padre.
- `user_profiles`, `user_settings`, `movements`, `investments`, `portfolio_snapshots`, `notifications` apuntan a `users`.
- `news_articles` y `market_assets` son catalogos globales del sistema.
- `audit_logs` puede apuntar a usuario o registrar eventos sin usuario autenticado.

---

## 4) Estrategia de migracion aplicada

Se mantuvo compatibilidad con tu fuente original `db.json`:

- usuarios se migran con `INSERT OR IGNORE`
- movimientos e inversiones se migran solo si esas tablas estan vacias
- se evita duplicacion al ejecutar `database_setup.py` multiples veces
- por cada usuario se crea automaticamente su `user_profile` y `user_settings` por defecto

---

## 5) Datos semilla incluidos

Durante inicializacion se cargan referencias para:

- `market_assets`: BTC, ETH, SPX, AAPL
- `news_articles`: noticias base para poblar el modulo de noticias

Esto garantiza una experiencia visual completa desde el primer arranque.

---

## 6) Justificacion tecnica del diseno

1. **Separacion de responsabilidades**  
   Usuarios, perfil, configuracion y finanzas se modelan por separado para facilitar mantenimiento.

2. **Escalabilidad progresiva**  
   Aunque SQLite es local, el modelo ya esta listo para migrar a PostgreSQL/MySQL con cambios minimos.

3. **Integridad de datos**  
   Se usan llaves foraneas, `CHECK`, `UNIQUE` e indices estrategicos.

4. **Trazabilidad y seguridad operativa**  
   Auditoria y timestamps permiten entender el ciclo de vida de los datos.

5. **Alineacion directa con la UI actual**  
   Cada pantalla de la app tiene respaldo en tablas concretas.

---

## 7) Mejoras futuras recomendadas (nivel profesional)

- cifrar/hash de contrasenas con `passlib[bcrypt]`
- tokens de sesion en backend (evitar depender de `localStorage` como sesion principal)
- versionado de movimientos/inversiones (historial de ediciones)
- tabla de categorias personalizadas por usuario
- sincronizacion real de precios de mercado con proveedor externo
- pruebas automatizadas de migracion y consistencia de esquema
