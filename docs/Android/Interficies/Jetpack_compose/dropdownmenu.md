# El composable DropdownMenu

`DropdownMenu` mostra una llista d'opcions flotant, ancorada a un altre composable, que apareix i desapareix segons l'acció de l'usuari (per exemple, en tocar una icona de menú o un camp de selecció). Equival al `PopupMenu` o al `Spinner` del sistema de Views, segons el cas d'ús.

Documentació oficial: https://developer.android.com/develop/ui/compose/components/menu

## 1. Concepte: un menú necessita un estat `expanded`

A diferència d'altres components vistos fins ara, `DropdownMenu` no representa una dada (com un text o un número), sinó la **visibilitat** d'un menú. Per això sempre va acompanyat d'una variable d'estat booleana (`expanded`) que indica si el menú s'ha de mostrar o no, seguint el mateix mecanisme de [Estats en Jetpack Compose](./estats.md).

`DropdownMenu` s'ha de col·locar dins d'un `Box`, que actua com a àncora: el menú es mostra "penjant" de la posició d'aquest `Box`, independentment d'on estigui a la pantalla.

```kotlin
var expandit by remember { mutableStateOf(false) }

Box {
    IconButton(onClick = { expandit = true }) {
        Icon(Icons.Default.MoreVert, contentDescription = "Més opcions")
    }

    DropdownMenu(
        expanded = expandit,
        onDismissRequest = { expandit = false }
    ) {
        DropdownMenuItem(
            text = { Text("Editar") },
            onClick = {
                expandit = false
                // acció d'editar
            }
        )
        DropdownMenuItem(
            text = { Text("Eliminar") },
            onClick = {
                expandit = false
                // acció d'eliminar
            }
        )
    }
}
```

Punts clau d'aquest patró:

- **`expanded`**: controla si el menú es veu o no. El `DropdownMenu` en si no es dibuixa (ocupa mida zero) mentre `expanded` és `false`.
- **`onDismissRequest`**: es crida quan l'usuari toca fora del menú, o prem enrere, indicant que el menú s'hauria de tancar. Cal actualitzar `expanded` a `false` dins d'aquest callback; si no es fa, el menú no es podria tancar tocant fora.
- **`DropdownMenuItem`**: cada opció del menú. El seu `onClick` ha de fer dues coses: executar l'acció corresponent, i tancar el menú (`expandit = false`), ja que seleccionar una opció no el tanca automàticament.

## 2. Icones a les opcions del menú

Igual que amb `Button`, `DropdownMenuItem` accepta un paràmetre opcional `leadingIcon` per afegir una icona a l'esquerra del text de l'opció:

```kotlin
DropdownMenuItem(
    text = { Text("Eliminar") },
    onClick = { expandit = false },
    leadingIcon = {
        Icon(Icons.Default.Delete, contentDescription = null)
    }
)
```

## 3. Menú desplegable de selecció: `ExposedDropdownMenuBox`

Quan el `DropdownMenu` no serveix per llançar accions, sinó per **triar una opció d'entre una llista** (equivalent a l'`Spinner` del sistema de Views), Compose ofereix un component combinat, `ExposedDropdownMenuBox`, que uneix un `TextField` de només lectura amb el menú desplegable:

```kotlin
val opcions = listOf("Català", "Castellà", "Anglès")
var expandit by remember { mutableStateOf(false) }
var opcioSeleccionada by remember { mutableStateOf(opcions[0]) }

ExposedDropdownMenuBox(
    expanded = expandit,
    onExpandedChange = { expandit = it }
) {
    TextField(
        value = opcioSeleccionada,
        onValueChange = {},
        readOnly = true,
        label = { Text("Idioma") },
        trailingIcon = { ExposedDropdownMenuDefaults.TrailingIcon(expanded = expandit) },
        modifier = Modifier.menuAnchor()
    )

    ExposedDropdownMenu(
        expanded = expandit,
        onDismissRequest = { expandit = false }
    ) {
        opcions.forEach { opcio ->
            DropdownMenuItem(
                text = { Text(opcio) },
                onClick = {
                    opcioSeleccionada = opcio
                    expandit = false
                }
            )
        }
    }
}
```

- **`readOnly = true`** al `TextField`: impedeix que l'usuari escrigui text lliurement; només pot triar una de les opcions del menú.
- **`Modifier.menuAnchor()`**: imprescindible sobre el `TextField` perquè `ExposedDropdownMenu` sàpiga a sota de quin component s'ha de desplegar.
- **`ExposedDropdownMenuDefaults.TrailingIcon`**: proporciona la fletxa desplegable estàndard de Material Design, que ja gira automàticament segons l'estat `expanded`.

Aquest patró és el que s'hauria d'utilitzar sempre que es necessiti un selector d'opcions tipus "Spinner" en Compose.
