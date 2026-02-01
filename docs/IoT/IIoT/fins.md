
# FINS (Factory Interface Network Service) - Llibreries i exemples

## Introducció

FINS és el protocol propietari d'Omron per comunicació amb PLCs. Suporta tant UDP com TCP i permet accedir a les àrees de memòria del PLC de forma directa.

### PLCs suportats

- Sèrie CS/CJ
- Sèrie CP
- Sèrie NJ/NX (amb retrocompatibilitat)
- Sèrie CV

## Llibreries disponibles

| Llibreria | Llenguatge | Llicència | Notes |
|-----------|------------|-----------|-------|
| **omron-fins** | Node.js | MIT | Més popular per Node |
| **fins** | Python | MIT | Implementació bàsica |
| **python-omron-fins** | Python | MIT | Més completa |
| Socket raw | Qualsevol | - | Implementació manual |

---

## Estructura del protocol FINS

### Capçalera FINS (10 bytes)

```
| ICF | RSV | GCT | DNA | DA1 | DA2 | SNA | SA1 | SA2 | SID |
```

| Byte | Camp | Descripció |
|------|------|------------|
| ICF | Information Control Field | 0x80 = comanda, 0xC0 = resposta |
| RSV | Reserved | Sempre 0x00 |
| GCT | Gateway Count | Nombre de gateways (normalment 0x02) |
| DNA | Destination Network Address | Xarxa destí (0x00 = local) |
| DA1 | Destination Node Address | Node destí |
| DA2 | Destination Unit Address | Unitat destí (0x00 = CPU) |
| SNA | Source Network Address | Xarxa origen |
| SA1 | Source Node Address | Node origen |
| SA2 | Source Unit Address | Unitat origen |
| SID | Service ID | Identificador de sessió |

### Àrees de memòria

| Codi | Àrea | Descripció | Accés |
|------|------|------------|-------|
| 0x00 | CIO | Core I/O | R/W |
| 0x80 | WR | Work Area (bit) | R/W |
| 0x81 | HR | Holding Area (bit) | R/W |
| 0x82 | DM | Data Memory | R/W |
| 0xB0 | WR | Work Area (word) | R/W |
| 0xB1 | HR | Holding Area (word) | R/W |
| 0xB2 | AR | Auxiliary Area | R |
| 0x02 | TIM/CNT | Timers/Counters PV | R/W |

### Comandes principals

| Codi | Nom | Descripció |
|------|-----|------------|
| 0101 | Memory Area Read | Llegir àrea de memòria |
| 0102 | Memory Area Write | Escriure àrea de memòria |
| 0103 | Memory Area Fill | Omplir àrea amb un valor |
| 0104 | Multiple Memory Area Read | Llegir múltiples àrees |
| 0401 | Run | Posar PLC en RUN |
| 0402 | Stop | Posar PLC en STOP |
| 0501 | Controller Data Read | Llegir info del PLC |
| 0601 | Controller Status Read | Llegir estat del PLC |
| 0701 | Cycle Time Read | Llegir temps de cicle |

---

## Python: Implementació amb sockets

### Client bàsic UDP

