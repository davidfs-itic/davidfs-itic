# Node-RED

## Què és?

Node-RED és una eina de programació visual basada en flux (flow-based programming) construïda sobre Node.js. Permet connectar dispositius, APIs i serveis de manera senzilla mitjançant una interfície de drag-and-drop al navegador.

### Característiques principals

- Programació visual amb nodes
- Extensa biblioteca de nodes (palettes)
- Integració nativa amb MQTT, HTTP, WebSocket
- Fàcil integració amb serveis IoT
- Exportació/importació de fluxos en format JSON

Referència oficial: [Node-RED Docker](https://nodered.org/docs/getting-started/docker)

## Instal·lació amb Docker

### 1. Crear xarxa Docker (si no existeix)

```bash
docker network create xarxa_IOT
```

### 2. Crear carpetes

```bash
sudo mkdir -p /opt/docker/node-red/data/
chown 1000:1000 /opt/docker/node-red/data/
chmod 750 /opt/docker/node-red/data/
```

L'usuari 1000 és l'usuari `node-red` dins del contenidor.

### 3. Docker Compose

```yaml
services:
  nodered:
    image: nodered/node-red:latest
    container_name: nodered
    restart: unless-stopped
    ports:
      - "1880:1880"
    environment:
      - TZ=Europe/Madrid
    volumes:
      - /opt/docker/node-red/data:/data
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

- **URL:** `http://localhost:1880`

## Configurar autenticació

Per defecte, Node-RED no requereix autenticació. Per habilitar-la:

### 1. Generar hash de password

```bash
docker exec -it nodered node-red admin hash-pw
```

Introduir el password i copiar el hash generat (format `$2b$...`).

### 2. Editar settings.js

Editar l'arxiu `/opt/docker/node-red/data/settings.js` i afegir:

```javascript
adminAuth: {
    type: "credentials",
    users: [{
        username: "admin",
        password: "$2b$08$HASH_GENERAT",
        permissions: "*"
    },
    {
        username: "viewer",
        password: "$2b$08$ALTRE_HASH",
        permissions: "read"
    }]
},
```

### 3. Reiniciar el contenidor

```bash
docker restart nodered
```

## Instal·lar nodes addicionals

### Des de la interfície web

1. Anar a **Menu** (≡) → **Manage palette**
2. Pestanya **Install**
3. Cercar el node desitjat i clicar **Install**

### Des de la línia de comandes

```bash
docker exec -it nodered npm install node-red-contrib-influxdb
docker restart nodered
```

## Nodes útils per IoT

| Node | Descripció |
|------|------------|
| `node-red-contrib-influxdb` | Connexió amb InfluxDB |
| `node-red-contrib-modbus` | Comunicació Modbus |
| `node-red-dashboard` | Crear dashboards web |
| `node-red-contrib-opcua` | Comunicació OPC-UA |
| `node-red-contrib-telegrambot` | Integració amb Telegram |

## Exemple: Rebre dades MQTT i enviar a InfluxDB

1. Afegir node **mqtt in** i configurar:
   - Server: `mqtt:1883`
   - Topic: `sensors/#`

2. Afegir node **json** per parsejar el missatge

3. Afegir node **influxdb out** i configurar:
   - Server: `http://influxdb3:8181`
   - Database: `IoT`

4. Connectar els nodes i fer **Deploy**

## Ports

| Port | Descripció |
|------|------------|
| 1880 | Interfície web i API |
