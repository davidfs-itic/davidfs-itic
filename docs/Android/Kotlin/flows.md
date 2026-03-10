# Flows

Un **Flow** es un tipus de Kotlin que permet emetre una seqüencia de valors de forma asíncrona al llarg del temps. A diferencia d'una funcio `suspend`, que retorna un sol valor, un Flow pot emetre multiples valors de manera reactiva.

Els Flows formen part de la llibreria `kotlinx.coroutines.flow` i estan estretament lligats a les [coroutines](./coroutines.md).

Documentacio oficial: [Kotlin Flows - Android Developers](https://developer.android.com/kotlin/flow)

## 1. Per que serveixen els Flows?

En aplicacions Android, sovint necessitem observar dades que canvien al llarg del temps:

- Preferencies de l'usuari que es modifiquen (per exemple, amb [DataStore](../Llibreries/datastore.md))
- Resultats d'una consulta a la base de dades que s'actualitzen
- Dades en temps real d'una API

Un Flow emet un nou valor cada cop que les dades canvien, i els observadors (collectors) reben automaticament l'actualitzacio.

## 2. Crear un Flow

Un Flow es crea amb el builder `flow { }`. Dins del bloc, s'utilitza `emit()` per enviar valors:

```kotlin
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.flow
import kotlinx.coroutines.delay

fun comptador(): Flow<Int> = flow {
    for (i in 1..5) {
        delay(1000) // espera 1 segon
        emit(i)     // emet el valor
    }
}
```

!!! info "Flows son freds (cold)"
    Un Flow no s'executa fins que algu el **col·lecta** (collect). Cada cop que es crida `collect`, el Flow torna a començar des del principi.

## 3. Rebre valors amb collect

Per rebre els valors d'un Flow s'utilitza la funcio `collect`, que es una funcio **suspend** i per tant necessita executar-se dins d'una coroutine:

```kotlin
import kotlinx.coroutines.launch
import androidx.lifecycle.lifecycleScope

lifecycleScope.launch {
    comptador().collect { valor ->
        println("Valor rebut: $valor")
    }
}
```

Sortida (un valor per segon):

```
Valor rebut: 1
Valor rebut: 2
Valor rebut: 3
Valor rebut: 4
Valor rebut: 5
```

!!! warning "collect es bloquejant"
    La funcio `collect` suspen la coroutine fins que el Flow finalitza. Si necessites col·lectar diversos Flows en paral·lel, cal llançar cada `collect` en una coroutine separada amb `launch`.

## 4. Operadors de transformacio

Els Flows es poden transformar amb operadors com `map`, `filter` o `combine`. Aquests operadors funcionen de forma similar als de les [col·leccions](./colleccions.md) de Kotlin, pero aplicats a fluxos asincrons.

```kotlin
import kotlinx.coroutines.flow.map

fun comptadorDoble(): Flow<Int> = comptador().map { it * 2 }
```

Per a una explicacio detallada d'aquests operadors, consulteu el document de [col·leccions](./colleccions.md).

## 5. Flows vs LiveData

Tant `Flow` com `LiveData` permeten observar canvis en les dades, pero tenen diferencies importants:

| Caracteristica | LiveData | Flow |
|---|---|---|
| Lifecycle-aware | Si, automatic | No directament (cal `lifecycleScope`) |
| Valor inicial | Opcional | No te |
| Operadors de transformacio | Limitats (`map`, `switchMap`) | Molt rics (`map`, `filter`, `combine`, `zip`...) |
| Funciona fora d'Android | No (depèn del framework Android) | Si (es Kotlin pur) |
| Emissio | Nomes al fil principal | Des de qualsevol fil |
| Reactivitat | Un sol valor actiu | Seqüencia de valors al llarg del temps |

### Quan utilitzar cada un?

- **LiveData**: Segueix sent valid per exposar dades del ViewModel a la UI de forma senzilla. Es especialment util quan no cal fer transformacions complexes.
- **Flow**: Recomanat per a fonts de dades (repositoris, DataStore, Room) i quan calen transformacions avançades. Al ViewModel, es pot convertir a [StateFlow](./stateflow.md) per exposar-lo a la UI.

### Exemple comparatiu

Amb LiveData:

```kotlin
// ViewModel
private val _nom = MutableLiveData<String>("")
val nom: LiveData<String> = _nom

// Activity
viewModel.nom.observe(this) { valor ->
    binding.tvNom.text = valor
}
```

Amb Flow (convertit a StateFlow):

```kotlin
// ViewModel
private val _nom = MutableStateFlow("")
val nom: StateFlow<String> = _nom

// Activity
lifecycleScope.launch {
    viewModel.nom.collect { valor ->
        binding.tvNom.text = valor
    }
}
```

Per a mes informacio sobre StateFlow, consulteu el document de [StateFlow](./stateflow.md).