```python
import socket
import struct

class FinsClient:
    def __init__(self, plc_ip, plc_port=9600):
        self.plc_ip = plc_ip
        self.plc_port = plc_port
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        self.sock.settimeout(2.0)
        self.sid = 0

    def _build_header(self, da1=0x00):
        """Construeix la capçalera FINS"""
        self.sid = (self.sid + 1) % 256
        return bytes([
            0x80,  # ICF: Command
            0x00,  # RSV
            0x02,  # GCT
            0x00,  # DNA: Local network
            da1,   # DA1: Destination node
            0x00,  # DA2: CPU unit
            0x00,  # SNA: Local network
            0x01,  # SA1: Source node (PC)
            0x00,  # SA2: Source unit
            self.sid  # SID
        ])

    def _send_command(self, command, data=b''):
        """Envia una comanda FINS i retorna la resposta"""
        header = self._build_header()
        packet = header + command + data

        self.sock.sendto(packet, (self.plc_ip, self.plc_port))
        response, _ = self.sock.recvfrom(2048)

        # Verificar codis d'error (bytes 12-13 de la resposta)
        if len(response) >= 14:
            main_code = response[12]
            sub_code = response[13]
            if main_code != 0 or sub_code != 0:
                raise Exception(f"Error FINS: {main_code:02X}{sub_code:02X}")

        return response[14:]  # Retorna només les dades

    def read_dm(self, address, count):
        """Llegeix registres DM (Data Memory)"""
        command = bytes([0x01, 0x01])  # Memory Area Read
        # Àrea DM (0x82), adreça (2 bytes), bit (1 byte), quantitat (2 bytes)
        data = struct.pack('>BHBH', 0x82, address, 0x00, count)
        response = self._send_command(command, data)
        # Convertir a llista de valors
        values = []
        for i in range(0, len(response), 2):
            values.append(struct.unpack('>H', response[i:i+2])[0])
        return values

    def write_dm(self, address, values):
        """Escriu registres DM"""
        command = bytes([0x01, 0x02])  # Memory Area Write
        data = struct.pack('>BHBH', 0x82, address, 0x00, len(values))
        for v in values:
            data += struct.pack('>H', v)
        self._send_command(command, data)

    def read_cio(self, address, count):
        """Llegeix àrea CIO"""
        command = bytes([0x01, 0x01])
        data = struct.pack('>BHBH', 0x00, address, 0x00, count)
        response = self._send_command(command, data)
        values = []
        for i in range(0, len(response), 2):
            values.append(struct.unpack('>H', response[i:i+2])[0])
        return values

    def read_wr(self, address, count):
        """Llegeix Work Area"""
        command = bytes([0x01, 0x01])
        data = struct.pack('>BHBH', 0xB0, address, 0x00, count)
        response = self._send_command(command, data)
        values = []
        for i in range(0, len(response), 2):
            values.append(struct.unpack('>H', response[i:i+2])[0])
        return values

    def read_hr(self, address, count):
        """Llegeix Holding Area"""
        command = bytes([0x01, 0x01])
        data = struct.pack('>BHBH', 0xB1, address, 0x00, count)
        response = self._send_command(command, data)
        values = []
        for i in range(0, len(response), 2):
            values.append(struct.unpack('>H', response[i:i+2])[0])
        return values

    def get_controller_status(self):
        """Llegeix l'estat del controlador"""
        command = bytes([0x06, 0x01])
        response = self._send_command(command)
        status = response[0] if response else 0
        modes = {0x00: 'PROGRAM', 0x01: 'DEBUG', 0x02: 'MONITOR', 0x04: 'RUN'}
        return modes.get(status, f'UNKNOWN ({status:02X})')

    def get_controller_info(self):
        """Llegeix informació del controlador"""
        command = bytes([0x05, 0x01])
        response = self._send_command(command)
        if len(response) >= 20:
            model = response[0:20].decode('ascii').strip('\x00')
            return model
        return None

    def close(self):
        self.sock.close()


# Exemple d'ús
if __name__ == '__main__':
    plc = FinsClient('192.168.1.100')

    try:
        # Llegir estat
        status = plc.get_controller_status()
        print(f"Estat PLC: {status}")

        # Llegir 10 registres DM a partir de D0
        values = plc.read_dm(0, 10)
        print(f"D0-D9: {values}")

        # Escriure valors a D100
        plc.write_dm(100, [1234, 5678])
        print("Escriptura completada")

    finally:
        plc.close()
```

### Client TCP amb handshake

Per a FINS sobre TCP, cal fer un handshake inicial:

