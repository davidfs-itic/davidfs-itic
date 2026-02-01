
# Modbus TCP - Llibreries i exemples

## Introducció

Modbus TCP és l'adaptació del protocol Modbus sobre TCP/IP. És un dels protocols industrials més utilitzats per la seva simplicitat i àmplia compatibilitat.

## Llibreries disponibles

| Llibreria | Llenguatge | Rol | Llicència |
|-----------|------------|-----|-----------|
| **pymodbus** | Python | Client i Server | BSD |
| **modbus-serial** | Node.js | Client | ISC |
| **jsmodbus** | Node.js | Client i Server | MIT |
| **node-red-contrib-modbus** | Node-RED | Client i Server | BSD |

---

## Python: pymodbus

La llibreria més completa per Modbus en Python. Suporta Modbus TCP, RTU i ASCII.

### Instal·lació

```bash
pip install pymodbus
```

### Client bàsic

```python
from pymodbus.client import ModbusTcpClient

# Connexió
client = ModbusTcpClient('192.168.1.100', port=502)
client.connect()

# Llegir 10 holding registers a partir de l'adreça 0
result = client.read_holding_registers(address=0, count=10, slave=1)
if not result.isError():
    print(f"Registres: {result.registers}")

# Tancar connexió
client.close()
```

### Funcions de lectura

```python
from pymodbus.client import ModbusTcpClient

client = ModbusTcpClient('192.168.1.100', port=502)
client.connect()

# Funció 01: Read Coils (sortides digitals)
coils = client.read_coils(address=0, count=16, slave=1)
print(f"Coils: {coils.bits[:16]}")

# Funció 02: Read Discrete Inputs (entrades digitals)
inputs = client.read_discrete_inputs(address=0, count=16, slave=1)
print(f"Inputs: {inputs.bits[:16]}")

# Funció 03: Read Holding Registers (registres R/W)
holding = client.read_holding_registers(address=0, count=10, slave=1)
print(f"Holding: {holding.registers}")

# Funció 04: Read Input Registers (registres només lectura)
input_regs = client.read_input_registers(address=0, count=10, slave=1)
print(f"Input Registers: {input_regs.registers}")

client.close()
```

### Funcions d'escriptura

```python
from pymodbus.client import ModbusTcpClient

client = ModbusTcpClient('192.168.1.100', port=502)
client.connect()

# Funció 05: Write Single Coil
client.write_coil(address=0, value=True, slave=1)

# Funció 06: Write Single Register
client.write_register(address=0, value=1234, slave=1)

# Funció 15: Write Multiple Coils
client.write_coils(address=0, values=[True, False, True, True], slave=1)

# Funció 16: Write Multiple Registers
client.write_registers(address=0, values=[100, 200, 300, 400], slave=1)

client.close()
```

### Treballar amb tipus de dades

Modbus només treballa amb registres de 16 bits. Per a altres tipus cal combinar registres:

```python
from pymodbus.client import ModbusTcpClient
from pymodbus.payload import BinaryPayloadDecoder, BinaryPayloadBuilder
from pymodbus.constants import Endian

client = ModbusTcpClient('192.168.1.100', port=502)
client.connect()

# Llegir un valor float (32 bits = 2 registres)
result = client.read_holding_registers(address=0, count=2, slave=1)
decoder = BinaryPayloadDecoder.fromRegisters(
    result.registers,
    byteorder=Endian.BIG,
    wordorder=Endian.BIG
)
float_value = decoder.decode_32bit_float()
print(f"Float: {float_value}")

# Llegir un valor int32 (32 bits = 2 registres)
result = client.read_holding_registers(address=2, count=2, slave=1)
decoder = BinaryPayloadDecoder.fromRegisters(
    result.registers,
    byteorder=Endian.BIG,
    wordorder=Endian.BIG
)
int32_value = decoder.decode_32bit_int()
print(f"Int32: {int32_value}")

# Escriure un valor float
builder = BinaryPayloadBuilder(byteorder=Endian.BIG, wordorder=Endian.BIG)
builder.add_32bit_float(3.14159)
registers = builder.to_registers()
client.write_registers(address=0, values=registers, slave=1)

# Escriure un valor int32
builder = BinaryPayloadBuilder(byteorder=Endian.BIG, wordorder=Endian.BIG)
builder.add_32bit_int(123456)
registers = builder.to_registers()
client.write_registers(address=2, values=registers, slave=1)

client.close()
```

