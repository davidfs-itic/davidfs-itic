
# Protocols de Comunicacions Industrials

## Introducció

### Què són els protocols industrials?

Un **protocol de comunicació industrial** és un conjunt de regles que defineixen com els dispositius d'un sistema d'automatització (PLCs, sensors, actuadors, HMIs, SCADA) intercanvien informació. A diferència dels protocols IT convencionals, els protocols industrials han de garantir:

- **Determinisme:** Les dades han d'arribar en un temps que es pugui predir.
- **Fiabilitat:** Tolerància a soroll elèctric i condicions adverses.
- **Temps real:** Resposta dins de terminis estrictes (mil·lisegons o microsegons)
- **Disponibilitat:** Funcionament continu 24/7

### Evolució històrica

```
1970s          1980s-90s           2000s              2010s-Actualitat
  │               │                  │                      │
  ▼               ▼                  ▼                      ▼
┌─────┐      ┌─────────┐      ┌────────────┐      ┌─────────────────┐
│Sèrie│  →   │Fieldbus │  →   │ Ethernet   │  →   │ IIoT / Indústria│
│RS232│      │Profibus │      │ Industrial │      │      4.0        │
│RS485│      │DeviceNet│      │EtherNet/IP │      │   OPC UA + TSN  │
└─────┘      │Modbus   │      │PROFINET    │      │   MQTT + Cloud  │
             └─────────┘      └────────────┘      └─────────────────┘
```

**Comunicació sèrie (1970s):**
Connexions punt a punt amb RS-232/RS-485. Simples però limitades en distància i nombre de dispositius.

**Fieldbus (1980s-90s):**
Xarxes industrials dedicades (Profibus, DeviceNet, Modbus RTU). Permetien connectar múltiples dispositius en un bus compartit.

**Ethernet Industrial (2000s):**
Adaptació d'Ethernet estàndard per entorns industrials. Aprofita infraestructura IT existent i permet velocitats més altes.

**IIoT i Indústria 4.0 (2010s-actualitat):**
Convergència IT/OT, connectivitat al núvol, i estàndards com OPC UA amb TSN per temps real sobre Ethernet estàndard.

### Model de capes en protocols industrials

Els protocols industrials s'organitzen sobre el model OSI, però amb adaptacions específiques:

```
┌─────────────────────────────────────────────────────────────────┐
│  Capa 7 - Aplicació                                             │
│  ┌─────────┐ ┌─────────┐ ┌──────┐ ┌────────┐ ┌────────────────┐ │
│  │ Modbus  │ │  FINS   │ │ CIP  │ │ OPC UA │ │ S7 (Siemens)   │ │
│  └─────────┘ └─────────┘ └──────┘ └────────┘ └────────────────┘ │
├─────────────────────────────────────────────────────────────────┤
│  Capa 4 - Transport                                             │
│  ┌─────────────────────┐  ┌─────────────────────┐               │
│  │        TCP          │  │        UDP          │               │
│  └─────────────────────┘  └─────────────────────┘               │
├─────────────────────────────────────────────────────────────────┤
│  Capa 3 - Xarxa                                                 │
│  ┌─────────────────────────────────────────────┐                │
│  │                    IP                       │                │
│  └─────────────────────────────────────────────┘                │
├─────────────────────────────────────────────────────────────────┤
│  Capa 2 - Enllaç                                                │
│  ┌─────────────────────┐  ┌─────────────────────┐               │
│  │  Ethernet estàndard │  │  PROFINET IRT/TSN   │               │
│  └─────────────────────┘  └─────────────────────┘               │
├─────────────────────────────────────────────────────────────────┤
│  Capa 1 - Física                                                │
│  ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌──────────────────┐ │
│  │ 100BASE-TX│ │1000BASE-T │ │   Fibra   │ │ Industrial (M12) │ │
│  └───────────┘ └───────────┘ └───────────┘ └──────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

### Diferències entre IT i OT

| Aspecte | IT (Tecnologies Informació) | OT (Tecnologies Operació) |
|---------|----------------------------|---------------------------|
| Prioritat | Confidencialitat | Disponibilitat |
| Latència | Tolerable (ms-s) | Crítica (µs-ms) |
| Cicle de vida | 3-5 anys | 15-20 anys |
| Actualitzacions | Freqüents | Poc freqüents |
| Fallades | Reinici acceptable | Inacceptable (seguretat) |
| Protocols | HTTP, SQL, REST | Modbus, OPC UA, PROFINET |

### Conceptes clau

#### Temps real vs Determinisme

Aquests dos conceptes estan molt relacionats però **no són sinònims**:

| Concepte | Definició | Pregunta clau |
|----------|-----------|---------------|
| **Temps real** | El sistema respon dins d'un termini definit (deadline) | *Quan* ha d'arribar la resposta? |
| **Determinisme** | El comportament del sistema és predictible i repetible | *Com de predictible* és el temps de resposta? |

**Temps real** no significa "molt ràpid". Un sistema pot tenir un deadline d'1 segon i ser temps real si **sempre** compleix aquest termini. El que importa és la **garantia** de complir el termini, no la velocitat absoluta.

**Determinisme** és la propietat que permet aconseguir temps real. Un sistema determinista té un temps de resposta amb **variació mínima** (baixa jitter). Sense determinisme, no pots garantir deadlines.

```
              Temps de resposta (latència)

