# 🌬️ AirGuard v3.5.1 – Portal Web de Calidad del Aire y Salud Ambiental

> **Universidad Nova Digital · Unidad de Salud Ambiental**  
> San Salvador, El Salvador

---

## 📋 Descripción

AirGuard es una **Aplicación Web Progresiva (PWA)** para el monitoreo ciudadano de la calidad del aire en zonas urbanas de San Salvador, El Salvador. Transforma datos técnicos complejos provenientes de sensores ambientales y APIs públicas en información clara y comprensible, permitiendo a los ciudadanos tomar **decisiones preventivas de salud**.

### Capacidades principales

- **6 contaminantes monitoreados:** PM2.5, PM10, CO, NO₂, O₃ y SO₂
- **Mapa interactivo** con marcadores codificados por color según el ICA
- **Comparativa entre estaciones** con gráficos agrupados y sparklines individuales
- **Tendencias históricas de 24 horas** con gráficos Recharts
- **Sistema de auto-refresco** configurable (Manual, 1, 5, 10, 30 min)
- **Notificaciones push** con umbrales personalizables
- **Exportación de datos** en CSV (nativo) y PDF (jsPDF con identidad institucional)
- **Modo Demo** automático cuando la API no está disponible
- **Diseño responsivo** mobile-first (apilado vertical en < 768px)
- **Persistencia histórica** mediante base de datos MongoDB con captura automática cada 30 minutos
- **Contenerización Docker** multi-stage (Node build → Nginx producción)

---

## 🔄 Flujo de Datos: API → Procesamiento → Persistencia → Visualización

```
┌─────────────────┐     ┌──────────────────────┐     ┌─────────────────────┐
│   OpenAQ API v3 │────▶│  openaqService.js     │────▶│  aqiCalculator.js   │
│  (Datos reales) │     │  - fetchLocations()   │     │  - normalizeRecord()│
│  País: SV       │     │  - fetchMeasurements() │     │  - getAQICategory() │
│  Ciudad: SS     │     │  - fetchLatestReadings│     │  - 6 parámetros ICA │
└─────────────────┘     └──────────┬───────────┘     └──────────┬──────────┘
                                   │                             │
                         ┌─────────▼──────────┐                 │
                         │  useAirQuality.js   │◀────────────────┘
                         │  (React Hook)       │
                         │  - Estado global    │
                         │  - Auto-refresh     │
                         │  - Promise.allSettld│
                         └─────────┬───────────┘
                                   │
              ┌────────────────────┼────────────────────┐
              ▼                    ▼                    ▼
     ┌────────────────┐  ┌─────────────────┐  ┌──────────────────┐
     │   AirMap.jsx   │  │ StationDetail   │  │ StationsList.jsx │
     │  (Leaflet)     │  │ .jsx + Trend    │  │  (Lista lateral) │
     │  Marcadores    │  │ Chart (Recharts)│  │  ordenada por    │
     │  coloreados    │  │  Últimas 24h    │  │  riesgo ICA      │
     └────────────────┘  └─────────────────┘  └──────────────────┘

    ┌──────────────────────────────────────────────────────────────────┐
    │  CAPA DE PERSISTENCIA HISTÓRICA                                   │
    │                                                                    │
    │  ┌─────────────┐    cada 30 min    ┌────────────────────────┐    │
    │  │ Automatización│───────────────▶│  MongoDB (AirReading)  │    │
    │  │ programada    │  fetch + ICA    │  - station_id          │    │
    │  │ (cron)        │  + normalizar   │  - parameter (6 tipos) │    │
    │  └─────────────┘                  │  - value, unit         │    │
    │                                    │  - ica_label, color    │    │
    │                                    │  - health_message      │    │
    │                                    │  - recorded_at (ISO)   │    │
    │                                    └────────────────────────┘    │
    └──────────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Stack Tecnológico

| Capa | Tecnología | Versión |
|------|-----------|---------|
| Frontend | React 18 | 18.2.0 |
| Mapas | Leaflet + React-Leaflet | 1.9.4 / 4.2.1 |
| Gráficas | Recharts (AreaChart, BarChart) | 2.10.3 |
| HTTP Client | Axios | 1.6.7 |
| Fechas | date-fns | 3.3.1 |
| Íconos | Lucide React | 0.312.0 |
| PDF | jsPDF + jsPDF-autotable | 2.5.1 / 3.8.2 |
| PWA | Service Worker + Web Manifest | — |
| Base de datos | MongoDB (entidad AirReading) | — |
| Servidor | Nginx (producción) | 1.25 |
| Contenedor | Docker multi-stage (Node 20 + Nginx) | — |
| Deploy | Vercel / Netlify / AWS Amplify / Docker | — |

---

## 📊 Parámetros Monitoreados

| Parámetro | Descripción | Unidad | Estándar ICA |
|-----------|-------------|--------|---------------|
| PM2.5 | Material particulado fino (< 2.5 µm) | µg/m³ | EPA |
| PM10 | Material particulado grueso (< 10 µm) | µg/m³ | EPA |
| CO | Monóxido de carbono | ppm | EPA |
| NO₂ | Dióxido de nitrógeno | µg/m³ | OMS / EPA |
| O₃ | Ozono troposférico | µg/m³ | EPA |
| SO₂ | Dióxido de azufre | µg/m³ | EPA / OMS |

---

## 🏥 Categorías de Salud (ICA)

| Categoría | Rango ICA | Color | Recomendación de salud |
|-----------|----------|-------|------------------------|
| 🟢 Buena | 0–50 | `#16a34a` | Sin impactos previstos en la salud |
| 🟡 Moderada | 51–100 | `#ca8a04` | Grupos sensibles: precaución leve |
| 🟠 Grupos Sensibles | 101–150 | `#ea580c` | Reducir actividades prolongadas al aire libre |
| 🔴 Dañina | 151–200 | `#dc2626` | Todos pueden sufrir efectos en la salud |
| 🟣 Muy Dañina | 201–300 | `#9333ea` | Alerta de salud: efectos graves |
| ⚫ Peligrosa | 301+ | `#991b1b` | Emergencia sanitaria |