### Llegir strings

```python
from pymodbus.client import ModbusTcpClient
from pymodbus.payload import BinaryPayloadDecoder
from pymodbus.constants import Endian

client = ModbusTcpClient('192.168.1.100', port=502)
client.connect()

# Llegir string (10 registres = 20 caràcters màxim)
result = client.read_holding_registers(address=100, count=10, slave=1)
decoder = BinaryPayloadDecoder.fromRegisters(
    result.registers,
    byteorder=Endian.BIG,
    wordorder=Endian.BIG
)
string_value = decoder.decode_string(20).decode('utf-8').strip('\x00')
print(f"String: {string_value}")

client.close()
```

### Client asíncron

```python
import asyncio
from pymodbus.client import AsyncModbusTcpClient

async def main():
    client = AsyncModbusTcpClient('192.168.1.100', port=502)
    await client.connect()

    # Llegir registres
    result = await client.read_holding_registers(address=0, count=10, slave=1)
    print(f"Registres: {result.registers}")

    # Escriure
    await client.write_register(address=0, value=999, slave=1)

    client.close()

asyncio.run(main())
```

### Servidor Modbus (simulador)

```python
from pymodbus.server import StartTcpServer
from pymodbus.datastore import ModbusSlaveContext, ModbusServerContext
from pymodbus.datastore import ModbusSequentialDataBlock

# Crear blocs de dades
# Paràmetres: adreça inicial, valors inicials
store = ModbusSlaveContext(
    di=ModbusSequentialDataBlock(0, [False] * 100),  # Discrete Inputs
    co=ModbusSequentialDataBlock(0, [False] * 100),  # Coils
    hr=ModbusSequentialDataBlock(0, [0] * 100),      # Holding Registers
    ir=ModbusSequentialDataBlock(0, [0] * 100)       # Input Registers
)

context = ModbusServerContext(slaves=store, single=True)

# Iniciar servidor
print("Servidor Modbus TCP iniciat a port 502...")
StartTcpServer(context=context, address=("0.0.0.0", 502))
```

### Gestió d'errors

```python
from pymodbus.client import ModbusTcpClient
from pymodbus.exceptions import ModbusException, ConnectionException

client = ModbusTcpClient('192.168.1.100', port=502, timeout=3)

try:
    if not client.connect():
        raise ConnectionException("No s'ha pogut connectar")

    result = client.read_holding_registers(address=0, count=10, slave=1)

    if result.isError():
        print(f"Error Modbus: {result}")
    else:
        print(f"Registres: {result.registers}")

except ConnectionException as e:
    print(f"Error de connexió: {e}")
except ModbusException as e:
    print(f"Error Modbus: {e}")
finally:
    client.close()
```

---

## Node.js: modbus-serial

Llibreria popular per a Node.js que suporta Modbus TCP i RTU.

### Instal·lació

```bash
npm install modbus-serial
```

### Client bàsic

```javascript
const ModbusRTU = require('modbus-serial');

const client = new ModbusRTU();

async function main() {
    // Connexió TCP
    await client.connectTCP('192.168.1.100', { port: 502 });
    client.setID(1);  // Slave ID

    // Llegir 10 holding registers
    const data = await client.readHoldingRegisters(0, 10);
    console.log('Registres:', data.data);

    client.close();
}

main().catch(console.error);
```

### Funcions de lectura

