# El composable LazyColumn

`LazyColumn` és l'equivalent en Compose del `RecyclerView` del sistema de Views (vegeu [RecyclerView](../recyclerview.md)): mostra una llista vertical d'elements, però només compon i dibuixa aquells que són visibles a la pantalla en cada moment, en lloc de crear-los tots de cop com faria una `Column` amb scroll.

Documentació oficial: https://developer.android.com/develop/ui/compose/lists

## 1. Per què no una `Column` amb scroll

Amb una `Column` normal i `Modifier.verticalScroll()` (vegeu [Layouts en Jetpack Compose](./layouts.md)), tots els elements de la llista es componen d'entrada, encara que la majoria no siguin visibles. Per a llistes curtes això no és cap problema, però per a llistes llargues (desenes o centenars d'elements) suposa compondre i mantenir en memòria molts més composables dels que calen.

`LazyColumn` resol això amb *lazy loading*: només compon els elements que es veuen a la pantalla (més un petit marge), i els allibera a mesura que desapareixen fent scroll, tal com fa `RecyclerView` reciclant les seves `ViewHolder`.

## 2. Els items com a `data class`

Igual que a `RecyclerView`, cada element de la llista es modela amb una `data class` que representa les dades, independent de com es dibuixarà:

```kotlin
data class Item(
    val id: Int,
    val nom: String,
    val descripcio: String
)
```

## 3. Ús bàsic: `items()`

`LazyColumn` no rep els seus fills directament com a paràmetres (com faria `Column`), sinó dins d'una lambda amb DSL pròpia (`LazyListScope`), on la funció `items()` genera un element composable per cada objecte de la llista:

```kotlin
@Composable
fun LlistaItems(items: List<Item>) {
    LazyColumn {
        items(items) { item ->
            Text(text = item.nom, modifier = Modifier.padding(16.dp))
        }
    }
}
```

Aquest patró (`LazyColumn { items(llista) { element -> ... } }`) és equivalent conceptualment a implementar un `Adapter` i un `ViewHolder` a `RecyclerView`, però sense necessitat de cap classe addicional: la lambda de `items()` fa exactament el paper d'`onBindViewHolder`.

## 4. Element com a composable propi: `ItemCard`

Igual que a `RecyclerView` es defineix un layout per fila, a `LazyColumn` és habitual extreure el disseny de cada element a un composable propi, que rep l'`Item` com a paràmetre:

```kotlin
@Composable
fun ItemCard(item: Item) {
    Card(
        modifier = Modifier
            .fillMaxWidth()
            .padding(horizontal = 16.dp, vertical = 4.dp)
    ) {
        Column(modifier = Modifier.padding(12.dp)) {
            Text(text = item.nom, style = MaterialTheme.typography.titleMedium)
            Text(text = item.descripcio, style = MaterialTheme.typography.bodyMedium)
        }
    }
}
```

```kotlin
LazyColumn {
    items(items) { item ->
        ItemCard(item = item)
    }
}
```

