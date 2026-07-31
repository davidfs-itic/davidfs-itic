# Botó "Tornar a dalt" en un LazyColumn

En llistes llargues és habitual mostrar un botó flotant per tornar a l'inici, però només quan l'usuari ja ha baixat prou perquè tingui sentit: no té sentit mostrar-lo si ja s'és a dalt de tot. Aquest document parteix de l'estructura bàsica de [LazyColumn](./lazycolumn.md), amb la `data class Item` i el composable `ItemCard` ja definits allà.

Documentació oficial: https://developer.android.com/develop/ui/compose/lists

## 1. Concepte: saber si cal mostrar el botó

Aquest patró combina l'estat de scroll de `LazyColumn` (`LazyListState`) amb `derivedStateOf`, ja explicat en detall a [derivedStateOf](./derivedstateof.md) (secció 2, on es desenvolupa exactament aquest exemple): la posició de scroll canvia contínuament, però el resultat que interessa —mostrar el botó o no— només canvia dues vegades (quan es passa de 0 a més d'1, i a l'inrevés).

```kotlin
val listState = rememberLazyListState()

val mostrarBoto by remember {
    derivedStateOf { listState.firstVisibleItemIndex > 0 }
}
```

`state = listState` s'ha de passar explícitament a `LazyColumn` perquè es pugui llegir (`firstVisibleItemIndex`) i controlar (`animateScrollToItem`) des de fora del propi `LazyColumn`.

## 2. Desplaçar-se cap amunt: `animateScrollToItem`

Per tornar a l'inici de la llista amb una animació de scroll, `LazyListState` ofereix `animateScrollToItem(0)`. Com que és una funció `suspend`, cal llançar-la des d'una coroutine amb `rememberCoroutineScope`, el mateix patró ja vist a [Scaffold §5](./scaffold.md#5-snackbarhost):

```kotlin
val scope = rememberCoroutineScope()

FloatingActionButton(
    onClick = { scope.launch { listState.animateScrollToItem(0) } }
) {
    Icon(Icons.Default.ArrowUpward, contentDescription = "Tornar a dalt")
}
```

## 3. Mostrar i amagar el botó amb `AnimatedVisibility`

Per evitar que el botó aparegui o desaparegui de cop, s'embolcalla amb `AnimatedVisibility`, controlat pel booleà `mostrarBoto` calculat al punt 1:

```kotlin
AnimatedVisibility(visible = mostrarBoto) {
    FloatingActionButton(
        onClick = { scope.launch { listState.animateScrollToItem(0) } }
    ) {
        Icon(Icons.Default.ArrowUpward, contentDescription = "Tornar a dalt")
    }
}
```

## 4. Exemple complet

El botó se superposa a la llista amb un `Box`, alineat a la cantonada inferior dreta:

```kotlin
@Composable
fun LlistaAmbTornarADalt(items: List<Item>) {
    val listState = rememberLazyListState()
    val scope = rememberCoroutineScope()

    val mostrarBoto by remember {
        derivedStateOf { listState.firstVisibleItemIndex > 0 }
    }

    Box(modifier = Modifier.fillMaxSize()) {
        LazyColumn(
            state = listState,
            contentPadding = PaddingValues(16.dp),
            verticalArrangement = Arrangement.spacedBy(8.dp)
        ) {
            items(items, key = { it.id }) { item ->
                ItemCard(item = item)
            }
        }

        AnimatedVisibility(
            visible = mostrarBoto,
            modifier = Modifier
                .align(Alignment.BottomEnd)
                .padding(16.dp)
        ) {
            FloatingActionButton(
                onClick = { scope.launch { listState.animateScrollToItem(0) } }
            ) {
                Icon(Icons.Default.ArrowUpward, contentDescription = "Tornar a dalt")
            }
        }
    }
}
```

Aquest exemple combina els tres punts anteriors: `listState` compartit entre el `LazyColumn` i el botó, `derivedStateOf` per decidir quan mostrar-lo sense provocar recomposicions innecessàries, i `animateScrollToItem` per l'animació de tornada a dalt.