```python
import socket
import struct

class FinsTcpClient:
    def __init__(self, plc_ip, plc_port=9600):
        self.plc_ip = plc_ip
        self.plc_port = plc_port
        self.sock = None
        self.client_node = 0
        self.server_node = 0
        self.sid = 0

    def connect(self):
        """Connecta i fa el handshake FINS/TCP"""
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.sock.settimeout(5.0)
        self.sock.connect((self.plc_ip, self.plc_port))

        # Enviar comanda de connexió FINS/TCP
        # Header: FINS (4 bytes) + Length (4 bytes) + Command (4 bytes) + Error (4 bytes) + Client Node (4 bytes)
        connect_cmd = b'FINS'  # Magic
        connect_cmd += struct.pack('>I', 12)  # Length
        connect_cmd += struct.pack('>I', 0)   # Command: Node Address Data Send
        connect_cmd += struct.pack('>I', 0)   # Error code
        connect_cmd += struct.pack('>I', 0)   # Client node (0 = auto-assign)

        self.sock.send(connect_cmd)
        response = self.sock.recv(24)

        if response[:4] != b'FINS':
            raise Exception("Resposta de handshake invàlida")

        # Parsejar resposta
        error_code = struct.unpack('>I', response[12:16])[0]
        if error_code != 0:
            raise Exception(f"Error de connexió: {error_code}")

        self.client_node = struct.unpack('>I', response[16:20])[0]
        self.server_node = struct.unpack('>I', response[20:24])[0]
        print(f"Connectat: client={self.client_node}, server={self.server_node}")

    def _build_header(self):
        """Construeix capçalera FINS"""
        self.sid = (self.sid + 1) % 256
        return bytes([
            0x80,  # ICF
            0x00,  # RSV
            0x02,  # GCT
            0x00,  # DNA
            self.server_node,  # DA1
            0x00,  # DA2
            0x00,  # SNA
            self.client_node,  # SA1
            0x00,  # SA2
            self.sid  # SID
        ])

    def _send_command(self, command, data=b''):
        """Envia comanda FINS sobre TCP"""
        fins_header = self._build_header()
        fins_frame = fins_header + command + data

        # TCP header
        tcp_header = b'FINS'
        tcp_header += struct.pack('>I', len(fins_frame) + 8)  # Length
        tcp_header += struct.pack('>I', 2)  # Command: FINS Frame Send
        tcp_header += struct.pack('>I', 0)  # Error code

        self.sock.send(tcp_header + fins_frame)

        # Rebre resposta
        response = self.sock.recv(2048)

        # Saltar TCP header (16 bytes) + FINS header (10 bytes) + response codes (2 bytes)
        return response[28:]

    def read_dm(self, address, count):
        """Llegeix registres DM"""
        command = bytes([0x01, 0x01])
        data = struct.pack('>BHBH', 0x82, address, 0x00, count)
        response = self._send_command(command, data)
        values = []
        for i in range(0, len(response), 2):
            if i + 2 <= len(response):
                values.append(struct.unpack('>H', response[i:i+2])[0])
        return values

    def write_dm(self, address, values):
        """Escriu registres DM"""
        command = bytes([0x01, 0x02])
        data = struct.pack('>BHBH', 0x82, address, 0x00, len(values))
        for v in values:
            data += struct.pack('>H', v)
        self._send_command(command, data)

    def close(self):
        if self.sock:
            self.sock.close()


# Exemple d'ús
if __name__ == '__main__':
    plc = FinsTcpClient('192.168.1.100')

    try:
        plc.connect()
        values = plc.read_dm(0, 10)
        print(f"D0-D9: {values}")
    finally:
        plc.close()
```

### Treballar amb tipus de dades

