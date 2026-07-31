# Swipe to dismiss amb LazyColumn

Per eliminar un element d'una llista lliscant-lo horitzontalment, Material 3 ofereix `SwipeToDismissBox`. Aquest document parteix de l'estructura bàsica de [LazyColumn](./lazycolumn.md), amb la `data class Item` i el composable `ItemCard` ja definits allà, i n'és l'equivalent en Compose de [RecyclerView SwipetoDelete](../recyclerviewswipetodelete.md).

Documentació oficial: https://developer.android.com/develop/ui/compose/lists

## 1. `SwipeToDismissBox` i el seu estat

`SwipeToDismissBox` necessita un estat propi, `SwipeToDismissBoxState`, creat amb `rememberSwipeToDismissBoxState`, que porta el control del gest (posició, direcció, si s'ha completat):

```kotlin
val dismissState = rememberSwipeToDismissBoxState()

SwipeToDismissBox(
    state = dismissState,
    backgroundContent = { /* contingut darrere de l'element */ }
) {
    ItemCard(item = item)
}
```

## 2. `confirmValueChange`: acceptar o rebutjar el gest

`rememberSwipeToDismissBoxState` accepta un paràmetre `confirmValueChange`, cridat abans que el gest es completi, que ha de retornar `true` si s'accepta el canvi o `false` si es rebutja (i l'element torna a la seva posició original):

```kotlin
val dismissState = rememberSwipeToDismissBoxState(
    confirmValueChange = { value ->
        if (value == SwipeToDismissBoxValue.EndToStart) {
            onDismiss(item)
            true
        } else {
            false
        }
    }
)
```

Aquí només es confirma quan el lliscament és `EndToStart` (de dreta a esquerra); qualsevol altra direcció es rebutja, de manera que l'usuari només pot eliminar l'element lliscant en aquest sentit concret.

## 3. `backgroundContent`

El paràmetre `backgroundContent` defineix el que es veu darrere de l'element mentre es llisca, normalment una icona d'eliminar sobre un fons de color d'avís:

```kotlin
backgroundContent = {
    Box(
        modifier = Modifier
            .fillMaxSize()
            .background(Color.Red)
            .padding(horizontal = 20.dp),
        contentAlignment = Alignment.CenterEnd
    ) {
        Icon(
            imageVector = Icons.Default.Delete,
            contentDescription = "Eliminar",
            tint = Color.White
        )
    }
}
```

## 4. Ús amb LazyColumn: llista mutable i `key`

Des de `LazyColumn`, la llista ha de ser mutable perquè `onDismiss` la pugui actualitzar. La `key = { it.id }` (vegeu la nota a [LazyColumn §6](./lazycolumn.md#6-itemsindexed-quan-els-items-no-tenen-id)) és imprescindible perquè l'animació d'eliminació funcioni correctament sobre l'element que realment ha desaparegut, en lloc d'aplicar-se per posició:

```kotlin
var items by remember { mutableStateOf(itemsInicials) }

LazyColumn {
    items(items, key = { it.id }) { item ->
        ItemSwipeToDismiss(
            item = item,
            onDismiss = { eliminat -> items = items - eliminat }
        )
    }
}
```

## 5. Exemple complet

```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun ItemSwipeToDismiss(item: Item, onDismiss: (Item) -> Unit) {
    val dismissState = rememberSwipeToDismissBoxState(
        confirmValueChange = { value ->
            if (value == SwipeToDismissBoxValue.EndToStart) {
                onDismiss(item)
                true
            } else {
                false
            }
        }
    )

    SwipeToDismissBox(
        state = dismissState,
        backgroundContent = {
            Box(
                modifier = Modifier
                    .fillMaxSize()
                    .background(Color.Red)
                    .padding(horizontal = 20.dp),
                contentAlignment = Alignment.CenterEnd
            ) {
                Icon(
                    imageVector = Icons.Default.Delete,
                    contentDescription = "Eliminar",
                    tint = Color.White
                )
            }
        }
    ) {
        ItemCard(item = item)
    }
}

@Composable
fun LlistaAmbSwipeToDismiss(itemsInicials: List<Item>) {
    var items by remember { mutableStateOf(itemsInicials) }

    LazyColumn(
        contentPadding = PaddingValues(16.dp),
        verticalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        items(items, key = { it.id }) { item ->
            ItemSwipeToDismiss(
                item = item,
                onDismiss = { eliminat -> items = items - eliminat }
            )
        }
    }
}
```

Aquest exemple combina l'estat del gest (`dismissState`), la confirmació selectiva per direcció (`confirmValueChange`), el fons visual (`backgroundContent`) i la llista mutable amb `key` estable que fa possible l'animació d'eliminació.
