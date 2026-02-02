# ThingsBoard

## Què és?

ThingsBoard és una plataforma IoT open source per a la gestió de dispositius, recol·lecció de dades, processament i visualització. Ofereix una solució completa per a projectes IoT amb dashboards, regles de processament i gestió d'alarmes.

### Característiques principals

- Gestió de dispositius i actius
- Dashboards personalitzables
- Motor de regles (Rule Engine)
- Suport multi-tenant
- Protocols: MQTT, HTTP, CoAP, LwM2M
- Integració amb plataformes cloud

### Edicions

- **Community Edition (CE):** Open source, gratuïta
- **Professional Edition (PE):** Funcionalitats avançades (de pagament)

## Instal·lació amb Docker

### 1. Crear xarxa Docker (si no existeix)

```bash
docker network create xarxa_IOT
```

### 2. Crear carpetes

```bash
mkdir -p /opt/docker/thingsboard/data
mkdir -p /opt/docker/thingsboard/logs
chown -R 799:799 /opt/docker/thingsboard/data
chown -R 799:799 /opt/docker/thingsboard/logs
```

### 3. Docker Compose

```yaml
services:
  thingsboard:
    image: thingsboard/tb-postgres:latest
    container_name: thingsboard
    restart: unless-stopped
    ports:
      - "8080:9090"      # UI i API REST
      - "1883:1883"      # MQTT
      - "7070:7070"      # Edge RPC
      - "5683-5688:5683-5688/udp"  # CoAP
    environment:
      - TB_QUEUE_TYPE=in-memory
    volumes:
      - /opt/docker/thingsboard/data:/data
      - /opt/docker/thingsboard/logs:/var/log/thingsboard
    networks:
      - xarxa_IOT

networks:
  xarxa_IOT:
    external: true
```

**Nota:** Si ja tens MQTT al port 1883, canvia el port de ThingsBoard:
```yaml
ports:
  - "8080:9090"
  - "1884:1883"   # MQTT en port alternatiu
```

### 4. Iniciar el servei

```bash
docker compose up -d
```

El primer inici pot trigar uns minuts mentre es configura la base de dades.

## Accés

- **URL:** `http://localhost:8080`
- **Usuari:** sysadmin@thingsboard.org
- **Password:** sysadmin

### Credencials per defecte

| Rol | Usuari | Password |
|-----|--------|----------|
| System Administrator | sysadmin@thingsboard.org | sysadmin |
| Tenant Administrator | tenant@thingsboard.org | tenant |

## Configuració inicial

### 1. Canviar passwords

Després del primer login, canviar immediatament els passwords per defecte.

### 2. Crear un dispositiu

1. Anar a **Devices** → **Add Device**
2. Donar un nom al dispositiu
3. Clicar **Add**
4. Obrir el dispositiu i copiar l'**Access Token**

### 3. Enviar dades al dispositiu

#### Via HTTP

```bash
curl -X POST \
  http://localhost:8080/api/v1/ACCESS_TOKEN/telemetry \
  -H "Content-Type: application/json" \
  -d '{"temperature": 25.5, "humidity": 60}'
```

#### Via MQTT

```bash
mosquitto_pub -h localhost -p 1883 \
  -t "v1/devices/me/telemetry" \
  -u "ACCESS_TOKEN" \
  -m '{"temperature": 25.5, "humidity": 60}'
```

## Crear un Dashboard

1. Anar a **Dashboards** → **Add Dashboard**
2. Donar un nom i clicar **Add**
3. Obrir el dashboard i clicar **Enter edit mode**
4. Afegir widgets arrossegant des de la biblioteca
5. Configurar cada widget per mostrar les dades del dispositiu

## Variables d'entorn útils

| Variable | Descripció | Valor per defecte |
|----------|------------|-------------------|
| `TB_QUEUE_TYPE` | Tipus de cua | in-memory |
| `SPRING_DATASOURCE_URL` | URL de la BD | PostgreSQL intern |
| `TB_SERVICE_ID` | ID del servei | tb-core |

## Ports

| Port | Protocol | Descripció |
|------|----------|------------|
| 9090 | HTTP | UI i API REST |
| 1883 | MQTT | Protocol MQTT |
| 7070 | gRPC | Edge RPC |
| 5683 | UDP | CoAP |

## Recursos

- [Documentació oficial](https://thingsboard.io/docs/)
- [GitHub](https://github.com/thingsboard/thingsboard)
