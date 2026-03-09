# Explicacion Detallada de `database_setup.py`

Este documento explica de forma profunda el archivo `database_setup.py` para que puedas presentarlo de manera profesional.

---

## 1) Objetivo del script

`database_setup.py` es el orquestador de persistencia. Su responsabilidad es:

1. crear todas las tablas necesarias
2. aplicar reglas de integridad
3. migrar datos historicos (`db.json`) sin duplicar
4. sembrar datos de referencia (mercado y noticias)
5. dejar la base lista para ejecucion inmediata

---

## 2) Bloques funcionales del archivo

## 2.1 Configuracion global

- `DB_FILE = "financepro.db"`: define el archivo principal SQLite.
- `JSON_FILE = "db.json"`: define fuente legacy para migracion inicial.

Esto permite separar claramente:

- base operativa actual (SQLite)
- fuente historica de respaldo (JSON)

## 2.2 Funciones utilitarias

- `_utc_now()`: genera timestamp UTC para consistencia temporal.
- `_split_name(full_name)`: divide nombre completo en nombre/apellido para poblar `user_profiles`.

Valor tecnico:

- evita valores nulos
- estandariza data inicial

## 2.3 Conexion robusta

- `get_connection()`:
  - abre conexion SQLite
  - activa `PRAGMA foreign_keys = ON`

Esto es clave porque SQLite no siempre aplica FK por defecto en todas las conexiones.

## 2.4 Creacion del esquema

La funcion `create_tables()` crea toda la estructura del sistema:

- `users`
- `user_profiles`
- `user_settings`
- `movements`
- `investments`
- `portfolio_snapshots`
- `news_articles`
- `market_assets`
- `notifications`
- `audit_logs`

Tambien crea indices para consultas frecuentes:

- movimientos por usuario/fecha
- inversiones por usuario
- notificaciones por estado lectura
- noticias por categoria/fecha

## 2.5 Validacion de existencia de datos

- `_has_rows(cursor, table_name)` devuelve si una tabla ya tiene registros.

Se usa para evitar reimportar datos legacy y crear duplicados.

## 2.6 Provision de perfil y settings

- `_ensure_user_profile_and_settings(...)`:
  - crea `user_profiles` por defecto si falta
  - crea `user_settings` por defecto si falta

Ventaja:

- cualquier usuario existente siempre queda completo a nivel funcional.

## 2.7 Migracion desde JSON

`migrate_data()` realiza:

1. lectura de `db.json`
2. insercion de usuarios con `INSERT OR IGNORE`
3. inicializacion de perfil + settings para cada usuario
4. migracion de `movements` solo si tabla esta vacia
5. migracion de `investments` solo si tabla esta vacia

Con esto la migracion es **idempotente**:

- puedes correr el script multiples veces sin romper integridad.

## 2.8 Datos semilla

`seed_reference_data()`:

- inserta/actualiza activos de mercado (`BTC`, `ETH`, `SPX`, `AAPL`)
- agrega noticias base con `INSERT OR IGNORE`
- garantiza perfiles/settings para todos los usuarios existentes

Objetivo:

- la app no arranca con vistas vacias.

## 2.9 Orquestacion final

- `initialize_database()` llama en orden:
  1) `create_tables()`
  2) `migrate_data()`
  3) `seed_reference_data()`

- `if __name__ == "__main__":` ejecuta ese flujo al correr el script.

---

## 3) Integridad y buenas practicas aplicadas

### 3.1 Llaves foraneas

Se definen referencias entre tablas para mantener consistencia relacional.

### 3.2 Restricciones CHECK

Campos como estados y banderas booleanas se limitan a valores validos.

### 3.3 Unicidad

Se usan claves unicas para evitar colisiones:

- `users.email`
- `market_assets.symbol`
- `news_articles.article_url`
- `portfolio_snapshots(user_email, snapshot_date)`

### 3.4 Idempotencia

El script puede ejecutarse muchas veces sin degradar datos:

- `INSERT OR IGNORE`
- verificacion de tablas vacias para migracion legacy

---

## 4) Relacion con las paginas de la app

- `login/register` -> `users`
- `profile` -> `user_profiles`
- `settings` -> `user_settings`
- `movements` -> `movements`
- `investments/portfolio` -> `investments`, `portfolio_snapshots`
- `news` -> `news_articles`
- dashboard market widgets -> `market_assets`
- dropdown alertas -> `notifications`
- seguridad/trazabilidad -> `audit_logs`

---

## 5) Lectura profesional de "que hace cada bloque"

Puedes presentar el script en 5 ideas:

1. **define** el contrato de datos de la plataforma
2. **protege** integridad y coherencia de registros
3. **migra** el historial anterior a un formato relacional
4. **siembra** contenido inicial util para demo y uso real
5. **garantiza** repetibilidad segura en cada arranque

---

## 6) Recomendaciones para una version enterprise

- mover credenciales de DB y modo a variables de entorno
- usar migraciones versionadas (Alembic) en lugar de solo `CREATE IF NOT EXISTS`
- agregar pruebas automaticas de esquema/migracion
- auditar eventos en tiempo real desde endpoints de API
- almacenar contrasenas hasheadas y rotar politica de seguridad

---

## 7) Conclusion

`database_setup.py` ya no es solo "crear 3 tablas". Ahora funciona como un modulo de infraestructura de datos completo, alineado con todos los apartados funcionales de la app y preparado para evolucionar con un enfoque profesional de mantenimiento y escalabilidad.