```python
import struct

def registers_to_float(reg1, reg2):
    """Converteix 2 registres a float32 (Big Endian)"""
    data = struct.pack('>HH', reg1, reg2)
    return struct.unpack('>f', data)[0]

def float_to_registers(value):
    """Converteix float32 a 2 registres"""
    data = struct.pack('>f', value)
    return struct.unpack('>HH', data)

def registers_to_int32(reg1, reg2):
    """Converteix 2 registres a int32"""
    data = struct.pack('>HH', reg1, reg2)
    return struct.unpack('>i', data)[0]

def int32_to_registers(value):
    """Converteix int32 a 2 registres"""
    data = struct.pack('>i', value)
    return struct.unpack('>HH', data)

def registers_to_string(registers):
    """Converteix registres a string ASCII"""
    data = b''
    for reg in registers:
        data += struct.pack('>H', reg)
    return data.decode('ascii').strip('\x00')


# Exemple
plc = FinsClient('192.168.1.100')

# Llegir float (D0-D1)
regs = plc.read_dm(0, 2)
float_val = registers_to_float(regs[0], regs[1])
print(f"Float: {float_val}")

# Escriure float a D10-D11
regs = float_to_registers(3.14159)
plc.write_dm(10, list(regs))

# Llegir string (D100-D109, 20 caràcters)
regs = plc.read_dm(100, 10)
string_val = registers_to_string(regs)
print(f"String: {string_val}")

plc.close()
```

---

## Node.js: omron-fins

### Instal·lació

```bash
npm install omron-fins
```

### Client bàsic

```javascript
const fins = require('omron-fins');

// Configuració
const client = fins.FinsClient(9600, '192.168.1.100');

// Event de connexió
client.on('reply', (msg) => {
    if (msg.values) {
        console.log('Valors:', msg.values);
    }
});

client.on('error', (err) => {
    console.error('Error:', err);
});

client.on('timeout', () => {
    console.error('Timeout');
});

// Llegir 10 registres DM a partir de D0
client.read('D0', 10);

// Escriure valors
client.write('D100', [1234, 5678, 9012]);

// Llegir altres àrees
client.read('W0', 10);   // Work Area
client.read('H0', 10);   // Holding Area
client.read('CIO0', 10); // Core I/O
```

### Classe wrapper amb Promises

```javascript
const fins = require('omron-fins');

class OmronPLC {
    constructor(ip, port = 9600) {
        this.client = fins.FinsClient(port, ip);
        this.pending = new Map();
        this.requestId = 0;

        this.client.on('reply', (msg) => {
            const resolve = this.pending.get(msg.sid);
            if (resolve) {
                this.pending.delete(msg.sid);
                resolve(msg);
            }
        });

        this.client.on('error', (err) => {
            console.error('FINS Error:', err);
        });
    }

    read(address, count) {
        return new Promise((resolve, reject) => {
            const sid = this.client.read(address, count);
            this.pending.set(sid, resolve);

            setTimeout(() => {
                if (this.pending.has(sid)) {
                    this.pending.delete(sid);
                    reject(new Error('Timeout'));
                }
            }, 5000);
        });
    }

    write(address, values) {
        return new Promise((resolve, reject) => {
            const sid = this.client.write(address, values);
            this.pending.set(sid, resolve);

            setTimeout(() => {
                if (this.pending.has(sid)) {
                    this.pending.delete(sid);
                    reject(new Error('Timeout'));
                }
            }, 5000);
        });
    }
}

// Ús amb async/await
async function main() {
    const plc = new OmronPLC('192.168.1.100');

    try {
        // Llegir
        const result = await plc.read('D0', 10);
        console.log('D0-D9:', result.values);

        // Escriure
        await plc.write('D100', [100, 200, 300]);
        console.log('Escriptura completada');

    } catch (err) {
        console.error('Error:', err.message);
    }
}

main();
```

### Polling continu

```javascript
const fins = require('omron-fins');

const client = fins.FinsClient(9600, '192.168.1.100');

let lastValues = [];

client.on('reply', (msg) => {
    if (msg.values) {
        // Detectar canvis
        for (let i = 0; i < msg.values.length; i++) {
            if (lastValues[i] !== msg.values[i]) {
                console.log(`D${i} canviat: ${lastValues[i]} -> ${msg.values[i]}`);
            }
        }
        lastValues = [...msg.values];
    }
});

// Polling cada 500ms
setInterval(() => {
    client.read('D0', 10);
}, 500);
```