Cada categoría incluye un **mensaje de salud** específico que se muestra en el panel de detalles y en las cards de comparativa.

---

## 🏗️ Estructura del Proyecto

```
airguard/
├── public/
│   ├── index.html            # HTML base con meta tags PWA
│   ├── manifest.json          # Configuración PWA (nombre, íconos, colores)
│   └── sw.js                  # Service Worker (cache offline + notificaciones)
├── src/
│   ├── components/
│   │   ├── Header.jsx          # Barra superior: refresco, notificaciones, countdown
│   │   ├── AirMap.jsx          # Mapa Leaflet con marcadores ICA coloreados
│   │   ├── TrendChart.jsx      # Gráfico Recharts AreaChart (tendencia 24h)
│   │   ├── StationDetail.jsx   # Panel de detalles + KPIs + mensaje de salud
│   │   ├── StationsList.jsx    # Lista lateral de estaciones ordenada por riesgo
│   │   ├── StationComparison.jsx # Vista comparativa: cards + BarChart agrupado
│   │   ├── FilterBar.jsx       # Filtros: parámetro (6 botones) + rango de fechas
│   │   └── NotificationPanel.jsx # Configuración de alertas push y umbrales
│   ├── hooks/
│   │   ├── useAirQuality.js    # Hook central: estado global, API, auto-refresh
│   │   └── usePushNotifications.js # Hook: permisos, umbrales, SW, notificaciones
│   ├── pages/
│   │   └── Dashboard.jsx       # Página principal: layout, tabs, responsive
│   ├── services/
│   │   └── openaqService.js    # Integración OpenAQ v3 + modo demo
│   ├── utils/
│   │   ├── aqiCalculator.js   # Lógica ICA: 6 parámetros, normalización, categorías
│   │   └── exportUtils.js     # Exportación CSV (nativo) y PDF (jsPDF)
│   ├── App.js                 # Punto de entrada React
│   └── index.js               # Bootstrap + Service Worker registration
├── .env.example               # Plantilla de variables de entorno
├── .gitignore                 # Excluye .env, node_modules, build
├── Dockerfile                 # Multi-stage: Node 20 build → Nginx 1.25
├── docker-compose.yml         # Orquestación: prod (Nginx) + dev (hot-reload)
├── netlify.toml               # Configuración de deploy en Netlify
├── vercel.json                # Configuración de deploy en Vercel
├── package.json               # Dependencias y scripts (v3.5.1)
├── README.md                  # Este archivo
├── DEPLOY.md                  # Guía detallada de despliegue
└── docs/
    ├── CHANGELOG.md           # Registro completo de versiones
    └── STATION_COMPARISON.md  # Documentación técnica del módulo comparativo
```

