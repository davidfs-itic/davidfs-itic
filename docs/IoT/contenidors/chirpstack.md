# ChirpStack

## Què és?

ChirpStack és una plataforma open source per a xarxes LoRaWAN. Proporciona tots els components necessaris per desplegar i gestionar una xarxa LoRaWAN privada.

### Característiques principals

- Network Server LoRaWAN complet
- Gestió de gateways i dispositius
- Suport per LoRaWAN 1.0.x i 1.1.x
- Integració amb múltiples backends (MQTT, HTTP, InfluxDB, etc.)
- API REST i gRPC
- Interfície web d'administració

### Components

- **ChirpStack:** Server principal (v4 unifica tots els components anteriors)
- **ChirpStack Gateway Bridge:** Connecta gateways amb ChirpStack

## Instal·lació amb Docker

### 1. Crear xarxa Docker (si no existeix)

```bash
docker network create xarxa_IOT
```

### 2. Crear carpetes

```bash
mkdir -p /opt/docker/chirpstack/config
mkdir -p /opt/docker/chirpstack/postgres
mkdir -p /opt/docker/chirpstack/redis
```

### 3. Crear fitxer de configuració

Crear `/opt/docker/chirpstack/config/chirpstack.toml`:

```toml
[logging]
level = "info"

[postgresql]
dsn = "postgres://chirpstack:chirpstack@postgres/chirpstack?sslmode=disable"
max_open_connections = 10

[redis]
servers = ["redis://redis/"]

[network]
net_id = "000000"
enabled_regions = ["eu868"]

[gateway]
  [gateway.backend]
    [gateway.backend.mqtt]
    server = "tcp://mqtt:1883"
    username = ""
    password = ""

[integration]
  [integration.mqtt]
  server = "tcp://mqtt:1883"
  username = ""
  password = ""
```

### 4. Docker Compose

```yaml
services:
  chirpstack:
    image: chirpstack/chirpstack:4
    container_name: chirpstack
    restart: unless-stopped
    command: -c /etc/chirpstack
    ports:
      - "8080:8080"   # Web UI i API
    volumes:
      - /opt/docker/chirpstack/config:/etc/chirpstack
    depends_on:
      - postgres
      - redis
      - mqtt
    networks:
      - xarxa_IOT

  chirpstack-gateway-bridge:
    image: chirpstack/chirpstack-gateway-bridge:4
    container_name: chirpstack-gateway-bridge
    restart: unless-stopped
    ports:
      - "1700:1700/udp"   # Semtech UDP
    environment:
      - INTEGRATION__MQTT__EVENT_TOPIC_TEMPLATE=eu868/gateway/{{ .GatewayID }}/event/{{ .EventType }}
      - INTEGRATION__MQTT__STATE_TOPIC_TEMPLATE=eu868/gateway/{{ .GatewayID }}/state/{{ .StateType }}
      - INTEGRATION__MQTT__COMMAND_TOPIC_TEMPLATE=eu868/gateway/{{ .GatewayID }}/command/#
      - INTEGRATION__MQTT__AUTH__GENERIC__SERVERS=tcp://mqtt:1883
    depends_on:
      - mqtt
    networks:
      - xarxa_IOT

  postgres:
    image: postgres:14-alpine
    container_name: chirpstack-postgres
    restart: unless-stopped
    environment:
      - POSTGRES_DB=chirpstack
      - POSTGRES_USER=chirpstack
      - POSTGRES_PASSWORD=chirpstack
    volumes:
      - /opt/docker/chirpstack/postgres:/var/lib/postgresql/data
    networks:
      - xarxa_IOT

  redis:
    image: redis:7-alpine
    container_name: chirpstack-redis
    restart: unless-stopped
    volumes:
      - /opt/docker/chirpstack/redis:/data
    networks:
      - xarxa_IOT

networks:
  xarxa_IOT:
    external: true
```

**Nota:** Aquest docker-compose assumeix que ja tens un broker MQTT (`mqtt`) a la xarxa. Si no, afegeix el servei MQTT.

### 5. Iniciar els serveis

```bash
docker compose up -d
```

## Accés

- **URL:** `http://localhost:8080`
- **Usuari:** admin
- **Password:** admin

## Configuració inicial

### 1. Canviar password

Després del primer login, canviar el password de l'administrador.

### 2. Crear un Device Profile

1. Anar a **Device profiles** → **Add device profile**
2. Configurar:
   - **Name:** Nom del perfil
   - **Region:** EU868
   - **MAC version:** LoRaWAN 1.0.3 (o la versió del dispositiu)
   - **Regional parameters revision:** A o B

### 3. Crear una Application

1. Anar a **Applications** → **Add application**
2. Donar un nom a l'aplicació

### 4. Afegir un dispositiu

1. Dins de l'aplicació, anar a **Devices** → **Add device**
2. Configurar:
   - **Name:** Nom del dispositiu
   - **Device EUI:** EUI del dispositiu (16 caràcters hex)
   - **Device profile:** Seleccionar el perfil creat

### 5. Configurar claus OTAA/ABP

Segons el mode d'activació del dispositiu, configurar les claus corresponents.

## Integració MQTT

ChirpStack publica els events dels dispositius a topics MQTT:

```
application/APPLICATION_ID/device/DEV_EUI/event/up
application/APPLICATION_ID/device/DEV_EUI/event/join
application/APPLICATION_ID/device/DEV_EUI/event/status
```

Per subscriure's a tots els events:

```bash
mosquitto_sub -h localhost -t "application/#" -v
```

## Configuració de Gateway

### Configurar el gateway LoRa

El gateway s'ha de configurar per enviar dades al Gateway Bridge:

- **Server:** IP del servidor ChirpStack
- **Port:** 1700 (UDP)

## Ports

| Port | Protocol | Descripció |
|------|----------|------------|
| 8080 | HTTP | Web UI i API |
| 1700 | UDP | Semtech UDP (gateways) |

## Recursos

- [Documentació oficial](https://www.chirpstack.io/docs/)
- [GitHub](https://github.com/chirpstack/chirpstack)
