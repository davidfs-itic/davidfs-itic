# Cerca i filtrat en viu amb LazyColumn

Un patró molt habitual en llistes és filtrar els elements a mesura que l'usuari escriu en un camp de cerca. Aquest document parteix de l'estructura bàsica de [LazyColumn](./lazycolumn.md), amb la `data class Item` i el composable `ItemCard` ja definits allà, i n'és l'equivalent en Compose de [Recyclerview filtrat](../recyclerviewfilter.md) i [RecyclerView amb searchview](../recyclersearchview.md).

Documentació oficial: https://developer.android.com/develop/ui/compose/lists

## 1. Estat de la cerca i filtratge amb `remember`

El text de cerca es guarda com un estat normal (`remember { mutableStateOf("") }`, vegeu [Estats en Jetpack Compose](./estats.md)), i la llista filtrada es recalcula amb `remember` afegint el text de cerca com a clau:

```kotlin
var query by remember { mutableStateOf("") }

val itemsFiltrats = remember(query, items) {
    items.filter { it.nom.contains(query, ignoreCase = true) }
}
```

`remember(query, items) { ... }` recalcula la llista filtrada només quan `query` o `items` canvien, evitant refer el filtratge a cada recomposició no relacionada amb la cerca.

## 2. Per què no `derivedStateOf`

Podria semblar que aquest és un cas per a `derivedStateOf` (vegeu [derivedStateOf](./derivedstateof.md)), ja que es calcula un estat a partir d'un altre. Però, segons el criteri establert a [derivedStateOf §4](./derivedstateof.md#4-derivedstateof-vs-calcular-ho-directament), `derivedStateOf` només aporta benefici quan l'estat d'origen canvia **més sovint** que el resultat calculat. Aquí passa el contrari: cada tecla que prem l'usuari sol canviar el conjunt d'elements que coincideixen amb la cerca, de manera que el resultat filtrat canvia aproximadament tan sovint com `query`. En aquest cas, `remember` amb clau és suficient i més senzill.

## 3. Mostrar un missatge quan no hi ha resultats

Quan la llista filtrada queda buida (cap element coincideix amb la cerca), convé mostrar un missatge en lloc de deixar la pantalla buida sense cap explicació:

```kotlin
if (itemsFiltrats.isEmpty()) {
    Text(
        text = "Cap element coincideix amb la cerca",
        modifier = Modifier.padding(16.dp)
    )
} else {
    LazyColumn {
        items(itemsFiltrats, key = { it.id }) { item ->
            ItemCard(item = item)
        }
    }
}
```

La `key = { it.id }` es manté igual que en un `LazyColumn` normal (vegeu [LazyColumn §5](./lazycolumn.md#5-key-identificar-els-elements-de-forma-estable)), perquè Compose continuï identificant correctament cada element encara que la llista visible canviï de mida en filtrar.

## 4. Exemple complet

```kotlin
@Composable
fun LlistaAmbCerca(items: List<Item>) {
    var query by remember { mutableStateOf("") }

    val itemsFiltrats = remember(query, items) {
        items.filter { it.nom.contains(query, ignoreCase = true) }
    }

    Column(modifier = Modifier.fillMaxSize()) {
        TextField(
            value = query,
            onValueChange = { query = it },
            label = { Text("Cercar") },
            modifier = Modifier
                .fillMaxWidth()
                .padding(16.dp)
        )

        if (itemsFiltrats.isEmpty()) {
            Text(
                text = "Cap element coincideix amb la cerca",
                modifier = Modifier.padding(16.dp)
            )
        } else {
            LazyColumn(
                contentPadding = PaddingValues(16.dp),
                verticalArrangement = Arrangement.spacedBy(8.dp)
            ) {
                items(itemsFiltrats, key = { it.id }) { item ->
                    ItemCard(item = item)
                }
            }
        }
    }
}
```

Aquest exemple combina el camp de cerca, el filtratge amb `remember(query, items)` i el missatge alternatiu quan no hi ha resultats.
