# Corrutines

Les corrutines són una de les característiques més potents i diferencials del llenguatge Kotlin. Serveixen per gestionar tasques asíncrones i concurrents d’una manera molt més senzilla, segura i llegible que amb fils (threads) tradicionals.

- https://www.youtube.com/watch?v=vQ0w4zAe68A&t=943
- https://developer.android.com/kotlin/coroutines?hl=es-419

## Què és una corrutina?

Una corrutina és una unitat d'execució lleugera que es pot pausar i reprendre sense bloquejar el fil principal del programa.

- Es podrien descriure com “threads lleugers”
- Permeten fer processos costosos sense congelar la UI
- Fan el codi asíncron més intuïtiu i lineal
- Es poden cancelar automàticament

## Per què s’utilitzen?

Quan programem, especialment en aplicacions Android, sovint tenim tasques com:

- Accedir a la base de dades
- Fer peticions HTTP (APIs)
- Llegir fitxers
- Fer càlculs costosos

Si aquestes tasques s'executessin al fil principal → bloquejaria la interfície gràfica.

Les corrutines resolen aquest problema executant-se en un fil secundari, i tornant el resultat quan està llest.


## Elements principals de les corrutines
### 1. CoroutineScope

És el “context” on viuen les corrutines.

Exemples de scopes:

Scope |	Ús
|:----|:----
GlobalScope	|corrutina global durant tota l'app
MainScope	|vinculat a UI
lifecycleScope	|Android – vinculat al cicle de vida
viewModelScope	|Android – s’atura quan el ViewModel mor

### 2. launch vs async

#### launch

Executa una corrutina sense retornar resultat

```kotlin
lifecycleScope.launch {
    println("Això és una tasca asíncrona")
}
```
#### async

Retorna un objecte Deferred<T> i es pot recollir el resultat amb await()

```kotlin
val resultat = async {
    10 + 20
}.await()
println(resultat)
```

### 3. Suspend functions

Una funció marcada amb suspend pot ser pausada i continua en un altre moment.

```kotlin
suspend fun carregarDades(): String {
    delay(2000) // simula temps d’espera
    return "Dades carregades"
}
```

Només es pot cridar des de:

- una corrutina
- una altra funció suspend


## Dispatchers (On s’executen?)

Els dispatchers indiquen en quin fil s'executa la corrutina:

Dispatcher|Ús
|:--------|:---
Dispatchers.Main	|tasques de UI
Dispatchers.IO	|fitxers, base de dades, xarxa
Dispatchers.Default	|càlculs pesats
Dispatchers.Unconfined	|poc utilitzat


Exemple:

```kotlin
launch(Dispatchers.IO) {
    val dades = carregarDades()
    withContext(Dispatchers.Main) {
        // actualitzar UI
        textView.text = dades
    }
}
```

### Què és withContext?

**withContext** és una funció de suspensió (suspend) que permet canviar el dispatcher o context dins d’una corrutina i executar un bloc de codi en aquest nou context, sense crear una nova corrutina.

És útil per fer tasques de fons (IO, càlculs) i després tornar a la UI per actualitzar-la.

Retorna el resultat del bloc que executa.

```kotlin
val resultat = withContext(Dispatchers.IO) {
    // codi que s'executa en un fil d'entrada/sortida (IO)
    carregarDadesPesades()
}
```

## Diferència entre withContext launch i async
Funció|	Quan usar-la|	Retorna resultat?
|:----|:------------|:---------
launch	|Crear una corrutina nova	|No
async	|Crear corrutina que retorna un resultat	|Sí, Deferred.await()
withContext	|Canviar context dins d’una corrutina existent	|Sí, retorna el valor del bloc




