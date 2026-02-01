
# EtherNet/IP - Llibreries i exemples

## Introducció

EtherNet/IP (Industrial Protocol) és un protocol industrial que utilitza CIP (Common Industrial Protocol) sobre TCP/IP i UDP/IP. És el protocol natiu dels PLCs Allen-Bradley (Rockwell Automation) i també és suportat per Omron Sysmac i altres fabricants.

## Llibreries disponibles

| Llibreria | Llenguatge | PLCs suportats | Llicència |
|-----------|------------|----------------|-----------|
| **pycomm3** | Python | Allen-Bradley (ControlLogix, CompactLogix, Micro800) | MIT |
| **libplctag** | C, Python, .NET | Allen-Bradley, Omron, Modbus | LGPL/MPL |
| **ethernet-ip** | Node.js | Allen-Bradley, genèric CIP | MIT |
| **node-red-contrib-cip-ethernet-ip** | Node-RED | Allen-Bradley | MIT |

---

## Python: pycomm3

La llibreria més popular i ben mantinguda per a Allen-Bradley en Python.

### Instal·lació

```bash
pip install pycomm3
```

### PLCs suportats

- ControlLogix (L6x, L7x, L8x)
- CompactLogix (L1x, L2x, L3x)
- Micro800 (810, 820, 830, 850)

### Exemple bàsic

```python
from pycomm3 import LogixDriver

# Connexió a un ControlLogix/CompactLogix
with LogixDriver('192.168.1.100') as plc:
    # Llegir un tag
    result = plc.read('MyTag')
    print(f"Valor: {result.value}")
    print(f"Tipus: {result.type}")

    # Escriure un valor
    plc.write('MyTag', 1234)
```

### Llegir i escriure múltiples tags

```python
from pycomm3 import LogixDriver

with LogixDriver('192.168.1.100') as plc:
    # Llegir múltiples tags (més eficient que llegir un a un)
    results = plc.read('Tag1', 'Tag2', 'Tag3')
    for tag in results:
        print(f"{tag.tag}: {tag.value}")

    # Escriure múltiples tags
    plc.write(('Tag1', 100), ('Tag2', 200), ('Tag3', 300))
```

### Treballar amb arrays

```python
from pycomm3 import LogixDriver

with LogixDriver('192.168.1.100') as plc:
    # Llegir un array complet
    array_data = plc.read('MyArray')
    print(array_data.value)  # Llista de valors

    # Llegir una porció de l'array (10 elements des de l'índex 5)
    partial = plc.read('MyArray[5]{10}')

    # Escriure a un array
    plc.write('MyArray', [1, 2, 3, 4, 5])

    # Escriure a una posició específica
    plc.write('MyArray[0]', 100)
```

### Treballar amb estructures (UDT)

```python
from pycomm3 import LogixDriver

with LogixDriver('192.168.1.100') as plc:
    # Llegir una estructura completa
    recipe = plc.read('Recipe1')
    print(recipe.value)  # Retorna un diccionari amb tots els membres
    # Exemple: {'Name': 'Recipe A', 'Temperature': 75.5, 'Time': 120}

    # Llegir un membre específic
    temp = plc.read('Recipe1.Temperature')
    print(f"Temperatura: {temp.value}")

    # Escriure a un membre de l'estructura
    plc.write('Recipe1.Temperature', 80.0)

    # Escriure estructura completa
    plc.write('Recipe1', {'Name': 'Recipe B', 'Temperature': 85.0, 'Time': 150})
```

### Obtenir llista de tags del PLC

```python
from pycomm3 import LogixDriver

with LogixDriver('192.168.1.100') as plc:
    # Obtenir tots els tags
    tags = plc.get_tag_list()

    for tag in tags:
        print(f"Nom: {tag['tag_name']}")
        print(f"Tipus: {tag['data_type']}")
        print(f"Dimensió: {tag.get('dimensions', 'N/A')}")
        print("---")

    # Filtrar per programa
    program_tags = plc.get_tag_list(program='MainProgram')
```

### Connexió amb slot específic

```python
from pycomm3 import LogixDriver

# Per defecte, slot 0
with LogixDriver('192.168.1.100') as plc:
    pass

# Especificar slot (per exemple, slot 2)
with LogixDriver('192.168.1.100/2') as plc:
    pass

# Ruta completa per a configuracions més complexes
with LogixDriver('192.168.1.100', path='1/0/2/192.168.2.100/0') as plc:
    pass
```

### Gestió d'errors

```python
from pycomm3 import LogixDriver
from pycomm3.exceptions import CommError, ResponseError

try:
    with LogixDriver('192.168.1.100') as plc:
        result = plc.read('TagQueNoExisteix')

        if result.error:
            print(f"Error llegint tag: {result.error}")
        else:
            print(f"Valor: {result.value}")

except CommError as e:
    print(f"Error de comunicació: {e}")
except ResponseError as e:
    print(f"Error de resposta del PLC: {e}")
```

---

## Python: libplctag

Llibreria més genèrica que suporta múltiples fabricants.

### Instal·lació

```bash
pip install libplctag
```

### Exemple bàsic

```python
from libplctag import Tag

# Configuració del tag
tag_path = "protocol=ab-eip&gateway=192.168.1.100&path=1,0&cpu=LGX&name=MyTag&elem_type=DINT"

# Crear tag
tag = Tag()
tag.create(tag_path)

# Llegir
tag.read()
value = tag.get_int32(0)
print(f"Valor: {value}")

# Escriure
tag.set_int32(0, 5678)
tag.write()

# Alliberar recursos
tag.destroy()
```

### Tipus de CPU suportats

