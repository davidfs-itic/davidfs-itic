# Grafana

## Què és?

Grafana és una plataforma open source per a visualització i monitorització de dades. Permet crear dashboards interactius amb gràfics, alertes i panells personalitzats.

### Característiques principals

- Suport per múltiples fonts de dades (InfluxDB, Prometheus, MySQL, PostgreSQL, etc.)
- Dashboards interactius i personalitzables
- Sistema d'alertes


## Instal·lació amb Docker

### 1. Crear xarxa Docker (si no existeix)

```bash
docker network create xarxa_IOT
```

### 2. Crear carpetes

```bash
mkdir -p /opt/docker/grafana/data
chown 472:472 /opt/docker/grafana/data
```

L'usuari 472 és l'usuari `grafana` dins del contenidor.

### 3. Docker Compose

```yaml
services:
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_USERS_ALLOW_SIGN_UP=false
    volumes:
      - /opt/docker/grafana/data:/var/lib/grafana
    networks:
      - xarxa_IOT

networks:
  xarxa_IOT:
    external: true
```

### 4. Iniciar el servei

```bash
docker compose up -d
```

## Accés

- **URL:** `http://localhost:3000`
- **Usuari:** admin
- **Password:** admin (canviar al primer accés)

## Configurar font de dades (InfluxDB)

1. Anar a **Configuration** → **Data Sources**
2. Clicar **Add data source**
3. Seleccionar **InfluxDB**
4. Configurar:
   - **URL:** `http://influxdb:8086` (o `http://influxdb3:8181` per InfluxDB3)
   - **Organization:** El nom de l'organització
   - **Token:** El token d'API
   - **Default Bucket:** El bucket per defecte

## Variables d'entorn útils

| Variable | Descripció | Valor per defecte |
|----------|------------|-------------------|
| `GF_SECURITY_ADMIN_USER` | Usuari administrador | admin |
| `GF_SECURITY_ADMIN_PASSWORD` | Password administrador | admin |
| `GF_USERS_ALLOW_SIGN_UP` | Permetre registre | true |
| `GF_SERVER_ROOT_URL` | URL base | http://localhost:3000 |
| `GF_SMTP_ENABLED` | Habilitar SMTP per alertes | false |

## Ports

| Port | Descripció |
|------|------------|
| 3000 | Interfície web |
