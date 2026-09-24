# derivedStateOf

`derivedStateOf` serveix per calcular un estat a partir d'un altre estat, evitant recomposicions innecessàries quan el valor calculat no canvia encara que l'estat d'origen canviï amb molta freqüència. És una eina d'optimització, no una funcionalitat nova: el mateix resultat es podria calcular sense `derivedStateOf`, però amb un cost de recomposició més alt.

Documentació oficial: https://developer.android.com/develop/ui/compose/side-effects#derivedstateof

## 1. El problema: un estat que canvia molt, un resultat que canvia poc

Imaginem una llista amb scroll on es vol mostrar un botó "Tornar a dalt" només quan l'usuari ha baixat més enllà del primer element. La posició de scroll (`firstVisibleItemIndex`) canvia constantment mentre es fa scroll, però el resultat que interessa —si cal mostrar el botó o no— només canvia dues vegades: quan es passa de 0 a més d'1, i a l'inrevés.

Si es calcula el booleà directament dins del composable, cada canvi mínim de la posició de scroll provoca una recomposició de tot allò que depengui d'aquest càlcul, encara que el resultat final (el booleà) no hagi canviat:

```kotlin
val listState = rememberLazyListState()

// Es recalcula, i recomposa el que en depèn, a CADA canvi de scroll
val mostrarBoto = listState.firstVisibleItemIndex > 0
```

## 2. La solució: `remember { derivedStateOf { ... } }`

`derivedStateOf` embolcalla el càlcul en un nou `State`, que només notifica als seus lectors quan el **resultat** canvia, no quan ho fan els valors que utilitza per calcular-lo:

```kotlin
val listState = rememberLazyListState()

val mostrarBoto by remember {
    derivedStateOf { listState.firstVisibleItemIndex > 0 }
}

if (mostrarBoto) {
    FloatingActionButton(onClick = { /* tornar a dalt */ }) {
        Icon(Icons.Default.ArrowUpward, contentDescription = "Tornar a dalt")
    }
}
```

Amb aquest canvi, `mostrarBoto` només provoca una recomposició quan realment passa de `true` a `false` o viceversa, encara que `firstVisibleItemIndex` canviï contínuament mentre es fa scroll.

## 3. Per què cal `remember` a més de `derivedStateOf`

`derivedStateOf { ... }` per si sol crea un nou objecte `State` cada vegada que s'executa; sense `remember`, es tornaria a crear a cada recomposició, perdent tot l'avantatge. Per això sempre s'utilitzen junts:

```kotlin
val mostrarBoto by remember {
    derivedStateOf { listState.firstVisibleItemIndex > 0 }
}
```

`remember` fa que l'objecte `derivedStateOf` es creï una sola vegada i es reutilitzi entre recomposicions; `derivedStateOf` és qui decideix, a cada canvi de l'estat d'origen, si cal notificar-ho o no.

## 4. `derivedStateOf` vs. calcular-ho directament

No sempre cal `derivedStateOf`. Té sentit quan es compleixen totes dues condicions:

- L'estat d'origen canvia **més sovint** que el resultat calculat (com l'exemple del scroll).
- El càlcul es fa servir en un lloc on una recomposició extra té un cost real (llistes llargues, animacions, jerarquies grans de composables).

Si el resultat canvia aproximadament tan sovint com l'estat d'origen (per exemple, mostrar directament `"${nom.length}/30"` a partir d'un `TextField`), `derivedStateOf` no aporta cap benefici: només afegeix una capa d'indirecció innecessària. En aquests casos, calcular el valor directament dins del composable és preferible.

## 5. Exemple complet

```kotlin
@Composable
fun LlistaAmbBotoDePujada(elements: List<String>) {
    val listState = rememberLazyListState()
    val scope = rememberCoroutineScope()

    val mostrarBoto by remember {
        derivedStateOf { listState.firstVisibleItemIndex > 0 }
    }

    Box(modifier = Modifier.fillMaxSize()) {
        LazyColumn(state = listState, modifier = Modifier.fillMaxSize()) {
            items(elements) { element ->
                Text(text = element, modifier = Modifier.padding(16.dp))
            }
        }

        if (mostrarBoto) {
            FloatingActionButton(
                onClick = {
                    scope.launch { listState.animateScrollToItem(0) }
                },
                modifier = Modifier
                    .align(Alignment.BottomEnd)
                    .padding(16.dp)
            ) {
                Icon(Icons.Default.ArrowUpward, contentDescription = "Tornar a dalt")
            }
        }
    }
}
```

Aquí `derivedStateOf` evita que el `FloatingActionButton` es recomposi a cada píxel de scroll, mentre que `animateScrollToItem` (llançat des d'una corrutina amb `rememberCoroutineScope`, com a [Scaffold](./scaffold.md#5-snackbarhost)) s'encarrega de l'animació de tornada a dalt.
