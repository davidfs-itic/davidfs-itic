# Col·leccions i operadors funcionals

Les **col·leccions** son estructures de dades fonamentals en Kotlin que permeten agrupar i manipular conjunts de valors. Kotlin ofereix una API molt rica d'**operadors funcionals** que es poden aplicar tant a llistes i mapes com a [Flows](./flows.md).

Documentacio oficial: [Collections overview - Kotlin](https://kotlinlang.org/docs/collections-overview.html)

## 1. Tipus de col·leccions

Kotlin distingeix entre col·leccions **immutables** (nomes lectura) i **mutables** (lectura i escriptura):

```kotlin
// Immutables (recomanades per defecte)
val noms: List<String> = listOf("Anna", "Marc", "Laia")
val edats: Set<Int> = setOf(20, 22, 20) // sense duplicats: {20, 22}
val mapa: Map<String, Int> = mapOf("Anna" to 20, "Marc" to 22)

// Mutables
val nomsMutables: MutableList<String> = mutableListOf("Anna", "Marc")
nomsMutables.add("Laia")
```

| Tipus | Immutable | Mutable |
|---|---|---|
| Llista ordenada | `List` / `listOf()` | `MutableList` / `mutableListOf()` |
| Conjunt sense duplicats | `Set` / `setOf()` | `MutableSet` / `mutableSetOf()` |
| Parells clau-valor | `Map` / `mapOf()` | `MutableMap` / `mutableMapOf()` |

!!! info "Immutable per defecte"
    Es recomanable utilitzar col·leccions immutables sempre que sigui possible. Aixo evita modificacions accidentals i fa el codi mes segur.

## 2. Operador map

Transforma cada element de la col·leccio aplicant una funcio:

```kotlin
val noms = listOf("anna", "marc", "laia")

val nomsMajuscules = noms.map { it.uppercase() }
// Resultat: ["ANNA", "MARC", "LAIA"]
```

Exemple amb objectes:

```kotlin
data class Alumne(val nom: String, val nota: Double)

val alumnes = listOf(
    Alumne("Anna", 8.5),
    Alumne("Marc", 6.0),
    Alumne("Laia", 9.2)
)

val nomsAlumnes: List<String> = alumnes.map { it.nom }
// Resultat: ["Anna", "Marc", "Laia"]
```

### map en Flows

L'operador `map` tambe funciona amb [Flows](./flows.md), transformant cada valor emes:

```kotlin
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.map

fun temperaturesEnFahrenheit(celsius: Flow<Double>): Flow<Double> {
    return celsius.map { it * 9 / 5 + 32 }
}
```

## 3. Operador filter

Retorna nomes els elements que compleixen una condicio:

```kotlin
val numeros = listOf(1, 2, 3, 4, 5, 6, 7, 8)

val parells = numeros.filter { it % 2 == 0 }
// Resultat: [2, 4, 6, 8]
```

Exemple amb objectes:

```kotlin
val aprovats = alumnes.filter { it.nota >= 5.0 }
val excel·lents = alumnes.filter { it.nota >= 9.0 }
```

### filter en Flows

```kotlin
import kotlinx.coroutines.flow.filter

fun temperaturesAltes(temperatures: Flow<Double>): Flow<Double> {
    return temperatures.filter { it > 30.0 }
}
```

## 4. Encadenar operadors

Un dels punts forts de Kotlin es que els operadors es poden encadenar per fer transformacions complexes de forma llegible:

```kotlin
val alumnes = listOf(
    Alumne("Anna", 8.5),
    Alumne("Marc", 4.0),
    Alumne("Laia", 9.2),
    Alumne("Pol", 6.5)
)

val nomsAprovats = alumnes
    .filter { it.nota >= 5.0 }
    .map { it.nom }
    .sorted()
// Resultat: ["Anna", "Laia", "Pol"]
```

Amb Flows funciona igual:

```kotlin
fun nomsAlumnesAprovats(alumnes: Flow<Alumne>): Flow<String> {
    return alumnes
        .filter { it.nota >= 5.0 }
        .map { it.nom }
}
```

## 5. Altres operadors utils

### forEach

Executa una accio per cada element (no retorna una nova col·leccio):

```kotlin
noms.forEach { nom ->
    println("Hola, $nom!")
}
```

### sortedBy

Ordena els elements segons un criteri:

```kotlin
val ordenatsPerNota = alumnes.sortedBy { it.nota }
val ordenatsDesc = alumnes.sortedByDescending { it.nota }
```

### firstOrNull / find

Retorna el primer element que compleix la condicio, o `null` si no n'hi ha cap:

```kotlin
val primerExcellent = alumnes.firstOrNull { it.nota >= 9.0 }
// Equivalent:
val primerExcellent2 = alumnes.find { it.nota >= 9.0 }
```

### any / all / none

Comproven condicions sobre la col·leccio:

```kotlin
val hiHaSuspesos = alumnes.any { it.nota < 5.0 }    // true
val totsAprovats = alumnes.all { it.nota >= 5.0 }    // false
val capSuspes = alumnes.none { it.nota < 5.0 }       // false
```

### groupBy

Agrupa elements per un criteri:

```kotlin
val perNota = alumnes.groupBy {
    if (it.nota >= 5.0) "Aprovat" else "Suspes"
}
// Resultat: {"Aprovat": [Anna, Laia, Pol], "Suspes": [Marc]}
```

### sumOf

Suma un valor numeric de cada element:

```kotlin
val sumaTotal = alumnes.sumOf { it.nota }
```

## 6. Operador combine (exclusiu de Flows)

L'operador `combine` combina els valors mes recents de dos o mes Flows. Cada cop que un dels Flows emet un nou valor, es recalcula el resultat:

```kotlin
import kotlinx.coroutines.flow.combine

val nom: Flow<String> = // ...
val edat: Flow<Int> = // ...

val perfil: Flow<String> = nom.combine(edat) { n, e ->
    "$n te $e anys"
}
```

Exemple practic amb un ViewModel:

```kotlin
class CercaViewModel : ViewModel() {
    private val _textCerca = MutableStateFlow("")
    private val _totsElsArticles = MutableStateFlow<List<Article>>(emptyList())

    val articlesFiltrats: Flow<List<Article>> = _textCerca
        .combine(_totsElsArticles) { text, articles ->
            if (text.isEmpty()) articles
            else articles.filter { it.nom.contains(text, ignoreCase = true) }
        }
}
```
