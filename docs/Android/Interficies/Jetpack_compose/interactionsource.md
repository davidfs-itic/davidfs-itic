# InteractionSource

`InteractionSource` permet observar les interaccions que un usuari fa sobre un composable —si l'està prement, si té el focus, si el ratolí hi passa per sobre...— més enllà del simple `onClick`. És l'eina que cal fer servir quan es vol que l'aspecte d'un component reaccioni a *com* l'usuari hi interactua, no només a què hi passa quan acaba d'interactuar-hi.

Documentació oficial: https://developer.android.com/develop/ui/compose/touch-input/pointer-input/handle-interaction

## 1. Concepte: `onClick` només informa del resultat final

Un `onClick` es dispara quan l'usuari prem i deixa anar un component: no diu res sobre l'instant en què el dit encara està sobre la pantalla. Si es vol, per exemple, que un botó s'enfosqueixi mentre està sent premut (i torni al color normal en deixar-lo anar), `onClick` no serveix, perquè només arriba al final del gest.

`MutableInteractionSource` resol això: és un flux d'esdeveniments d'interacció (`PressInteraction.Press`, `FocusInteraction.Focus`, etc.) que es pot associar a un component i observar en temps real durant tota la interacció, no només al final.

## 2. Escoltar si un component està premut

El cas més habitual és saber si un component està premut en aquest mateix instant, amb `collectIsPressedAsState()`:

```kotlin
val interactionSource = remember { MutableInteractionSource() }
val premut by interactionSource.collectIsPressedAsState()

Box(
    modifier = Modifier
        .background(if (premut) Color.DarkGray else Color.Gray)
        .clickable(
            interactionSource = interactionSource,
            indication = null,
            onClick = { /* acció */ }
        )
        .padding(16.dp)
) {
    Text("Prem-me", color = Color.White)
}
```

- **`remember { MutableInteractionSource() }`**: es crea una sola vegada i es manté durant la vida del composable.
- **`collectIsPressedAsState()`**: converteix el flux d'interaccions en un `State<Boolean>`, de manera que es pot fer servir directament dins de la composició (recomposant automàticament quan canvia, igual que qualsevol altre estat de [Estats en Jetpack Compose](./estats.md)).
- **`indication = null`** al `clickable`: es desactiva l'efecte ripple per defecte de Material, ja que en aquest exemple l'efecte visual es controla manualment amb el canvi de color.

## 3. Altres tipus d'interacció

`InteractionSource` no es limita a `pressed`. Els mateixos component `clickable`, `Button`, `TextField`, etc. accepten un `interactionSource`, i s'hi pot preguntar per altres estats amb funcions equivalents:

- **`collectIsPressedAsState()`**: si el component està sent premut.
- **`collectIsFocusedAsState()`**: si el component té el focus (rellevant sobretot en `TextField` o en navegació amb teclat).
- **`collectIsHoveredAsState()`**: si el punter (ratolí, stylus) hi passa per sobre, sense arribar a prémer.
- **`collectIsDraggedAsState()`**: si el component està sent arrossegat.

```kotlin
val interactionSource = remember { MutableInteractionSource() }
val enfocat by interactionSource.collectIsFocusedAsState()

TextField(
    value = text,
    onValueChange = { text = it },
    interactionSource = interactionSource,
    label = { Text(if (enfocat) "Escrivint..." else "Nom") }
)
```

## 4. Cas d'ús: personalitzar l'aspecte segons la interacció

El motiu principal per fer servir `InteractionSource` és aplicar canvis visuals immediats (escala, color, elevació) que depenen de l'instant exacte de la interacció, sense haver de gestionar aquest estat manualment amb `onClick`/`onRelease` per separat:

```kotlin
@Composable
fun BotoAmbEscala(text: String, onClick: () -> Unit) {
    val interactionSource = remember { MutableInteractionSource() }
    val premut by interactionSource.collectIsPressedAsState()
    val escala = if (premut) 0.95f else 1f

    Button(
        onClick = onClick,
        interactionSource = interactionSource,
        modifier = Modifier.graphicsLayer(scaleX = escala, scaleY = escala)
    ) {
        Text(text)
    }
}
```

Aquest botó s'encongeix lleugerament mentre l'usuari el manté premut, i torna a la mida original en deixar-lo anar, un efecte que no es podria aconseguir només amb `onClick`.

## 5. Quan fer-lo servir

`InteractionSource` només cal quan l'aspecte d'un component ha de dependre de l'instant exacte de la interacció (mentre es prem, mentre té el focus...). Per a la majoria de casos —executar una acció quan l'usuari toca un botó— n'hi ha prou amb `onClick`; introduir `InteractionSource` sense necessitat només afegeix complexitat.