| Paràmetre cpu | Descripció |
|---------------|------------|
| `LGX` | ControlLogix, CompactLogix |
| `PLC5` | PLC-5 |
| `SLC` | SLC 500 |
| `MLGX` | MicroLogix |

### Exemple per Omron

```python
from libplctag import Tag

# Omron NJ/NX via EtherNet/IP
tag_path = "protocol=ab-eip&gateway=192.168.1.100&path=18,10.10.10.1,1,0&cpu=omron-njnx&name=MyVar"

tag = Tag()
tag.create(tag_path)
tag.read()
value = tag.get_float32(0)
print(f"Valor: {value}")
tag.destroy()
```

---

## Node.js: ethernet-ip

### Instal·lació

```bash
npm install ethernet-ip
```

### Exemple bàsic

```javascript
const { Controller, Tag } = require('ethernet-ip');

const plc = new Controller();

// Connexió
plc.connect('192.168.1.100', 0).then(() => {
    console.log('Connectat al PLC');
    console.log(`Nom: ${plc.properties.name}`);
    console.log(`Slot: ${plc.properties.slot}`);

    // Crear un tag
    const myTag = new Tag('MyTag');

    // Llegir
    plc.readTag(myTag).then(() => {
        console.log(`Valor: ${myTag.value}`);
        console.log(`Tipus: ${myTag.type}`);
    });

}).catch(err => {
    console.error('Error de connexió:', err);
});
```

### Llegir i escriure

```javascript
const { Controller, Tag } = require('ethernet-ip');

async function main() {
    const plc = new Controller();

    try {
        await plc.connect('192.168.1.100', 0);

        // Llegir
        const tempTag = new Tag('Temperature');
        await plc.readTag(tempTag);
        console.log(`Temperatura actual: ${tempTag.value}`);

        // Escriure
        const setpointTag = new Tag('Setpoint');
        setpointTag.value = 75.5;
        await plc.writeTag(setpointTag);
        console.log('Setpoint actualitzat');

    } catch (err) {
        console.error('Error:', err);
    } finally {
        plc.disconnect();
    }
}

main();
```

### Grups de tags

```javascript
const { Controller, Tag, TagGroup } = require('ethernet-ip');

async function main() {
    const plc = new Controller();
    await plc.connect('192.168.1.100', 0);

    // Crear grup de tags per lectura/escriptura eficient
    const group = new TagGroup();
    group.add(new Tag('Temperature'));
    group.add(new Tag('Pressure'));
    group.add(new Tag('FlowRate'));

    // Llegir tots els tags del grup (una sola petició)
    await plc.readTagGroup(group);

    group.forEach(tag => {
        console.log(`${tag.name}: ${tag.value}`);
    });

    // Modificar i escriure
    group.forEach(tag => tag.value = 0);
    await plc.writeTagGroup(group);

    plc.disconnect();
}

main();
```

### Subscripció a canvis (polling)

```javascript
const { Controller, Tag } = require('ethernet-ip');

const plc = new Controller();

plc.connect('192.168.1.100', 0).then(() => {
    // Afegir tags a monitoritzar
    plc.subscribe(new Tag('Temperature'));
    plc.subscribe(new Tag('Pressure'));
    plc.subscribe(new Tag('AlarmStatus'));

    // Configurar interval de polling (ms)
    plc.scan_rate = 500;

    // Event quan un tag canvia de valor
    plc.on('TagChanged', (tag, prevValue) => {
        console.log(`${tag.name}: ${prevValue} -> ${tag.value}`);
    });

    // Event d'error
    plc.on('error', (err) => {
        console.error('Error de comunicació:', err);
    });
});

// Per aturar el polling
// plc.scan_rate = 0;
```

### TypeScript

```typescript
import { Controller, Tag, TagGroup } from 'ethernet-ip';

interface ProcessData {
    temperature: number;
    pressure: number;
    running: boolean;
}

async function readProcessData(): Promise<ProcessData> {
    const plc = new Controller();
    await plc.connect('192.168.1.100', 0);

    const tempTag = new Tag('Temperature');
    const pressTag = new Tag('Pressure');
    const runTag = new Tag('Running');

    await plc.readTag(tempTag);
    await plc.readTag(pressTag);
    await plc.readTag(runTag);

    plc.disconnect();

    return {
        temperature: tempTag.value as number,
        pressure: pressTag.value as number,
        running: runTag.value as boolean
    };
}
```

---

## Node-RED

Per a Node-RED existeix el paquet `node-red-contrib-cip-ethernet-ip`.

### Instal·lació

```bash
cd ~/.node-red
npm install node-red-contrib-cip-ethernet-ip
```

### Nodes disponibles

- **eth-ip-endpoint**: Configuració de connexió al PLC
- **eth-ip-read**: Llegir tags
- **eth-ip-write**: Escriure tags

---

## Comparativa de llibreries

| Característica | pycomm3 | libplctag | ethernet-ip |
|----------------|---------|-----------|-------------|
| Llenguatge | Python | Multi | Node.js |
| Facilitat d'ús | Alta | Mitjana | Alta |
| Suport UDT | Sí | Parcial | Bàsic |
| Multi-fabricant | No (AB només) | Sí | No (AB només) |
| Async/await | No | No | Sí |
| Documentació | Excel·lent | Bona | Bona |
| Manteniment | Actiu | Actiu | Actiu |

---

## Recursos

- [pycomm3 GitHub](https://github.com/ottowayi/pycomm3)
- [pycomm3 Documentació](https://docs.pycomm3.dev/)
- [libplctag GitHub](https://github.com/libplctag/libplctag)
- [ethernet-ip npm](https://www.npmjs.com/package/ethernet-ip)
- [ODVA - EtherNet/IP](https://www.odva.org/)
