# MQTT (Mosquitto)

## Què és?

MQTT (Message Queuing Telemetry Transport) és un protocol de missatgeria lleuger dissenyat per a dispositius amb recursos limitats i xarxes amb ample de banda reduït. És el protocol més utilitzat en IoT.

**Mosquitto** és un broker MQTT open source que implementa les versions 5.0, 3.1.1 i 3.1 del protocol.

### Característiques principals

- Protocol publish/subscribe
- Lleuger i eficient
- Suport per a QoS (Quality of Service) nivells 0, 1 i 2
- Retenció de missatges
- Last Will and Testament (LWT)
- Suport per WebSockets

## Instal·lació amb Docker

### 1. Crear xarxa Docker (si no existeix)

```bash
docker network create xarxa_IOT
```

### 2. Crear carpetes

```bash
mkdir -p /opt/docker/mqtt/data
mkdir -p /opt/docker/mqtt/config
chown 1883:1883 /opt/docker/mqtt/config /opt/docker/mqtt/data
```

### 3. Crear fitxer de configuració

Crear l'arxiu `/opt/docker/mqtt/config/mosquitto.conf`:

```properties
# Fitxer de passwords
password_file /mosquitto/config/pwd_usuaris

# MQTT Default listener (port estàndard)
listener 1883 0.0.0.0

# MQTT over WebSockets
listener 9001 0.0.0.0
protocol websockets

# Persistència
persistence true
persistence_location /mosquitto/data/

# Logs
log_dest file /mosquitto/log/mosquitto.log
log_dest stdout
```

### 4. Docker Compose

```yaml
services:
  mqtt:
    image: eclipse-mosquitto:latest
    container_name: mqtt
    restart: unless-stopped
    ports:
      - "1883:1883"   # MQTT
      - "9001:9001"   # WebSockets
    volumes:
      - /opt/docker/mqtt/config:/mosquitto/config
      - /opt/docker/mqtt/data:/mosquitto/data
      - /opt/docker/mqtt/log:/mosquitto/log
    networks:
      - xarxa_IOT

networks:
  xarxa_IOT:
    external: true
```

### 5. Crear usuaris

Un cop el contenidor estigui en marxa, crear usuaris amb:

```bash
# Crear primer usuari (opció -c crea el fitxer)
docker exec -it mqtt mosquitto_passwd -c /mosquitto/config/pwd_usuaris usuari1

# Afegir més usuaris (sense -c)
docker exec -it mqtt mosquitto_passwd /mosquitto/config/pwd_usuaris usuari2
```

Reiniciar el contenidor per aplicar els canvis:

```bash
docker restart mqtt
```

## Testejar la connexió

### Amb mosquitto_sub i mosquitto_pub

```bash
# Subscriure's a un topic
docker exec -it mqtt mosquitto_sub -h localhost -u usuari1 -P password -t "test/topic"

# Publicar un missatge (en una altra terminal)
docker exec -it mqtt mosquitto_pub -h localhost -u usuari1 -P password -t "test/topic" -m "Hola MQTT!"
```

## Ports

| Port | Protocol | Descripció |
|------|----------|------------|
| 1883 | TCP | MQTT estàndard |
| 8883 | TCP | MQTT amb TLS |
| 9001 | TCP | MQTT sobre WebSockets |
