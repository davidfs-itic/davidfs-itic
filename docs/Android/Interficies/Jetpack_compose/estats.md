# Estats en Jetpack Compose

Un dels conceptes centrals de Compose és l'**estat**: les dades que poden canviar al llarg del temps i que determinen què es mostra en pantalla. Entendre com Compose detecta aquests canvis i quan torna a dibuixar la interfície (la **recomposició**) és imprescindible per treballar amb aquest toolkit.

Documentació oficial: https://developer.android.com/develop/ui/compose/state

## 1. Estats i recomposició

Un error molt habitual quan es comença amb Compose és fer servir una variable normal de Kotlin per guardar un valor que hauria de canviar la interfície:

```kotlin
@Composable
fun MyState(modifier: Modifier) {
    var numero = 0
    Text("Premut: ${numero}", modifier = modifier.clickable {
        numero += 1
        Log.i("Tag: MyState", "Premut ${numero}")
    })
}
```

Si es prova aquest codi, el `Log` mostra que `numero` sí que s'incrementa cada vegada que es prem el text, però **el text de la pantalla no canvia mai** i es queda sempre a "Premut: 0".

La causa és que Compose no té cap manera de saber que `numero` ha canviat. Una funció `@Composable` només es torna a executar (es *recomposa*) quan Compose detecta que una de les dades de les quals depèn ha canviat, i per detectar-ho necessita que aquesta dada estigui embolcallada en un objecte especial que ell mateix pugui observar. Una `var` normal és només una variable de Kotlin: el `clickable` la modifica, però ningú n'avisa Compose.

### `mutableStateOf` i `remember`

Per solucionar-ho calen dues peces:

- **`mutableStateOf(valorInicial)`**: crea un objecte `State<T>` observable. Quan es llegeix la seva propietat `.value` dins d'un composable, Compose "s'apunta" que aquell composable depèn d'aquest estat. Quan `.value` canvia, Compose sap exactament quins composables cal tornar a executar.
- **`remember { ... }`**: fa que el valor generat dins la lambda es conservi entre recomposicions. Sense `remember`, cada vegada que la funció es tornés a executar es tornaria a cridar `mutableStateOf(0)` i el valor es reiniciaria a 0.

Cal les dues coses alhora: `mutableStateOf` perquè el canvi es detecti, i `remember` perquè el valor sobrevisqui a la recomposició que aquest mateix canvi provoca.

```kotlin
@Composable
fun MyState(modifier: Modifier) {
    var numero = remember { mutableStateOf(0) }
    Text("Premut: ${numero.value}", modifier = modifier.clickable {
        numero.value += 1
        Log.i("Tag: MyState", "Premut ${numero.value}")
    })
}
```

Ara sí: cada clic modifica `numero.value`, Compose ho detecta, i torna a executar el `Text` per mostrar el nou valor.

### Recomposició selectiva

Un dels punts forts de Compose és que la recomposició és **intel·ligent**: si una funció composable no llegeix un estat que ha canviat, Compose no la torna a executar, encara que estigui "al costat" d'una altra que sí que ho fa.

```kotlin
@Composable
fun MyState(modifier: Modifier) {
    var numero = remember { mutableStateOf(0) }
    Column {
        Text("Premut: ${numero.value}", modifier = modifier.clickable {
            numero.value += 1
            Log.i("Tag: MyState", "Premut ${numero.value}")
        })
        Text("Premut: ??")
        Text("Premut: ${numero.value}", modifier = modifier.clickable {
            numero.value += 1
            Log.i("Tag: MyState", "Premut ${numero.value}")
        })
    }
}
```

En aquest exemple, els dos `Text` que llegeixen `numero.value` es recomposen quan es fa clic, però el `Text("Premut: ??")` del mig, que no depèn d'aquest estat, no es torna a executar mai. Aquest seguiment fi de dependències és el que fa que Compose sigui eficient encara que tota la UI estigui definida com una gran funció: no cal repintar tota la pantalla per canviar un sol valor.

## 2. `remember` i el cicle de vida de l'Activity

Com ja s'ha vist a [Activities](../activities.md), un canvi de configuració (per exemple, girar el dispositiu) fa que l'Activity es destrueixi i es torni a crear. Amb els composables passa el mateix: en una rotació de pantalla, tot l'arbre de composició es descarta i es torna a construir des de zero.

Això té una conseqüència important: **`remember` per si sol no sobreviu a un canvi de configuració**. El valor guardat amb `remember` es perd en girar la pantalla, exactament igual que es perdria una propietat normal d'una Activity que no s'hagués guardat al `Bundle` d'estat.

