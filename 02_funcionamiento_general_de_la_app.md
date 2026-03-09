# Funcionamiento General de FinancePro

## 1) Objetivo de la plataforma

FinancePro es un dashboard financiero web para:

- registrar ingresos y gastos
- gestionar inversiones
- visualizar resumen patrimonial
- consultar informacion de mercado y noticias
- personalizar perfil y configuracion

La aplicacion usa:

- **Backend**: FastAPI (`main.py`)
- **Frontend**: Jinja2 + HTML + Tailwind + JavaScript
- **Persistencia**: SQLite (`financepro.db`)

---

## 2) Flujo tecnico de arranque

Archivo clave: `iniciar.bat`.

Secuencia:

1. valida instalacion de Python
2. crea/activa entorno virtual
3. instala dependencias (`requirements.txt`)
4. ejecuta `database_setup.py` para asegurar esquema completo
5. exporta dump SQL legible (`financepro_dump.sql`)
6. inicia servidor FastAPI con `python main.py`

Resultado:

- servidor en `http://localhost:8000`
- datos persistentes inicializados y consistentes

---

## 3) Arquitectura funcional por capas

### 3.1 Capa de presentacion

Plantillas en `app/templates`:

- `layout.html`: estructura global (sidebar, topbar, modal global, FAB, notificaciones).
- `dashboard.html`: vista general de mercado y KPIs.
- `movements.html`: historial y resumen de transacciones.
- `investments.html`: gestion de inversiones y rendimiento.
- `portfolio.html`: distribucion y valor total del portafolio.
- `news.html`: noticias financieras.
- `profile.html`: datos personales.
- `settings.html`: preferencias de sistema.
- `login.html` y `register.html`: autenticacion.

### 3.2 Capa de comportamiento cliente

Script principal: `app/static/js/app.js`.

Responsabilidades:

- verificar sesion local (`localStorage`)
- cargar y refrescar datos de APIs
- actualizar UI dinamicamente
- gestionar charts con ApexCharts
- controlar dropdowns, filtros, modal y acciones rapidas

### 3.3 Capa de API

`main.py` expone endpoints:

- Auth:
  - `POST /api/register`
  - `POST /api/login`
- Finanzas:
  - `GET /api/movements`
  - `POST /api/add-movement`
  - `GET /api/investments`
  - `POST /api/add-investment`
  - `GET /api/export-expenses`
- Mercado:
  - `GET /api/market-data`

Ademas expone rutas de render para cada pagina HTML.

### 3.4 Capa de datos

`database_setup.py` crea y mantiene el esquema completo, migra datos legacy y carga seeds.

---

## 4) Flujo de usuario de extremo a extremo

## 4.1 Registro

1. Usuario completa `register.html`.
2. Frontend envia datos a `POST /api/register`.
3. Backend valida email existente.
4. Si no existe, crea registro en `users`.
5. Redirecciona a login con `registered=true`.

## 4.2 Login

1. Usuario ingresa credenciales en `login.html`.
2. Frontend llama `POST /api/login`.
3. Si es correcto, backend devuelve objeto usuario.
4. Frontend guarda `currentUser` en `localStorage`.
5. Redireccion a dashboard (`/?user=new` si usuario nuevo).

## 4.3 Operacion diaria

- Dashboard consulta movimientos + inversiones y calcula KPIs.
- Modal global permite alta de:
  - gasto
  - ingreso
  - inversion
- Cambios persisten por API y luego la UI recarga.

## 4.4 Consulta de historial

- `movements.html` lista transacciones con resumen:
  - total ingresos
  - total gastos
  - balance neto

## 4.5 Portafolio e inversiones

- `investments.html` y `portfolio.html` consultan inversiones.
- Se calcula rendimiento total y distribucion por tipo de activo.

## 4.6 Perfil y ajustes

- `profile.html` consume datos de `localStorage` en version actual.
- `settings.html` muestra estructura de preferencias lista para persistir.

---

## 5) Como se logro la implementacion

## 5.1 Tecnologias elegidas y por que

- **FastAPI**: rapidez para construir APIs tipadas y ligeras.
- **SQLite**: ideal para despliegue local sin servidores externos.
- **Jinja2**: render server-side simple para rutas principales.
- **Tailwind + CSS custom**: interfaz moderna sin complejidad excesiva.
- **ApexCharts**: visualizacion financiera clara.

## 5.2 Enfoque de desarrollo

- primero se cubrio flujo principal (registro/login + movimientos/inversiones)
- luego se agrego experiencia visual y dashboard
- finalmente se profesionalizo modelo de datos y documentacion tecnica

## 5.3 Decisiones de producto/ingenieria

- uso de `localStorage` para sesion simulada (rapidez de prototipo)
- datos de mercado mock para no depender de API de terceros en desarrollo
- exportacion SQL para inspeccion humana del estado de la base

---

## 6) Limitaciones actuales (transparentes y profesionales)

- contrasenas sin hash en implementacion actual (debe endurecerse)
- sesion de frontend en `localStorage` sin token backend robusto
- datos de mercado y noticias con semilla/mock, no feed en vivo
- varios controles de perfil/ajustes aun no conectados a endpoints dedicados

---

## 7) Plan recomendado de evolucion

Fase 1 (seguridad):

- hash de contrasena con bcrypt
- sesion JWT + refresh token
- auditoria de login/fallo de acceso

Fase 2 (producto):

- CRUD completo para perfil y settings
- API real de mercado/noticias
- notificaciones persistentes por usuario

Fase 3 (analitica):

- snapshots diarios automaticos
- forecasting de gastos
- reportes PDF/CSV avanzados

---

## 8) Conclusion profesional

La app ya tiene una base funcional solida y ahora cuenta con un modelo de datos integral que cubre cada apartado visible del sistema. El diseno actual permite evolucionar a un producto mas robusto sin rehacer la arquitectura principal, porque separa correctamente dominios (auth, perfil, settings, finanzas, mercado y auditoria) y define un camino claro de escalabilidad.