```javascript
const ModbusRTU = require('modbus-serial');

const client = new ModbusRTU();

async function readAll() {
    await client.connectTCP('192.168.1.100', { port: 502 });
    client.setID(1);

    // Funció 01: Read Coils
    const coils = await client.readCoils(0, 16);
    console.log('Coils:', coils.data);

    // Funció 02: Read Discrete Inputs
    const inputs = await client.readDiscreteInputs(0, 16);
    console.log('Inputs:', inputs.data);

    // Funció 03: Read Holding Registers
    const holding = await client.readHoldingRegisters(0, 10);
    console.log('Holding:', holding.data);

    // Funció 04: Read Input Registers
    const inputRegs = await client.readInputRegisters(0, 10);
    console.log('Input Registers:', inputRegs.data);

    client.close();
}

readAll().catch(console.error);
```

### Funcions d'escriptura

```javascript
const ModbusRTU = require('modbus-serial');

const client = new ModbusRTU();

async function writeData() {
    await client.connectTCP('192.168.1.100', { port: 502 });
    client.setID(1);

    // Funció 05: Write Single Coil
    await client.writeCoil(0, true);

    // Funció 06: Write Single Register
    await client.writeRegister(0, 1234);

    // Funció 15: Write Multiple Coils
    await client.writeCoils(0, [true, false, true, true]);

    // Funció 16: Write Multiple Registers
    await client.writeRegisters(0, [100, 200, 300, 400]);

    console.log('Escriptura completada');
    client.close();
}

writeData().catch(console.error);
```

### Treballar amb floats

```javascript
const ModbusRTU = require('modbus-serial');

const client = new ModbusRTU();

// Funció per convertir 2 registres a float32
function registersToFloat(reg1, reg2) {
    const buffer = Buffer.alloc(4);
    buffer.writeUInt16BE(reg1, 0);
    buffer.writeUInt16BE(reg2, 2);
    return buffer.readFloatBE(0);
}

// Funció per convertir float32 a 2 registres
function floatToRegisters(value) {
    const buffer = Buffer.alloc(4);
    buffer.writeFloatBE(value, 0);
    return [buffer.readUInt16BE(0), buffer.readUInt16BE(2)];
}

async function main() {
    await client.connectTCP('192.168.1.100', { port: 502 });
    client.setID(1);

    // Llegir float (2 registres)
    const data = await client.readHoldingRegisters(0, 2);
    const floatValue = registersToFloat(data.data[0], data.data[1]);
    console.log('Float:', floatValue);

    // Escriure float
    const registers = floatToRegisters(3.14159);
    await client.writeRegisters(0, registers);

    client.close();
}

main().catch(console.error);
```

### Polling continu

```javascript
const ModbusRTU = require('modbus-serial');

const client = new ModbusRTU();
let isConnected = false;

async function connect() {
    try {
        await client.connectTCP('192.168.1.100', { port: 502 });
        client.setID(1);
        client.setTimeout(2000);
        isConnected = true;
        console.log('Connectat');
    } catch (err) {
        console.error('Error de connexió:', err.message);
        isConnected = false;
    }
}

async function poll() {
    if (!isConnected) {
        await connect();
        return;
    }

    try {
        const data = await client.readHoldingRegisters(0, 10);
        console.log('Dades:', data.data);
    } catch (err) {
        console.error('Error de lectura:', err.message);
        isConnected = false;
    }
}

// Polling cada 500ms
connect().then(() => {
    setInterval(poll, 500);
});
```

### Gestió d'errors amb reconnexió

```javascript
const ModbusRTU = require('modbus-serial');

class ModbusClient {
    constructor(ip, port = 502, slaveId = 1) {
        this.ip = ip;
        this.port = port;
        this.slaveId = slaveId;
        this.client = new ModbusRTU();
        this.connected = false;
    }

    async connect() {
        try {
            await this.client.connectTCP(this.ip, { port: this.port });
            this.client.setID(this.slaveId);
            this.client.setTimeout(3000);
            this.connected = true;
            console.log(`Connectat a ${this.ip}:${this.port}`);
        } catch (err) {
            this.connected = false;
            throw err;
        }
    }

    async ensureConnected() {
        if (!this.connected) {
            await this.connect();
        }
    }

    async readRegisters(address, count) {
        await this.ensureConnected();
        try {
            return await this.client.readHoldingRegisters(address, count);
        } catch (err) {
            this.connected = false;
            throw err;
        }
    }

    async writeRegister(address, value) {
        await this.ensureConnected();
        try {
            return await this.client.writeRegister(address, value);
        } catch (err) {
            this.connected = false;
            throw err;
        }
    }

    close() {
        this.client.close();
        this.connected = false;
    }
}

// Ús
const plc = new ModbusClient('192.168.1.100');

async function main() {
    const data = await plc.readRegisters(0, 10);
    console.log(data.data);
}

main().catch(console.error);
```