---

## ⚙️ Configuración

### 1. Obtener API Key de OpenAQ
1. Registrarse en [explore.openaq.org/register](https://explore.openaq.org/register)
2. Obtener tu API Key gratuita (permite ~1,000 requests/día)

### 2. Variables de entorno
```bash
# Copiar el archivo de ejemplo
cp .env.example .env

# Editar .env con tu API Key
REACT_APP_OPENAQ_API_KEY=tu_api_key_aqui
```

| Variable | Requerida | Valor por defecto | Descripción |
|----------|-----------|-------------------|-------------|
| `REACT_APP_OPENAQ_API_KEY` | ✅ Sí | — | API Key de OpenAQ |
| `REACT_APP_OPENAQ_BASE_URL` | No | `https://api.openaq.org/v3` | URL base de la API |
| `REACT_APP_TARGET_COUNTRY` | No | `SV` | Código ISO del país |
| `REACT_APP_TARGET_CITY` | No | `San Salvador` | Ciudad objetivo |

> ⚠️ **Seguridad**: El archivo `.env` está incluido en `.gitignore`. Nunca lo subas a control de versiones. Sin la API Key, la app funciona en **Modo Demo** con datos simulados realistas.

---

## 🚀 Instalación y Ejecución

### Desarrollo local
```bash
npm install
npm start
# Abrir http://localhost:3000
```

### Build de producción
```bash
npm run build
# Archivos estáticos en /build/ listos para deploy
```

### Docker (producción)
```bash
# Build y run con Docker
docker build \
  --build-arg REACT_APP_OPENAQ_API_KEY=tu_api_key \
  -t airguard:latest .

docker run -p 3000:80 airguard:latest
```

### Docker Compose
```bash
# Copiar y configurar .env primero
cp .env.example .env
# Editar .env con tu API Key

# Iniciar en producción
docker compose up -d airguard

# Iniciar en modo desarrollo (hot-reload en puerto 3001)
docker compose --profile dev up airguard-dev
```

---

## 🗄️ Capa de Persistencia Histórica

AirGuard cuenta con una capa de persistencia que almacena lecturas históricas de calidad del aire en una base de datos **MongoDB**, mediante la entidad `AirReading`.

### Esquema de la entidad AirReading

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `station_id` | number | ID numérico de la estación en OpenAQ |
| `station_name` | string | Nombre legible de la estación |
| `city` | string | Ciudad (San Salvador) |
| `latitude` | number | Latitud geográfica |
| `longitude` | number | Longitud geográfica |
| `parameter` | string | Contaminante: pm25, pm10, co, no2, o3, so2 |
| `value` | number | Valor numérico de la medición |
| `unit` | string | Unidad de medida (µg/m³ o ppm) |
| `ica_label` | string | Categoría ICA: Buena, Moderada, Dañina... |
| `ica_color` | string | Color hex de la categoría ICA |
| `health_message` | string | Mensaje preventivo de salud |
| `source` | string | Origen: "openaq" o "demo" |
| `recorded_at` | string | ISO 8601 — momento de la captura |

### Automatización de captura

Existe una automatización programada que ejecuta cada **30 minutos**:

1. Consulta la API de OpenAQ para las estaciones de San Salvador
2. Obtiene las lecturas más recientes de los 6 parámetros
3. Calcula el ICA correspondiente (categoría, color, mensaje de salud)
4. Normaliza y persiste cada lectura en la entidad `AirReading`

Esto permite acumular un historial de mediciones para análisis retrospectivo y validación académica.

---

## 🔄 Resiliencia de API — Reintentos con Backoff Exponencial

AirGuard implementa una política de reintentos con backoff exponencial para manejar fallos temporales de la API de OpenAQ (caídas, rate limiting, timeouts). Esto evita que microcortes del servicio manden al usuario innecesariamente al modo demo.

### Política de reintentos

| Intento | Espera | Condición |
|---------|--------|-----------|
| 1 (inmediato) | — | Petición inicial |
| 2 | 1 segundo | Si intento 1 falló con error recuperable |
| 3 | 2 segundos | Si intento 2 falló con error recuperable |
| 4 | 4 segundos | Si intento 3 falló con error recuperable |
| — | — | Si intento 4 falla → fallback a modo demo |

### Errores recuperables vs. no recuperables

| Tipo de error | ¿Reintenta? | Ejemplos |
|---------------|------------|----------|
| Timeout / sin respuesta | ✅ Sí | Red caída, DNS, conexión rechazada |
| HTTP 429 (Too Many Requests) | ✅ Sí | Rate limiting temporal de OpenAQ |
| HTTP 5xx (Server Error) | ✅ Sí | 502 Bad Gateway, 503 Service Unavailable, 504 Timeout |
| HTTP 4xx (excepto 429) | ❌ No | 400 Bad Request, 401 Unauthorized, 404 Not Found |

> Los errores 4xx (excepto 429) no se reintentan porque son errores del cliente — reintentar la misma petición no va a cambiar el resultado. Los errores 429 y 5xx sí son temporales y pueden resolverse esperando.

### Implementación

El mecanismo está en `src/services/openaqService.js` como función `retryWithBackoff()`. Las tres funciones de la API (`fetchLocations`, `fetchMeasurements`, `fetchLatestReadings`) envuelven sus llamadas axios con esta función.

## 📡 Estaciones Monitoreadas

| Estación | Latitud | Longitud | Perfil |
|----------|---------|----------|--------|
| San Salvador Centro | 13.6929 | -89.2182 | Urbano denso |
| San Salvador Este | 13.7000 | -89.1800 | Industrial / alta contaminación |
| San Salvador Norte | 13.7200 | -89.2100 | Residencial / más limpio |
| San Salvador Oeste | — | — | Urbano mixto |

---

## 🔒 Seguridad

- **API Keys**: Gestionadas exclusivamente via variables de entorno (`REACT_APP_*`)
- **`.env` en `.gitignore`**: Nunca se expone en el repositorio
- **Headers de seguridad Nginx**: X-Frame-Options, X-Content-Type-Options, X-XSS-Protection, Referrer-Policy
- **Docker multi-stage**: El stage de producción (Nginx) no incluye código fuente ni dependencias de desarrollo
- **Sanitización de inputs**: Validación de parámetros y location IDs antes de enviar a la API
- **Logging seguro**: `console.error` solo en desarrollo (`NODE_ENV !== 'production'`)
- **HTTPS**: Recomendado en producción detrás de un reverse proxy (CloudFlare, AWS ALB, etc.)

---

## 🔔 Sistema de Notificaciones Push

AirGuard incluye un sistema de notificaciones locales que alerta a los usuarios cuando la calidad del aire supera umbrales configurables.

### Umbrales por defecto

| Parámetro | Nivel de precaución | Nivel crítico |
|-----------|--------------------|---------------| 
| PM2.5 | 35.4 µg/m³ | 55.4 µg/m³ |
| PM10 | 154 µg/m³ | 254 µg/m³ |

### Características

- Solicita permiso explícito al usuario (Notification API)
- Registra un Service Worker para notificaciones en segundo plano
- Cooldown de 30 minutos entre alertas del mismo tipo
- Umbrales personalizables desde el panel de configuración
- Persistencia de preferencias en `localStorage`

---

## 📤 Exportación de Datos

### CSV (nativo del navegador)
- Generación directa con `Blob` + `URL.createObjectURL`
- Cabecera institucional con timestamp en zona `America/El_Salvador`
- BOM UTF-8 para compatibilidad con Excel Windows
- Escape de celdas según RFC 4180

### PDF (jsPDF)
- Informe institucional con identidad visual de AirGuard
- Encabezado con colores de alto contraste (título blanco, subtítulo slate-300)
- Tabla de lecturas con categorías ICA codificadas por color
- Pie de página con paginación automática
- Fallback a CSV si jsPDF no está disponible

---

## 📄 Licencia

Desarrollado para **Universidad Nova Digital** · Unidad de Salud Ambiental.  
Datos de calidad del aire provistos por [OpenAQ](https://openaq.org) bajo licencia abierta.
