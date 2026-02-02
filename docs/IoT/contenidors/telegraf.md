# Telegraf

## Què és?

Telegraf és un agent de col·lecció de mètriques open source, part de l'stack TICK (Telegraf, InfluxDB, Chronograf, Kapacitor). Permet recollir, processar i enviar mètriques a diverses destinacions.

### Característiques principals

- Més de 300 plugins d'entrada (inputs)
- Múltiples sortides (outputs): InfluxDB, Prometheus, Kafka, etc.
- Baix consum de recursos
- Configuració basada en fitxers TOML
- Suport per MQTT, SNMP, HTTP, i molts altres protocols

## Instal·lació amb Docker

### 1. Crear xarxa Docker (si no existeix)

```bash
docker network create xarxa_IOT
```

### 2. Crear carpetes i configuració base

```bash
mkdir -p /opt/docker/telegraf/conf/telegraf.d

# Generar configuració per defecte
docker run --rm telegraf telegraf config > /opt/docker/telegraf/conf/telegraf.conf
```

### 3. Configurar sortida a InfluxDB

Editar `/opt/docker/telegraf/conf/telegraf.conf` i afegir/modificar la secció `outputs.influxdb_v2`:

```toml
[[outputs.influxdb_v2]]
  urls = ["http://influxdb:8086"]

  ## Token d'autenticació
  token = "EL_TEU_TOKEN"

  ## Organització (ha d'existir)
  organization = "La_Teva_Org"

  ## Bucket de destinació
  bucket = "telegraf"
```

Per a **InfluxDB 3**, utilitzar:

```toml
[[outputs.influxdb_v2]]
  urls = ["http://influxdb3:8181"]
  token = "apiv3_EL_TEU_TOKEN"
  organization = ""
  bucket = "IoT"
```

### 4. Configurar entrada MQTT (opcional)

Afegir a la configuració per rebre dades de MQTT:

```toml
[[inputs.mqtt_consumer]]
  servers = ["tcp://mqtt:1883"]
  topics = [
    "sensors/#",
    "devices/#"
  ]
  username = "telegraf"
  password = "password"
  data_format = "json"
```

### 5. Docker Compose

```yaml
services:
  telegraf:
    image: telegraf:latest
    container_name: telegraf
    restart: unless-stopped
    volumes:
      - /opt/docker/telegraf/conf/telegraf.conf:/etc/telegraf/telegraf.conf:ro
      - /opt/docker/telegraf/conf/telegraf.d:/etc/telegraf/telegraf.d:ro
    depends_on:
      - influxdb3
    networks:
      - xarxa_IOT

networks:
  xarxa_IOT:
    external: true
```

### 6. Iniciar el servei

```bash
docker compose up -d
```

## Exemples de configuració

### Input: Mètriques del sistema

```toml
[[inputs.cpu]]
  percpu = true
  totalcpu = true
  collect_cpu_time = false

[[inputs.mem]]

[[inputs.disk]]
  ignore_fs = ["tmpfs", "devtmpfs"]
```

### Input: HTTP Listener (rebre dades via HTTP)

```toml
[[inputs.http_listener_v2]]
  service_address = ":8080"
  paths = ["/telegraf"]
  data_format = "json"
```

### Processador: Afegir tags

```toml
[[processors.override]]
  [processors.override.tags]
    environment = "production"
    location = "barcelona"
```

## Verificar configuració

```bash
# Testejar la configuració
docker exec -it telegraf telegraf --config /etc/telegraf/telegraf.conf --test

# Veure logs
docker logs -f telegraf
```

## Plugins més utilitzats en IoT

| Plugin | Tipus | Descripció |
|--------|-------|------------|
| `mqtt_consumer` | Input | Subscriure's a topics MQTT |
| `http_listener_v2` | Input | Rebre dades via HTTP POST |
| `modbus` | Input | Llegir dispositius Modbus |
| `snmp` | Input | Consultar dispositius SNMP |
| `influxdb_v2` | Output | Enviar a InfluxDB 2.x/3.x |
| `mqtt` | Output | Publicar a MQTT |