Aquesta separació (dades a `Item`, disseny a `ItemCard`) manté cada element de la llista fàcil de provar i de fer servir en un `@Preview` (vegeu [Introducció a Jetpack Compose](./index.md#3-anotadors-composable-i-preview)) sense necessitat de muntar tota la llista.

## 5. `key`: identificar els elements de forma estable

Per defecte, `LazyColumn` identifica cada element per la seva posició a la llista. Si la llista canvia (s'afegeix, s'elimina o es reordena un element), Compose pot arribar a confondre quins elements són "els mateixos" que abans, provocant recomposicions innecessàries o animacions incorrectes. El paràmetre `key` de `items()` evita aquest problema, indicant explícitament un identificador estable per element:

```kotlin
LazyColumn {
    items(items, key = { item -> item.id }) { item ->
        ItemCard(item = item)
    }
}
```

Amb `key = { item.id }`, Compose sap que un `Item` amb `id = 3` és sempre el mateix element encara que canviï de posició a la llista, cosa especialment important quan la llista es filtra, s'ordena o s'hi afegeixen/eliminen elements dinàmicament.

## 6. `itemsIndexed`: quan els items no tenen `id`

Quan els elements de la llista no disposen de cap camp identificador propi (per exemple, una simple `List<String>`), es pot fer servir `itemsIndexed` en lloc de `items`, que proporciona també la posició de cada element dins la lambda:

```kotlin
LazyColumn {
    itemsIndexed(noms) { index, nom ->
        Text(text = "$index. $nom", modifier = Modifier.padding(16.dp))
    }
}
```

L'`index` és útil per mostrar la posició a la interfície (com en aquest exemple) o per a lògica que depengui de l'ordre (marcar el primer o l'últim element de forma diferent).

`itemsIndexed` també accepta un paràmetre `key`, igual que `items`, però en aquest cas la lambda rep tant l'índex com l'element (`key = { index, item -> ... }`):

```kotlin
LazyColumn {
    itemsIndexed(noms, key = { _, nom -> nom }) { index, nom ->
        Text(text = "$index. $nom", modifier = Modifier.padding(16.dp))
    }
}
```

Cal tenir cura amb què s'utilitza com a `key`: si es fa servir directament l'`index`, no s'aconsegueix cap identificador estable (si la llista es reordena o es filtra, un mateix element pot passar a tenir un índex diferent, i deixaria de ser "el mateix" per a Compose). En canvi, si els elements no tenen `id` però són valors únics per si mateixos (com un `String` que no es repeteix, en l'exemple anterior), es pot fer servir directament l'element com a `key`. Per això, sempre que sigui possible, és preferible que els items tinguin un identificador propi.

!!! info "No posar `key` no és només estètic"
    Sense `key`, Compose identifica cada element per la seva **posició** a la llista: si es reordena o s'insereix un element enmig, Compose no sap que és "el mateix" element mogut de lloc, i el tracta com si el contingut d'aquella posició hagués canviat. Amb `key`, en canvi, Compose segueix la identitat lògica de l'element independentment d'on estigui. Això afecta coses concretes: l'estat intern (`remember`) d'un element pot quedar "enganxat" a la posició en lloc de seguir l'element, les animacions de moviment (`Modifier.animateItem()`) necessiten `key` per funcionar, i inserir un element al mig sense `key` fa que Compose recomposi tots els elements posteriors en lloc de només el nou.

## 7. Separadors i capçaleres

Dins de la lambda de `LazyColumn` es poden combinar diverses crides, no només `items()`. Per exemple, `item()` (en singular) afegeix un únic element, útil per a capçaleres, i `HorizontalDivider` es pot intercalar dins del propi `ItemCard` o entre elements:

```kotlin
LazyColumn {
    item {
        Text(
            text = "Llista d'elements",
            style = MaterialTheme.typography.headlineSmall,
            modifier = Modifier.padding(16.dp)
        )
    }

    items(items, key = { it.id }) { item ->
        ItemCard(item = item)
        HorizontalDivider()
    }
}
```

## 8. Espaiat entre elements

En lloc d'afegir `padding` manualment a cada element per separar-los, `LazyColumn` accepta el paràmetre `verticalArrangement`, que permet indicar un espaiat uniforme entre tots els elements, igual que faria una `Column` (vegeu [Layouts en Jetpack Compose](./layouts.md)):

```kotlin
LazyColumn(
    verticalArrangement = Arrangement.spacedBy(8.dp),
    contentPadding = PaddingValues(16.dp)
) {
    items(items, key = { it.id }) { item ->
        ItemCard(item = item)
    }
}
```

- **`Arrangement.spacedBy(8.dp)`**: afegeix 8dp entre cada parell d'elements consecutius, sense afegir espai abans del primer ni després de l'últim.
- **`contentPadding`**: a diferència d'un `Modifier.padding()` normal, aquest padding s'aplica al contingut desplaçable per dins, de manera que el primer i l'últim element també queden separats de les vores, sense que aquest espai desaparegui en fer scroll.

## 9. Exemple complet

```kotlin
data class Item(
    val id: Int,
    val nom: String,
    val descripcio: String
)

@Composable
fun ItemCard(item: Item) {
    Card(modifier = Modifier.fillMaxWidth()) {
        Column(modifier = Modifier.padding(12.dp)) {
            Text(text = item.nom, style = MaterialTheme.typography.titleMedium)
            Text(text = item.descripcio, style = MaterialTheme.typography.bodyMedium)
        }
    }
}

@Composable
fun LlistaItems(items: List<Item>) {
    LazyColumn(
        modifier = Modifier.fillMaxSize(),
        contentPadding = PaddingValues(16.dp),
        verticalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        item {
            Text(
                text = "Elements (${items.size})",
                style = MaterialTheme.typography.headlineSmall
            )
        }

        items(items, key = { it.id }) { item ->
            ItemCard(item = item)
        }
    }
}
```

Aquest exemple combina una capçalera (`item`), la llista d'elements amb `key` estable (`items`), l'espaiat uniforme (`verticalArrangement`) i el marge del contingut desplaçable (`contentPadding`), formant el patró habitual per mostrar qualsevol llista en Compose.

## 10. Patrons habituals

Sobre aquest mateix esquelet es combinen sovint altres patrons d'interacció, documentats cadascun en un fitxer propi:

- [Botó "Tornar a dalt"](./lazycolumnscrolltop.md): mostrar un botó flotant només quan cal, segons la posició de scroll.
- [Cerca i filtrat en viu](./lazycolumnfilter.md): filtrar la llista a mesura que l'usuari escriu.
- [Swipe to dismiss](./lazycolumnswipetodismiss.md): eliminar un element lliscant-lo.
- Scroll infinit / paginació — detectar quan l'usuari arriba al final de la llista (derivedStateOf sobre LazyListState) per carregar-ne més.
- Estats de càrrega/buit/error — mostrar un CircularProgressIndicator, un missatge "sense elements" o un error segons l'estat, típic quan la llista ve d'un ViewModel.
- Sticky headers — llistes agrupades amb capçalera fixa (stickyHeader dins LazyListScope).
- Pull to refresh — PullToRefreshBox combinat amb LazyColumn.