No determinista (Ethernet estàndard):
    ├──────────────────────────────────────────────────────►
    │    ○      ○  ○       ○           ○    ○        ○
    │  (variació alta - no pots garantir deadline)

Determinista (PROFINET IRT, EtherCAT):
    ├──────────────────────────────────────────────────────►
    │         ○○○○○○○○
    │       (variació mínima - pots garantir deadline)
              ▲
              └── Latència consistent
```

**Relació pràctica:**

- Per tenir **temps real**, necessites **determinisme**
- Pots tenir determinisme sense requisits de temps real (sistema predictible però sense deadlines estrictes)
- Ethernet estàndard **no és determinista** perquè el temps d'accés al medi varia (col·lisions, congestió)

#### Tipus de temps real

| Tipus | Comportament si falla el deadline | Exemple |
|-------|----------------------------------|---------|
| **Hard Real-Time** | Fallada catastròfica del sistema | Control de moviment, airbag |
| **Firm Real-Time** | El resultat ja no és útil, però no és catastròfic | Vídeo en streaming |
| **Soft Real-Time** | Degradació acceptable de qualitat | Monitorització SCADA |


#### Com s'aconsegueix determinisme en Ethernet?

Ethernet estàndard no és determinista. Les solucions industrials modifiquen el comportament:

| Tecnologia | Mètode | Capa OSI |
|------------|--------|----------|
| **PROFINET IRT** | Reserves de temps (time slots) a capa 2 | Capa 2 |
| **EtherCAT** | Processament "on the fly" sense store-and-forward | Capa 2 |
| **TSN** | IEEE 802.1 - Prioritats, time-aware shaping | Capa 2 |
| **EtherNet/IP amb CIP Sync** | Sincronització de rellotges (IEEE 1588 PTP) | Capa 3-4 |



### Piràmide d'automatització i protocols

```
                    ┌───────────────┐
         Nivell 4   │   ERP/MES     │  ── HTTP, REST, SQL
                    │   (Gestió)    │
                    └───────┬───────┘
                            │ OPC UA, MQTT
                    ┌───────┴───────┐
         Nivell 3   │    SCADA      │  ── OPC UA, Modbus TCP
                    │  (Supervisió) │
                    └───────┬───────┘
                            │ EtherNet/IP, PROFINET
                    ┌───────┴───────┐
         Nivell 2   │  PLC / DCS    │  ── PROFINET, EtherNet/IP
                    │   (Control)   │
                    └───────┬───────┘
                            │ PROFINET, EtherCAT, IO-Link
                    ┌───────┴───────┐
         Nivell 1   │   I/O remota  │  ── Fieldbus, IO-Link
                    │  (Dispositius)│
                    └───────┬───────┘
                            │ Senyals elèctrics
                    ┌───────┴───────┐
         Nivell 0   │Sensors/Actuad.│  ── 4-20mA, 0-10V, Digital
                    │   (Camp)      │
                    └───────────────┘
