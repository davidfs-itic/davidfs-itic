# El composable Dialog

`Dialog` és el composable base de Compose per mostrar contingut flotant per sobre de la resta de la pantalla. A diferència d'`AlertDialog`, no imposa cap estructura (títol, text, botons): només s'encarrega de fer flotar el seu contingut i de gestionar el tancament, deixant totalment lliure el disseny intern.

Documentació oficial: https://developer.android.com/develop/ui/compose/components/dialog

## 1. Concepte: la base sobre la qual es construeix `AlertDialog`

`AlertDialog` (vegeu [AlertDialog](./alertdialog.md)) ja permet resoldre la majoria de casos: un títol, un text i uns botons d'acció. Però quan el contingut del diàleg no encaixa en aquest patró (un formulari amb diversos camps, una llista, una imatge...), cal recórrer al composable més genèric, `Dialog`, que ja s'introduïa breument a [AlertDialog §5](./alertdialog.md#5-contingut-personalitzat-dialog):

```kotlin
Dialog(onDismissRequest = { /* tancar */ }) {
    // qualsevol contingut, sense cap estructura predefinida
}
```

## 2. Estat de visibilitat i `onDismissRequest`

Com qualsevol diàleg, `Dialog` només s'ha d'incloure a la composició quan toca mostrar-se, controlat per una variable d'estat booleana (vegeu [Estats en Jetpack Compose](./estats.md)):

```kotlin
var mostrarDialeg by remember { mutableStateOf(false) }

if (mostrarDialeg) {
    Dialog(onDismissRequest = { mostrarDialeg = false }) {
        // contingut
    }
}
```

`onDismissRequest` es crida quan l'usuari toca fora del diàleg o prem enrere, exactament igual que a `AlertDialog`: és responsabilitat del codi que el crida actualitzar l'estat a `false` dins d'aquest callback.

## 3. Donar aspecte visual al contingut

`Dialog` no aplica cap fons, ni cantonades arrodonides, ni elevació: només posiciona el contingut flotant al centre de la pantalla. Sense res més, el contingut es veuria "pla", sense distingir-se del fons:

```kotlin
// Sense Surface/Card: el text flota sense cap fons, es veu estrany
Dialog(onDismissRequest = { mostrarDialeg = false }) {
    Text("Contingut sense estil")
}
```

Per això, gairebé sempre cal embolcallar el contingut amb un `Surface` o un `Card`, que sí que proporcionen fons, forma i elevació:

```kotlin
Dialog(onDismissRequest = { mostrarDialeg = false }) {
    Surface(
        shape = RoundedCornerShape(16.dp),
        tonalElevation = 4.dp
    ) {
        Column(modifier = Modifier.padding(24.dp)) {
            Text("Contingut amb aspecte de diàleg")
        }
    }
}
```

- **`shape`**: la forma del contenidor; `RoundedCornerShape` és l'habitual per a diàlegs.
- **`tonalElevation`** (a `Surface`) o **`elevation`** (a `Card`): dona la sensació de profunditat pròpia de Material Design, distingint el diàleg del fons de la pantalla.

## 4. `DialogProperties`

El paràmetre `properties` permet ajustar el comportament del diàleg mitjançant `DialogProperties`:

```kotlin
Dialog(
    onDismissRequest = { mostrarDialeg = false },
    properties = DialogProperties(
        dismissOnBackPress = true,
        dismissOnClickOutside = true,
        usePlatformDefaultWidth = false
    )
) {
    // contingut
}
```

- **`dismissOnBackPress`**: si `true` (per defecte), prémer enrere tanca el diàleg.
- **`dismissOnClickOutside`**: si `true` (per defecte), tocar fora del diàleg el tanca.
- **`usePlatformDefaultWidth`**: per defecte `true`, limita l'amplada del diàleg a l'amidada estàndard de Material. Posar-lo a `false` permet que el contingut ocupi tota l'amplada disponible (per exemple, amb `Modifier.fillMaxWidth()` dins del `Surface`), útil quan el contingut és ample, com un formulari llarg o una taula.

## 5. Exemple pràctic: editar un `Item`

Un cas d'ús habitual és un diàleg de formulari per editar un objecte de dades. Es defineix una `data class` senzilla:

```kotlin
data class Item(
    val titol: String = "",
    val descripcio: String = ""
)
```

I un composable `MyItemDialog` que rep l'`item` a editar i, en lloc de modificar-lo directament, **proposa** un nou valor a través d'un callback quan l'usuari confirma:

```kotlin
@Composable
fun MyItemDialog(
    item: Item,
    onDismissRequest: () -> Unit,
    onConfirm: (Item) -> Unit
) {
    var titol by remember { mutableStateOf(item.titol) }
    var descripcio by remember { mutableStateOf(item.descripcio) }

    Dialog(onDismissRequest = onDismissRequest) {
        Surface(
            shape = RoundedCornerShape(16.dp),
            tonalElevation = 4.dp
        ) {
            Column(modifier = Modifier.padding(24.dp)) {
                Text("Editar element", style = MaterialTheme.typography.titleLarge)
                Spacer(modifier = Modifier.height(16.dp))

                TextField(
                    value = titol,
                    onValueChange = { titol = it },
                    label = { Text("Títol") }
                )
                Spacer(modifier = Modifier.height(8.dp))
                TextField(
                    value = descripcio,
                    onValueChange = { descripcio = it },
                    label = { Text("Descripció") }
                )

                Spacer(modifier = Modifier.height(16.dp))
                Row(
                    modifier = Modifier.fillMaxWidth(),
                    horizontalArrangement = Arrangement.End
                ) {
                    TextButton(onClick = onDismissRequest) {
                        Text("Cancel·lar")
                    }
                    TextButton(onClick = { onConfirm(Item(titol, descripcio)) }) {
                        Text("Desar")
                    }
                }
            }
        }
    }
}
```

Punts clau d'aquest disseny:

- **`item` és només el valor inicial**: `titol` i `descripcio` són còpies locals editables (`remember { mutableStateOf(item.titol) }`), seedejades a partir d'`item` quan el diàleg es crea. Mentre l'usuari escriu, l'`item` original no canvia.
- **`onConfirm` "recupera" el resultat**: en prémer "Desar", es construeix un nou `Item(titol, descripcio)` i es passa a `onConfirm`. El diàleg mai modifica res per si mateix: només notifica quin hauria de ser el nou valor.

Aquest és el mateix principi de *state hoisting* unidireccional vist a [Estats en Jetpack Compose](./estats.md): l'estat "de veritat" viu fora del diàleg, i el diàleg només el llegeix (per inicialitzar-se) i el proposa (per actualitzar-lo).

Des de la pantalla que el crida:

```kotlin
var item by remember { mutableStateOf(Item("Comprar llet", "Anar al supermercat")) }
var mostrarDialeg by remember { mutableStateOf(false) }

Button(onClick = { mostrarDialeg = true }) {
    Text("Editar")
}

if (mostrarDialeg) {
    MyItemDialog(
        item = item,
        onDismissRequest = { mostrarDialeg = false },
        onConfirm = { itemActualitzat ->
            item = itemActualitzat
            mostrarDialeg = false
        }
    )
}
```

Aquí `item` és l'única font de veritat de la pantalla: es passa al diàleg per mostrar els valors actuals, i s'actualitza únicament quan `onConfirm` retorna un nou valor. Aquest exemple mostra per què calia `Dialog` i no `AlertDialog`: el contingut és un formulari amb el seu propi estat intern, cosa que l'estructura fixa de títol+text+botons d'`AlertDialog` no permet.

## 6. Quan triar `Dialog` en lloc d'`AlertDialog`

- **`AlertDialog`** (o `DatePickerDialog`): quan el contingut encaixa en el patró estàndard de Material (títol, missatge i botons de confirmació/cancel·lació).
- **`Dialog`**: quan cal control total sobre el disseny intern, com a l'exemple de `MyItemDialog`, on el contingut és un formulari amb el seu propi estat, o qualsevol altre cas que no es limiti a mostrar text.
