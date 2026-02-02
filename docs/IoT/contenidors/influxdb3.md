# InfluxDB 3 i InfluxDB3-UI

## Què és?

**InfluxDB** és una base de dades de sèries temporals (time-series database) optimitzada per a emmagatzemar i consultar dades amb timestamps, ideal per a IoT, monitorització i mètriques.

**InfluxDB 3** és la nova versió que utilitza Apache Arrow i DataFusion per a un rendiment molt superior. Suporta tant InfluxQL com SQL.

**InfluxDB3-UI** és una interfície web per gestionar i visualitzar les dades d'InfluxDB 3.

### Característiques principals

- Optimitzada per a dades de sèries temporals
- Suport per SQL i InfluxQL
- Alta compressió de dades
- Escriptura d'alt rendiment

## Instal·lació amb Docker

### 1. Crear xarxa Docker (si no existeix)

```bash
docker network create xarxa_IOT
```

### 2. Crear carpetes

```bash
# Directoris per InfluxDB3
sudo mkdir -p /opt/docker/influxdb3/data
sudo mkdir -p /opt/docker/influxdb3/plugins

# Directoris per la UI
mkdir -p /opt/docker/influxdb3/ui/db
mkdir -p /opt/docker/influxdb3/ui/config
mkdir -p /opt/docker/influxdb3/ui/ssl

# Permisos
sudo chown -R 1000:1000 /opt/docker/influxdb3/
sudo chown -R 1500:1500 /opt/docker/influxdb3/data
sudo chmod -R 755 /opt/docker/influxdb3/
```

### 3. Generar secret per la UI

```bash
echo "SESSION_SECRET_KEY=$(openssl rand -base64 32)" > /opt/docker/influxdb3/.env
```

### 4. Docker Compose

```yaml
services:
  influxdb3:
    image: influxdb:3-core
    container_name: influxdb3
    restart: unless-stopped
    ports:
      - "8181:8181"
    volumes:
      - /opt/docker/influxdb3/data:/var/lib/influxdb3
      - /opt/docker/influxdb3/plugins:/plugins
    environment:
      - INFLUXDB3_NODE_IDENTIFER_PREFIX=iot-node
    networks:
      - xarxa_IOT

  influxdb3-ui:
    image: influxdata/influxdb3-ui:latest
    container_name: influxdb3-ui
    restart: unless-stopped
    ports:
      - "8888:8888"
    volumes:
      - /opt/docker/influxdb3/ui/db:/app/db
      - /opt/docker/influxdb3/ui/config:/app/config
      - /opt/docker/influxdb3/ui/ssl:/app/ssl
    env_file:
      - /opt/docker/influxdb3/.env
    depends_on:
      - influxdb3
    networks:
      - xarxa_IOT

networks:
  xarxa_IOT:
    external: true
```

### 5. Iniciar el servei

```bash
docker compose up -d
```

## Configuració post-instal·lació

### Generar token d'administrador

```bash
docker exec -it influxdb3 influxdb3 create token --admin
```

Guardar el token generat (format `apiv3_xxxx`).

### Crear configuració de la UI

Crear l'arxiu `/opt/docker/influxdb3/ui/config/config.json`:

```json
{
  "DEFAULT_INFLUX_SERVER": "http://influxdb3:8181",
  "DEFAULT_INFLUX_DATABASE": "",
  "DEFAULT_API_TOKEN": "apiv3_EL_TEU_TOKEN",
  "DEFAULT_SERVER_NAME": ""
}
```

### Crear base de dades

```bash
docker exec -it influxdb3 influxdb3 create database IoT --token apiv3_EL_TEU_TOKEN
```

**Nota:** La versió Core (no Enterprise) només suporta admin tokens.

## Accés

| Servei | URL | Descripció |
|--------|-----|------------|
| InfluxDB3 API | `http://localhost:8181` | API HTTP |
| InfluxDB3-UI | `http://localhost:8888` | Interfície web |

## Comandes útils

```bash
# Llistar bases de dades
docker exec -it influxdb3 influxdb3 show databases --token apiv3_TOKEN

# Crear base de dades
docker exec -it influxdb3 influxdb3 create database NOM_DB --token apiv3_TOKEN

# Eliminar base de dades
docker exec -it influxdb3 influxdb3 delete database NOM_DB --token apiv3_TOKEN
```

## Ports

| Port | Servei | Descripció |
|------|--------|------------|
| 8181 | InfluxDB3 | API HTTP |
| 8888 | InfluxDB3-UI | Interfície web |