```

---

## Comparativa ràpida

| Protocol | Capa | Port | Temps real | Seguretat | Complexitat |
|----------|------|------|------------|-----------|-------------|
| Modbus TCP | Aplicació/TCP | 502 | No | Baixa | Baixa |
| EtherNet/IP | Aplicació/TCP-UDP | 44818, 2222 | Sí (CIP Sync) | Mitjana | Mitjana |
| FINS | Aplicació/TCP-UDP | 9600 | No | Baixa | Baixa |
| OPC UA | Aplicació/TCP | 4840 | Sí (TSN) | Alta | Alta |
| PROFINET | Capa 2 + TCP/IP | - | Sí (IRT) | Mitjana | Alta |

---

## Modbus TCP

Protocol obert basat en l'arquitectura client/servidor. És l'evolució del Modbus RTU (sèrie) cap a Ethernet.

### Característiques

- **Port:** 502 (TCP)
- **Model:** Client (Master) / Servidor (Slave)
- **Adreçament:** Unit ID (0-255) + Registres (0-65535)
- **Format dades:** Big-endian

### Funcions principals

| Codi | Funció | Descripció |
|------|--------|------------|
| 01 | Read Coils | Llegir sortides digitals |
| 02 | Read Discrete Inputs | Llegir entrades digitals |
| 03 | Read Holding Registers | Llegir registres de 16 bits |
| 04 | Read Input Registers | Llegir registres d'entrada |
| 05 | Write Single Coil | Escriure una sortida |
| 06 | Write Single Register | Escriure un registre |
| 15 | Write Multiple Coils | Escriure múltiples sortides |
| 16 | Write Multiple Registers | Escriure múltiples registres |

### Llibreries disponibles

| Llibreria | Llenguatge | Rol | Llicència |
|-----------|------------|-----|-----------|
| **pymodbus** | Python | Client i Server | BSD |
| **modbus-serial** | Node.js | Client | ISC |
| **jsmodbus** | Node.js | Client i Server | MIT |


**[Exemples complets amb Python i Node.js](modbus-tcp.md)**

### Avantatges i inconvenients

- Simple d'implementar
- Àmpliament suportat
- Obert i sense llicències
- Sense seguretat integrada
- No és temps real
- Limitat a 65536 registres per tipus


## EtherNet/IP (CIP)

Protocol industrial desenvolupat per ODVA. Utilitza CIP (Common Industrial Protocol) sobre TCP/IP i UDP/IP.

### Característiques

- **Ports:** 44818 (TCP), 2222 (UDP per I/O implícit)
- **Model:** Productor/Consumidor i Client/Servidor
- **Suport:** Allen-Bradley (Rockwell), Omron Sysmac, altres

### Tipus de comunicació

1. **Explicit Messaging (TCP):** Peticions tipus request/response per configuració i diagnòstic
2. **Implicit Messaging (UDP):** Comunicació cíclica en temps real per I/O

### Conceptes clau

- **Tag-based:** Accés per nom de variable, no per adreça de memòria
- **Assemblies:** Agrupacions de dades per comunicació eficient
- **CIP Objects:** Model orientat a objectes (Identity, Connection Manager, etc.)

### Llibreries disponibles

| Llibreria | Llenguatge | PLCs suportats | Llicència |
|-----------|------------|----------------|-----------|
| **pycomm3** | Python | Allen-Bradley | MIT |
| **libplctag** | C, Python, .NET | Allen-Bradley, Omron | LGPL/MPL |
| **ethernet-ip** | Node.js | Allen-Bradley | MIT |
| **Kepware** | Multi | Multi-fabricant | Comercial |


**[Exemples complets amb Python i Node.js](ethernet-ip.md)**



## FINS (Factory Interface Network Service)

Protocol propietari d'Omron per comunicació amb PLCs de la sèrie CS, CJ, CP i NX/NJ.

### Característiques

- **Ports:** 9600 (TCP/UDP)
- **Model:** Client/Servidor
- **Adreçament:** Àrees de memòria (D, W, H, A, E, etc.)

### Comandes principals

| Codi | Descripció |
|------|------------|
| 0101 | Memory Area Read |
| 0102 | Memory Area Write |
| 0501 | Controller Data Read |
| 0601 | Controller Status Read |
| 0401 | Run/Stop |

### Àrees de memòria Omron

| Codi | Àrea | Descripció |
|------|------|------------|
| 82 | D | Data Memory |
| B0 | W | Work Area |
| B1 | H | Holding Area |
| 00 | CIO | Core I/O |

### Llibreries disponibles

| Llibreria | Llenguatge | Llicència |
|-----------|------------|-----------|
| **omron-fins** | Node.js | MIT |
| **Socket raw** | Python | - |

📖 **[Exemples complets amb Python i Node.js](fins.md)**



## OPC UA (Unified Architecture)

Estàndard internacional (IEC 62541) per interoperabilitat industrial. Successor d'OPC Classic (COM/DCOM).

### Característiques

- **Port:** 4840 (per defecte)
- **Model:** Client/Servidor i Pub/Sub
- **Seguretat:** Certificats X.509, xifrat, autenticació

### Arquitectura

```
┌─────────────────────────────────────────┐
│           OPC UA Application            │
├─────────────────────────────────────────┤
│     Information Model (Address Space)   │
├─────────────────────────────────────────┤
│  Services (Read, Write, Subscribe...)   │
├─────────────────────────────────────────┤
│   Security (Authentication, Encryption) │
├─────────────────────────────────────────┤
│        Transport (TCP, HTTPS, WSS)      │
└─────────────────────────────────────────┘
```

### Address Space

Els nodes s'organitzen en un espai d'adreces jeràrquic:

- **Objects:** Contenidors de variables i mètodes
- **Variables:** Valors llegibles/escrivibles
- **Methods:** Funcions executables
- **Views:** Vistes filtrades de l'espai d'adreces

### NodeId

Identificador únic de cada node:

```
ns=2;s=MyDevice.Temperature    (String)
ns=2;i=1234                     (Numeric)
```

### Exemple amb Python (opcua-asyncio)

```python
from asyncua import Client
import asyncio