### Treballar amb floats

```javascript
const fins = require('omron-fins');

const client = fins.FinsClient(9600, '192.168.1.100');

// Convertir 2 registres a float32
function registersToFloat(reg1, reg2) {
    const buffer = Buffer.alloc(4);
    buffer.writeUInt16BE(reg1, 0);
    buffer.writeUInt16BE(reg2, 2);
    return buffer.readFloatBE(0);
}

// Convertir float32 a 2 registres
function floatToRegisters(value) {
    const buffer = Buffer.alloc(4);
    buffer.writeFloatBE(value, 0);
    return [buffer.readUInt16BE(0), buffer.readUInt16BE(2)];
}

client.on('reply', (msg) => {
    if (msg.values && msg.values.length >= 2) {
        const floatVal = registersToFloat(msg.values[0], msg.values[1]);
        console.log('Float value:', floatVal);
    }
});

// Llegir float (D0-D1)
client.read('D0', 2);

// Escriure float
const regs = floatToRegisters(3.14159);
client.write('D10', regs);
```

---

## Codis d'error FINS

### Codis d'error principals

| Codi | Descripció |
|------|------------|
| 0000 | Normal completion |
| 0001 | Service canceled |
| 0101 | Local node not in network |
| 0102 | Token timeout |
| 0103 | Retries failed |
| 0104 | Too many send frames |
| 0105 | Node address range error |
| 0106 | Node address duplication |
| 0201 | Destination node not in network |
| 0202 | Unit missing |
| 0203 | Third node missing |
| 0204 | Destination node busy |
| 0205 | Response timeout |
| 0401 | Command too long |
| 0402 | Command too short |
| 0501 | Destination CPU error |
| 1001 | Command not supported |
| 1101 | Area classification error |
| 1102 | Access size error |
| 1103 | Address range error |
| 1104 | Address range exceeded |
| 2002 | CPU protected |
| 2003 | PLC in program mode |

### Gestió d'errors en Python

```python
FINS_ERRORS = {
    0x0000: "Normal completion",
    0x0001: "Service canceled",
    0x0101: "Local node not in network",
    0x0201: "Destination node not in network",
    0x0205: "Response timeout",
    0x1001: "Command not supported",
    0x1101: "Area classification error",
    0x1103: "Address range error",
    0x2002: "CPU protected",
    0x2003: "PLC in program mode"
}

def check_fins_error(response):
    """Verifica i interpreta errors FINS"""
    if len(response) >= 14:
        main_code = response[12]
        sub_code = response[13]
        error_code = (main_code << 8) | sub_code

        if error_code != 0:
            error_msg = FINS_ERRORS.get(error_code, f"Unknown error")
            raise Exception(f"FINS Error {error_code:04X}: {error_msg}")
```

---

## Eines de diagnòstic

### Wireshark
Wireshark té un dissector per FINS que permet veure les trames decodificades.

Filtre: `omron`

### CX-Programmer
Eina oficial d'Omron per programar i monitoritzar PLCs. Permet veure l'estat de les àrees de memòria.

### Test amb netcat

```bash
# Enviar paquet FINS bàsic (no recomanat per producció)
echo -ne '\x80\x00\x02\x00\x00\x00\x00\x01\x00\x01\x05\x01' | nc -u 192.168.1.100 9600
```

---

## Comparativa UDP vs TCP

| Aspecte | FINS/UDP | FINS/TCP |
|---------|----------|----------|
| Port | 9600 | 9600 |
| Handshake | No | Sí |
| Fiabilitat | Menor | Major |
| Latència | Menor | Major |
| Firewalls | Més difícil | Més fàcil |
| Recomanat | Xarxa local | Xarxes complexes |

---

## Recursos

- [Omron FINS Protocol Reference](https://www.myomron.com) (requereix compte)
- [omron-fins npm](https://www.npmjs.com/package/omron-fins)
- [FINS Protocol Wireshark Wiki](https://wiki.wireshark.org/Protocols/omron-fins)
