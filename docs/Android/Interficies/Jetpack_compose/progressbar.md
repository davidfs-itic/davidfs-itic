# Els composables de progrés (ProgressIndicator)

Compose ofereix dos composables per indicar visualment que una operació està en curs: `CircularProgressIndicator` (un cercle giratori) i `LinearProgressIndicator` (una barra horitzontal). Equivalen al `ProgressBar` del sistema de Views, en les seves variants circular i horitzontal.

Documentació oficial: https://developer.android.com/develop/ui/compose/components/progress-indicators

## 1. Indicadors indeterminats

Quan no se sap quant trigarà una operació (per exemple, esperant la resposta d'una petició de xarxa), es fa servir un indicador **indeterminat**: gira o es mou contínuament, sense representar cap percentatge concret. N'hi ha prou de cridar el composable sense paràmetre de progrés:

```kotlin
CircularProgressIndicator()

LinearProgressIndicator(modifier = Modifier.fillMaxWidth())
```

Aquest és el cas d'ús més habitual: mostrar-lo mentre es carreguen dades i amagar-lo quan ja han arribat.

## 2. Indicadors determinats

Quan sí que se sap quin percentatge s'ha completat (per exemple, la baixada d'un fitxer), es fa servir la variant **determinada**, passant el progrés actual com un valor entre `0f` i `1f`:

```kotlin
var progres by remember { mutableStateOf(0f) }

CircularProgressIndicator(progress = { progres })

LinearProgressIndicator(
    progress = { progres },
    modifier = Modifier.fillMaxWidth()
)
```

!!! info "El paràmetre `progress` és una lambda"
    A les versions actuals de Compose Material 3, `progress` no s'indica com un simple `Float`, sinó com una funció que el retorna (`progress = { progres }`). Aquest disseny permet que Compose llegeixi el valor només quan cal redibuixar l'indicador, sense provocar recomposicions innecessàries d'altres parts de la pantalla. En versions més antigues de la llibreria es pot trobar encara la forma `progress = progres`, directament amb el `Float`.

El valor de `progres` ha d'anar-se actualitzant des del codi que controla l'operació real (per exemple, dins d'una coroutine que llegeix el percentatge d'una baixada), i com que és un `mutableStateOf`, cada actualització recompon automàticament l'indicador.

## 3. Personalització

Ambdós indicadors accepten paràmetres per ajustar el seu aspecte:

```kotlin
CircularProgressIndicator(
    progress = { progres },
    color = Color.Green,
    strokeWidth = 6.dp,
    trackColor = Color.LightGray
)
```

- **`color`**: color de la part que representa el progrés completat.
- **`trackColor`**: color del "camí" de fons, és a dir, la part encara no completada.
- **`strokeWidth`** (només `CircularProgressIndicator`): gruix del traç del cercle.

## 4. Cas d'ús pràctic: mostrar durant una càrrega

Un patró molt habitual és combinar un indicador indeterminat amb un estat booleà que representa si s'està carregant, mostrant l'indicador o el contingut real segons el seu valor:

```kotlin
@Composable
fun PantallaAmbCarrega() {
    var carregant by remember { mutableStateOf(true) }

    Box(modifier = Modifier.fillMaxSize(), contentAlignment = Alignment.Center) {
        if (carregant) {
            CircularProgressIndicator()
        } else {
            Text("Dades carregades correctament")
        }
    }
}
```

Aquí, `carregant` seria normalment actualitzat des d'un `ViewModel` un cop finalitzada la càrrega real de dades; l'indicador de progrés desapareix automàticament tan bon punt aquest valor passa a `false`, gràcies al mateix mecanisme de recomposició explicat a [Estats en Jetpack Compose](./estats.md).
