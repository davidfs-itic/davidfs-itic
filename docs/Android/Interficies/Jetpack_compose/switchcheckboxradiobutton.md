# Els composables Switch, Checkbox i RadioButton

`Switch`, `Checkbox` i `RadioButton` són els tres controls de selecció bàsics de Compose. Tots tres serveixen perquè l'usuari triï entre opcions, però cadascun s'utilitza en un context diferent: activar/desactivar una única opció, marcar diverses opcions independents, o triar una única opció d'entre un grup.

Documentació oficial: https://developer.android.com/develop/ui/compose/components/switch

## 1. Switch

`Switch` representa un interruptor d'encès/apagat per a **una sola opció independent** (per exemple, "Notificacions activades"). Igual que `TextField`, és un component controlat: rep l'estat actual amb `checked` i notifica els canvis amb `onCheckedChange`.

```kotlin
var notificacionsActivades by remember { mutableStateOf(true) }

Switch(
    checked = notificacionsActivades,
    onCheckedChange = { notificacionsActivades = it }
)
```

És habitual combinar-lo amb un `Text` descriptiu dins d'una `Row`, i fer que tota la fila reaccioni al toc (no només l'interruptor), per ampliar l'àrea tocable:

```kotlin
Row(
    modifier = Modifier
        .fillMaxWidth()
        .clickable { notificacionsActivades = !notificacionsActivades },
    verticalAlignment = Alignment.CenterVertically
) {
    Text("Notificacions", modifier = Modifier.weight(1f))
    Switch(checked = notificacionsActivades, onCheckedChange = { notificacionsActivades = it })
}
```

### `enabled`

Igual que la resta de components de selecció, `Switch` accepta un paràmetre `enabled` per desactivar-lo. Quan `enabled = false`, l'interruptor es mostra amb un estil atenuat i deixa de respondre als tocs, encara que el seu estat (`checked`) es continuï mostrant amb normalitat:

```kotlin
Switch(
    checked = notificacionsActivades,
    onCheckedChange = { notificacionsActivades = it },
    enabled = false
)
```

Aquest cas és habitual quan l'opció depèn d'una altra condició prèvia (per exemple, un `Switch` de "So de notificacions" que només té sentit si "Notificacions activades" ja està encès).

### `thumbContent`

Per defecte, el cercle mòbil de l'interruptor (el *thumb*) és un simple cercle de color pla. El paràmetre `thumbContent` permet substituir-lo per qualsevol composable, com una `Icon` petita que indiqui visualment l'estat actual:

```kotlin
Switch(
    checked = notificacionsActivades,
    onCheckedChange = { notificacionsActivades = it },
    thumbContent = if (notificacionsActivades) {
        {
            Icon(
                imageVector = Icons.Default.Check,
                contentDescription = null,
                modifier = Modifier.size(SwitchDefaults.IconSize)
            )
        }
    } else {
        null
    }
)
```

`thumbContent` és de tipus `(@Composable () -> Unit)?`, és a dir, una lambda composable opcional: es pot passar `null` (el comportament per defecte, sense contingut al thumb) o triar dinàmicament quina icona mostrar segons l'estat, com a l'exemple anterior, on només es mostra una icona de check quan l'interruptor està activat. `SwitchDefaults.IconSize` proporciona la mida recomanada per Material Design perquè la icona encaixi bé dins del thumb.

## 2. Checkbox

`Checkbox` s'utilitza quan hi ha **diverses opcions independents** entre si, i l'usuari en pot marcar tantes com vulgui (a diferència del `RadioButton`, que és excloent). La seva API és pràcticament idèntica a la de `Switch`: `checked` i `onCheckedChange`.

```kotlin
var acceptaTermes by remember { mutableStateOf(false) }

Row(verticalAlignment = Alignment.CenterVertically) {
    Checkbox(
        checked = acceptaTermes,
        onCheckedChange = { acceptaTermes = it }
    )
    Text("Accepto els termes i condicions")
}
```

Quan es tenen diverses opcions independents (per exemple, una llista d'ingredients a triar), cada `Checkbox` ha de tenir el seu propi estat, normalment guardat en una estructura de dades (una llista o un `Map`) en lloc d'una variable booleana per cada opció.

## 3. RadioButton

`RadioButton` s'utilitza quan l'usuari ha de triar **una única opció d'entre un grup** (per exemple, un mètode de pagament). A diferència de `Switch` i `Checkbox`, `RadioButton` no gestiona per si sol la lògica d'exclusivitat: només rep `selected` (si aquesta opció concreta és la triada) i `onClick`. És responsabilitat del codi que l'envolta assegurar-se que només una opció del grup estigui marcada alhora.

```kotlin
val opcions = listOf("Targeta", "Transferència", "Contra reemborsament")
var opcioSeleccionada by remember { mutableStateOf(opcions[0]) }

Column(Modifier.selectableGroup()) {
    opcions.forEach { opcio ->
        Row(
            modifier = Modifier
                .fillMaxWidth()
                .selectable(
                    selected = (opcio == opcioSeleccionada),
                    onClick = { opcioSeleccionada = opcio },
                    role = Role.RadioButton
                ),
            verticalAlignment = Alignment.CenterVertically
        ) {
            RadioButton(
                selected = (opcio == opcioSeleccionada),
                onClick = null // el clic es gestiona a nivell de Row amb .selectable
            )
            Text(opcio)
        }
    }
}
```

Alguns punts importants d'aquest patró:

- **Una sola font de veritat**: `opcioSeleccionada` guarda quina és l'opció triada; cada `RadioButton` només compara si ell mateix és aquesta opció (`opcio == opcioSeleccionada`).
- **`Modifier.selectableGroup()`** al contenidor: agrupa semànticament les opcions perquè els serveis d'accessibilitat les anunciïn com un sol grup de selecció, no com elements solts.
- **`Modifier.selectable(...)`** a cada fila: permet que tota la fila (no només el cercle del `RadioButton`) respongui al toc, ampliant l'àrea clicable de manera accessible. Quan es fa servir aquest patró, el `onClick` del `RadioButton` en si es deixa a `null`, perquè el clic ja el gestiona la `Row`.

## 4. Quin triar

- **`Switch`**: una opció, independent, d'encès/apagat immediat (sol aplicar-se l'efecte a l'instant, sense necessitat de confirmar amb un botó).
- **`Checkbox`**: diverses opcions independents, es pot marcar-ne qualsevol combinació (inclosa cap o totes).
- **`RadioButton`**: diverses opcions mútuament excloents, se n'ha de triar exactament una.