### `rememberSaveable`

Per solucionar-ho, Compose ofereix `rememberSaveable`, que funciona igual que `remember` però, a més, guarda el valor en un `Bundle` (el mateix mecanisme d'`onSaveInstanceState` que ja es fa servir a les Activities) i el restaura automàticament després d'un canvi de configuració.

```kotlin
@Composable
fun MyState(modifier: Modifier) {
    var numero = rememberSaveable { mutableStateOf(0) }
    Text("Premut: ${numero.value}", modifier = modifier.clickable {
        numero.value += 1
        Log.i("Tag: MyState", "Premut ${numero.value}")
    })
}
```

La regla pràctica és senzilla: si es vol que un valor sobrevisqui a una rotació de pantalla (o a qualsevol altre canvi de configuració), cal `rememberSaveable`; si només ha de sobreviure a la recomposició normal, n'hi ha prou amb `remember`.

### Delegació de propietats amb `by`

Fins ara s'ha accedit sempre al valor amb `.value` (`numero.value`). Kotlin permet simplificar això amb la paraula clau `by`, que fa servir el mecanisme de **propietats delegades** del llenguatge: en lloc que `numero` sigui un objecte `State<Int>` al qual cal demanar `.value`, `numero` esdevé directament un `Int`, i és el delegat qui s'encarrega, per darrere, de llegir i escriure el `.value` de l'estat.

```kotlin
@Composable
fun MyState(modifier: Modifier) {
    var numero by rememberSaveable { mutableStateOf(0) }
    Text("Premut: $numero", modifier = modifier.clickable {
        numero += 1
        Log.i("Tag: MyState", "Premut $numero")
    })
}
```

El comportament és idèntic a l'exemple anterior (el valor es continua guardant en un `State<Int>` observable i persistent), però el codi queda més net: es llegeix i s'escriu `numero` directament, com si fos una variable normal, sense haver de repetir `.value` a cada línia.

## 3. State hoisting

Fins ara, els estats vistos estan emmagatzemats **dins** del mateix composable que els fa servir. Però sovint interessa que aquest valor estigui disponible també fora d'aquell composable en concret: per exemple, perquè el necessiten diversos composables germans, o perquè s'ha de guardar en un `ViewModel` o en una altra classe.

La tècnica per aconseguir-ho s'anomena **state hoisting** (literalment, "elevar l'estat"): consisteix a treure la variable d'estat del composable que la mostra i pujar-la a un composable superior (el seu "pare" a l'arbre de composició), que passa a ser l'única font de la veritat (*single source of truth*). El composable inferior deixa de guardar cap estat propi: rep el valor ja calculat com a paràmetre, i rep també una funció de tipus *callback* (per exemple `onClick: () -> Unit`) per demanar que aquest valor canviï, sense modificar-lo directament ell mateix.

```kotlin
@Composable
fun MyStateAdvanced(modifier: Modifier) {
    var numero by remember { mutableStateOf(0) }

    Column {
        Number1(numero) { numero++ }
        Number2(numero) { numero-- }
    }
}

@Composable
fun Number1(numero: Int, onClick: () -> Unit) {
    Text("Aquest és el primer número: $numero", modifier = Modifier.clickable { onClick() })
}

@Composable
fun Number2(numero: Int, onClick: () -> Unit) {
    Text("Aquest és el segon número: $numero", modifier = Modifier.clickable { onClick() })
}
```

Aquí `numero` viu únicament a `MyStateAdvanced`. Els composables `Number1` i `Number2` no en saben res, no tenen el seu propi `remember`: només reben el valor actual i una lambda per demanar-ne el canvi. Aquest patró fa que `Number1` i `Number2` siguin composables *stateless* (sense estat propi), molt més fàcils de reutilitzar i de provar, ja que el seu comportament depèn únicament dels paràmetres que reben.

Aquest mateix principi és el que permet, més endavant, moure la lògica que modifica l'estat (l'increment, en aquest cas) fora dels composables i cap a un `ViewModel`: el composable arrel deixaria de tenir el seu propi `remember` i, en el seu lloc, llegiria un `State` exposat pel `ViewModel` i li delegaria la modificació del valor. Es tracta exactament del mateix mecanisme de *state hoisting*, portat un nivell més amunt, fora de la jerarquia de composables. Vegeu [ViewModel](../../Arquitectura/viewmodel.md) per a més detall sobre aquest patró.
