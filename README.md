# Friolink — Plataforma IoT Industrial para Cadena de Frío

**Friolink** (anteriormente Cold Sense) es una plataforma SaaS full-stack para monitoreo térmico en tiempo real de cadenas de refrigeración, diseñada para logística farmacéutica y alimentaria.

<!-- Screenshot principal del dashboard -->
![Dashboard Principal](docs/images/dashboard.png)

## Arquitectura

```
┌─────────────┐     MQTT (1883)     ┌──────────────────┐     HTTP/WS (4000)     ┌─────────────┐
│  ESP32-C6   │◄───────────────────►│   Node.js Server │◄──────────────────────►│  PWA Client │
│  Firmware   │  Mauri Protocol v3.5│  Express + Aedes  │   REST + Socket.io    │  Vanilla JS │
└─────────────┘                     │  SQLite3          │                       └─────────────┘
                                    └────────┬─────────┘
                                             │ Axios
                                    ┌────────▼─────────┐
                                    │   WAHA (WhatsApp) │
                                    └──────────────────┘
```

## Stack Tecnológico

| Capa | Tecnología |
|------|-----------|
| Backend | Node.js, Express 5, JWT, Bcrypt |
| Tiempo real | Socket.io (WebSocket), Aedes (Broker MQTT) |
| Base de datos | SQLite3 |
| Frontend | Vanilla JS, Chart.js, Leaflet, Lucide Icons |
| Firmware | C++/Arduino, ESP32-C6 (Pulsar UNIT Electronics) |
| Notificaciones | WAHA (WhatsApp Business API) |

## Funcionalidades

### Hardware (Protocolo Mauri v3.5)
- Auto-descubrimiento y adopción de dispositivos vía handshake MQTT
- Estrategia dual-topic: ruta SaaS + ruta unificada
- Telemetría: temperatura interior/exterior, humedad, estado de puerta, RSSI, uptime
- ACK de comandos con trazabilidad completa (reboot, OTA, buzzer, puerta)
- Actualización OTA segura vía túneles LAN-aware

<!-- Screenshot del dispositivo ESP32-C6 o foto del hardware -->
![Hardware ESP32-C6](docs/images/hardware.png)

### Portal Cliente (PWA)
- Dashboard en tiempo real con tarjetas de estado por sensor
- Gráficos históricos de temperatura (Chart.js)
- Bitácora de alertas con resolución y seguimiento
- Gestión de dispositivos y organización por sucursales
- Integración WhatsApp con comandos (`/resumen`, `/temp`, `/uptime`, `/historial`)
- Modo offline con Service Worker caching

<!-- Screenshot del portal cliente (vista monitor) -->
![Portal Cliente — Monitor](docs/images/client-monitor.png)

<!-- Screenshot de gráficos históricos -->
![Historial de Temperatura](docs/images/client-history.png)

### Consola Administrativa
- Gestión global de flota multi-tenant
- Provisioning de clientes SaaS (planes, suscripciones, credenciales)
- Repositorio de firmwares con versionado de binarios .bin
- Modo impersonación para soporte remoto
- Log de tráfico de red (stream MQTT en vivo)
- Configuración granular de buzzer/alarmas por sensor

<!-- Screenshot de la consola admin -->
![Consola Admin — Flota](docs/images/admin-fleet.png)

<!-- Screenshot del log de tráfico MQTT -->
![Admin — Tráfico MQTT](docs/images/admin-mqtt.png)

### Sistema de Alertas
- Monitoreo de umbrales de temperatura (server-side)
- Detección de alarmas nativas del dispositivo
- Alertas de puerta abierta con delay configurable
- Cooldown de 1 minuto para prevenir spam
- Notificaciones WhatsApp vía WAHA
- Toggle de activación/desactivación por sensor

<!-- Screenshot de alertas o notificación WhatsApp -->
![Alertas y WhatsApp](docs/images/alerts.png)

## Inicio Rápido

### Requisitos
- Node.js 18+
- npm

### Instalación

```bash
git clone https://github.com/flavioGonz/cold-sense-landing.git
cd cold-sense-landing/server
npm install
```

### Configuración

Crear un archivo `.env` en `/server`:

```env
PORT=4000
JWT_SECRET=tu_clave_secreta_segura
```

### Ejecución