---

## Node.js: jsmodbus

Alternativa amb suport per servidor.

### Instal·lació

```bash
npm install jsmodbus
```

### Client

```javascript
const Modbus = require('jsmodbus');
const net = require('net');

const socket = new net.Socket();
const client = new Modbus.client.TCP(socket, 1);  // unit ID = 1

socket.on('connect', async () => {
    try {
        // Llegir holding registers
        const resp = await client.readHoldingRegisters(0, 10);
        console.log('Registres:', resp.response.body.values);

        // Escriure un registre
        await client.writeSingleRegister(0, 1234);

        socket.end();
    } catch (err) {
        console.error(err);
    }
});

socket.on('error', console.error);
socket.connect({ host: '192.168.1.100', port: 502 });
```

### Servidor

```javascript
const Modbus = require('jsmodbus');
const net = require('net');

const server = new net.Server();
const modbusServer = new Modbus.server.TCP(server);

// Configurar dades inicials
modbusServer.holding.writeUInt16BE(100, 0);  // Registre 0 = 100
modbusServer.holding.writeUInt16BE(200, 2);  // Registre 1 = 200

server.listen(502, '0.0.0.0', () => {
    console.log('Servidor Modbus TCP escoltant a port 502');
});
```

---

## Node-RED

### Instal·lació

```bash
cd ~/.node-red
npm install node-red-contrib-modbus
```

### Nodes disponibles

- **modbus-client**: Configuració de connexió
- **modbus-read**: Llegir registres
- **modbus-write**: Escriure registres
- **modbus-getter**: Llegir sota demanda
- **modbus-flex-getter**: Lectura flexible
- **modbus-server**: Servidor Modbus

---

## Eines de diagnòstic

### Modbus Poll (Windows)
Client Modbus amb interfície gràfica per testejar dispositius.

### mbpoll (Linux)
```bash
# Instal·lació
sudo apt install mbpoll

# Llegir 10 holding registers
mbpoll -a 1 -r 0 -c 10 192.168.1.100

# Escriure un valor
mbpoll -a 1 -r 0 192.168.1.100 1234
```

### pymodbus.console
```bash
# Instal·lar
pip install pymodbus[repl]

# Iniciar consola
pymodbus.console tcp --host 192.168.1.100 --port 502

# Dins la consola:
> client.read_holding_registers 0 10 1
```

---

## Mapa de registres típic

Exemple d'organització de registres en un dispositiu:

| Adreça | Tipus | Descripció |
|--------|-------|------------|
| 0-9 | Holding | Setpoints |
| 10-19 | Holding | Paràmetres configuració |
| 100-109 | Input | Valors de procés |
| 110-119 | Input | Estat dels sensors |
| 0-15 | Coil | Sortides digitals |
| 0-15 | Discrete | Entrades digitals |

---

## Recursos

- [pymodbus GitHub](https://github.com/pymodbus-dev/pymodbus)
- [pymodbus Documentació](https://pymodbus.readthedocs.io/)
- [modbus-serial npm](https://www.npmjs.com/package/modbus-serial)
- [jsmodbus npm](https://www.npmjs.com/package/jsmodbus)
- [Modbus Organization](https://modbus.org/)
- [Especificació Modbus TCP](https://modbus.org/docs/Modbus_Messaging_Implementation_Guide_V1_0b.pdf)