async def main():
    async with Client("opc.tcp://192.168.1.100:4840") as client:
        # Llegir un node
        node = client.get_node("ns=2;s=Temperature")
        value = await node.read_value()
        print(f"Temperatura: {value}")

        # Subscripció a canvis
        handler = SubHandler()
        sub = await client.create_subscription(500, handler)
        await sub.subscribe_data_change(node)

        await asyncio.sleep(60)

class SubHandler:
    def datachange_notification(self, node, val, data):
        print(f"Canvi: {node} = {val}")

asyncio.run(main())
```

### Eines útils

- **UaExpert** - Client gràfic gratuït per explorar servidors
- **opcua-asyncio** - Llibreria Python
- **node-opcua** - Llibreria Node.js
- **open62541** - Implementació C open source

### Avantatges i inconvenients

✅ Seguretat integrada (certificats, xifrat)
✅ Interoperabilitat multi-fabricant
✅ Model de dades ric i extensible
✅ Companion Specifications per sectors
❌ Major complexitat d'implementació
❌ Requereix més recursos computacionals



## PROFINET

Protocol de Siemens basat en Ethernet industrial. Estàndard IEC 61158/61784.

### Classes de comunicació

| Classe | Nom | Cicle | Ús |
|--------|-----|-------|-----|
| CC-A | RT | 10-100 ms | Automatització estàndard |
| CC-B | RT | 1-10 ms | Aplicacions ràpides |
| CC-C | IRT | < 1 ms | Motion control, sincronització |

### Característiques

- **Capa 2 (IRT):** Accés directe a Ethernet per temps real
- **TCP/IP (RT):** Compatible amb infraestructura IT estàndard
- **GSD/GSDML:** Fitxers de descripció de dispositius

### Diferències amb EtherNet/IP

| Aspecte | PROFINET | EtherNet/IP |
|---------|----------|-------------|
| Origen | Siemens | Rockwell/ODVA |
| Temps real | IRT (Capa 2) | CIP Sync |
| Configuració | GSDML (XML) | EDS |
| Adopció | Europa | Amèrica |



## Altres protocols

### BACnet/IP
Protocol per automatització d'edificis (HVAC, il·luminació).
- Port UDP 47808

### DNP3
Protocol per sistemes SCADA en utilities (electricitat, aigua).
- Port TCP 20000

### MQTT
Protocol lleuger pub/sub per IoT. Molt usat per connectar dispositius industrials amb el núvol.
- Port TCP 1883 (sense TLS), 8883 (amb TLS)

### CoAP
Protocol REST-like per dispositius amb recursos limitats.
- Port UDP 5683



## Seguretat en protocols industrials

### Consideracions

1. **Segmentació de xarxa:** Separar OT de IT amb firewalls i DMZ
2. **Xifratge:** Utilitzar TLS quan sigui possible (OPC UA, MQTT)
3. **Autenticació:** Implementar control d'accés
4. **Monitorització:** Detectar anomalies en el tràfic
5. **Actualitzacions:** Mantenir firmware i software actualitzats

### Protocols i seguretat nativa

| Protocol | Xifratge | Autenticació |
|----------|----------|--------------|
| Modbus TCP | ❌ | ❌ |
| EtherNet/IP | ⚠️ CIP Security | ⚠️ Opcional |
| FINS | ❌ | ❌ |
| OPC UA | ✅ TLS | ✅ Certificats |
| PROFINET | ⚠️ Classe 4 | ⚠️ Opcional |


## Referències

- [Modbus Organization](https://modbus.org/)
- [ODVA - EtherNet/IP](https://www.odva.org/)
- [OPC Foundation](https://opcfoundation.org/)
- [PROFIBUS & PROFINET International](https://www.profibus.com/)
- [libplctag](https://github.com/libplctag/libplctag)
- [opcua-asyncio](https://github.com/FreeOpcUa/opcua-asyncio)