```bash
# Desarrollo (con auto-reload)
npm run dev

# Producción
npm start
```

El servidor inicia en:
- **HTTP/WebSocket**: `http://localhost:4000`
- **Broker MQTT**: `mqtt://localhost:1883`

### Base de Datos

SQLite se auto-inicializa en el primer arranque. Scripts de migración disponibles para actualizar el esquema:

```bash
node migrate_v5.js   # Campos WhatsApp + tabla de alertas
node migrate_v6.js   # Columnas de configuración de buzzer
node migrate_v7.js   # Mejoras al historial de comandos
```

## Topics MQTT (Protocolo Mauri)

| Topic | Dirección | Propósito |
|-------|-----------|-----------|
| `coldsense/pending/{ID}` | Dispositivo → Servidor | Anuncio de descubrimiento |
| `Friolink/config/{ID}` | Servidor → Dispositivo | Configuración (retained) |
| `Friolink/cmd/{ID}` | Servidor → Dispositivo | Comandos remotos |
| `coldsense/data/{ID}` | Dispositivo → Servidor | Telemetría |
| `coldsense/ack/{ID}` | Dispositivo → Servidor | Confirmación de comando |
| `clients/{CID}/sensors/{ID}/cmd` | Servidor → Dispositivo | Comandos con scope SaaS |

## Resumen de API

| Grupo | Endpoints |
|-------|----------|
| Auth | `POST /api/auth/login`, `GET /api/auth/me`, `POST /api/auth/logout` |
| Admin Sensores | `GET/PUT/DELETE /api/admin/sensors/:id`, `POST .../command` |
| Admin Clientes | `GET/POST/PUT/DELETE /api/admin/clients/:id` |
| Admin Firmware | `GET/POST/DELETE /api/admin/firmwares` |
| Cliente Sensores | `GET /api/client/sensors`, telemetría, alertas, sucursales |
| Alertas | `GET /api/admin/alerts`, `GET /api/client/alerts`, `PUT .../resolve` |
| Dispositivo | `GET /time`, `POST /devices/:id/readings/batch` |

## Firmware

Target: **ESP32-C6** (Pulsar Edition by UNIT Electronics)

| Componente | Modelo | GPIO |
|------------|--------|------|
| Temp Interior | DS18B20 (1-Wire) | 4 |
| Temp+Humedad Exterior | DHT11 | 5 |
| Sensor de Puerta | Reed Switch (NC) | 20 |
| LED Verde (WiFi) | — | 18 |
| LED Azul (MQTT) | — | 19 |
| Buzzer | Activo | 21 |
| Factory Reset | Botón Boot | 9 |

Especificación completa del protocolo: [SPEC_FIRMWARE_MAURI.md](./SPEC_FIRMWARE_MAURI.md)

<!-- Diagrama de pines o foto del PCB -->
![Pinout ESP32-C6](docs/images/pinout.png)

## Capturas

| Vista | Preview |
|-------|---------|
| Login | ![Login](docs/images/login.png) |
| Monitor Cliente | ![Monitor](docs/images/client-monitor.png) |
| Admin Flota | ![Admin](docs/images/admin-fleet.png) |
| Alertas | ![Alertas](docs/images/alerts.png) |
| PWA Mobile | ![Mobile](docs/images/mobile.png) |

## Estructura del Proyecto

```
server/
├── index.js            # Servidor principal (Express + Aedes + Socket.io)
├── database.js         # Inicialización del esquema SQLite
├── database.sqlite     # Base de datos SQLite (auto-creada)
├── migrate_v5-v7.js    # Scripts de migración de esquema
├── package.json
└── public/
    ├── index.html      # Portal cliente
    ├── admin.html      # Consola administrativa
    ├── client.js       # Lógica client-side
    ├── admin.js        # Lógica del panel admin
    ├── style.css       # Hoja de estilos
    ├── login.html      # Página de autenticación
    ├── sw.js           # Service Worker (PWA)
    ├── manifest.json   # Manifiesto PWA
    └── uploads/        # Binarios de firmware

firmware/
└── ColdSense_Pro_v3.2.ino   # Código fuente ESP32-C6
```

## Licencia

Desarrollado por **Antigravity AI** para el equipo **Friolink**.
Especializado en ecosistemas IoT industriales de alta confiabilidad.

© 2026 Friolink Global.
