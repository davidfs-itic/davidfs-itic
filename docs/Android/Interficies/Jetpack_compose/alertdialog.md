# El composable AlertDialog

`AlertDialog` mostra una finestra emergent que interromp el flux de la pantalla per demanar una confirmació a l'usuari o informar-lo d'alguna cosa important (per exemple, confirmar l'eliminació d'un element, o avisar d'un error). Equival al `AlertDialog.Builder` del sistema de Views, però es defineix de forma declarativa com qualsevol altre composable.

Documentació oficial: https://developer.android.com/develop/ui/compose/components/dialog

## 1. Concepte: un diàleg necessita un estat de visibilitat

Igual que el `DropdownMenu` (vegeu [DropdownMenu](./dropdownmenu.md)), `AlertDialog` no és un component que estigui sempre present a l'arbre de composició: **només es crida quan s'ha de mostrar**. Per tant, cal una variable d'estat booleana que controli si el diàleg és visible, seguint el mateix mecanisme de [Estats en Jetpack Compose](./estats.md):

```kotlin
var mostrarDialeg by remember { mutableStateOf(false) }

Button(onClick = { mostrarDialeg = true }) {
    Text("Eliminar element")
}

if (mostrarDialeg) {
    AlertDialog(
        onDismissRequest = { mostrarDialeg = false },
        title = { Text("Eliminar element") },
        text = { Text("Aquesta acció no es pot desfer. Vols continuar?") },
        confirmButton = {
            TextButton(onClick = { mostrarDialeg = false }) {
                Text("Eliminar")
            }
        },
        dismissButton = {
            TextButton(onClick = { mostrarDialeg = false }) {
                Text("Cancel·lar")
            }
        }
    )
}
```

El `if (mostrarDialeg)` que embolcalla la crida a `AlertDialog` és imprescindible: mentre `mostrarDialeg` sigui `false`, el diàleg simplement no s'inclou a la composició, i per tant no es dibuixa.

## 2. `onDismissRequest`

`onDismissRequest` es crida quan l'usuari intenta tancar el diàleg sense prémer cap dels botons: tocant fora del diàleg o prement el botó d'enrere. Cal actualitzar l'estat a `false` dins d'aquest callback:

```kotlin
AlertDialog(
    onDismissRequest = { mostrarDialeg = false },
    // ...
)
```

Si s'oblida aquest pas, el diàleg quedarà obert indefinidament tocant fora, ja que `AlertDialog` mai actualitza l'estat per si sol: només notifica la intenció de tancar-se, i és responsabilitat del codi que el crida decidir què fer-ne.

## 3. `confirmButton` i `dismissButton`

A diferència de `title` o `text`, els botons no accepten text directament, sinó una lambda `@Composable` amb el botó ja construït (normalment un `TextButton`):

- **`confirmButton`**: obligatori. L'acció principal del diàleg (per exemple, "Eliminar", "Acceptar").
- **`dismissButton`**: opcional. L'acció de cancel·lar. Si no es necessita cap acció secundària (per exemple, en un diàleg purament informatiu), es pot ometre.

```kotlin
AlertDialog(
    onDismissRequest = { mostrarDialeg = false },
    title = { Text("Sessió tancada") },
    text = { Text("La teva sessió ha caducat.") },
    confirmButton = {
        TextButton(onClick = { mostrarDialeg = false }) {
            Text("D'acord")
        }
    }
)
```

Cada botó ha de tancar el diàleg explícitament (`mostrarDialeg = false`) dins del seu propi `onClick`, a banda de fer l'acció corresponent (eliminar, confirmar, etc.), ja que `AlertDialog` no ho fa automàticament.

## 4. `icon`

De manera opcional, `AlertDialog` accepta un paràmetre `icon` per mostrar una icona centrada a sobre del títol, útil per reforçar visualment el tipus d'avís (per exemple, una icona d'alerta en accions destructives):

```kotlin
AlertDialog(
    onDismissRequest = { mostrarDialeg = false },
    icon = {
        Icon(Icons.Default.Warning, contentDescription = null)
    },
    title = { Text("Eliminar element") },
    text = { Text("Aquesta acció no es pot desfer.") },
    confirmButton = {
        TextButton(onClick = { mostrarDialeg = false }) {
            Text("Eliminar")
        }
    },
    dismissButton = {
        TextButton(onClick = { mostrarDialeg = false }) {
            Text("Cancel·lar")
        }
    }
)
```

## 5. Contingut personalitzat: `Dialog`

`AlertDialog` està pensat per al patró clàssic títol + text + botons. Quan es necessita un contingut totalment lliure dins del diàleg (un formulari, una llista, una imatge...), es pot fer servir directament el composable `Dialog`, que només s'encarrega de mostrar la finestra flotant per sobre de la resta de la pantalla, sense imposar cap estructura interna:

```kotlin
Dialog(onDismissRequest = { mostrarDialeg = false }) {
    Card {
        Column(modifier = Modifier.padding(16.dp)) {
            Text("Contingut totalment personalitzat")
            // qualsevol altre composable: TextField, LazyColumn, Image...
        }
    }
}
```

Amb `Dialog` cal construir manualment l'aspecte visual (per exemple, amb un `Card` o `Surface` que li doni fons i elevació), ja que a diferència d'`AlertDialog` no aplica cap estil de Material per defecte.

## 6. Exemple complet

```kotlin
@Composable
fun PantallaAmbConfirmacio() {
    var mostrarDialeg by remember { mutableStateOf(false) }
    var elementEliminat by remember { mutableStateOf(false) }

    Button(onClick = { mostrarDialeg = true }) {
        Text("Eliminar element")
    }

    if (elementEliminat) {
        Text("L'element s'ha eliminat.")
    }

    if (mostrarDialeg) {
        AlertDialog(
            onDismissRequest = { mostrarDialeg = false },
            icon = { Icon(Icons.Default.Warning, contentDescription = null) },
            title = { Text("Eliminar element") },
            text = { Text("Aquesta acció no es pot desfer. Vols continuar?") },
            confirmButton = {
                TextButton(onClick = {
                    elementEliminat = true
                    mostrarDialeg = false
                }) {
                    Text("Eliminar")
                }
            },
            dismissButton = {
                TextButton(onClick = { mostrarDialeg = false }) {
                    Text("Cancel·lar")
                }
            }
        )
    }
}
```

Aquest exemple combina l'estat de visibilitat del diàleg, la icona d'avís, i les dues accions (`confirmButton`/`dismissButton`), cadascuna tancant el diàleg i actualitzant l'estat de la pantalla segons correspongui.
