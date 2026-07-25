# El composable Button

`Button` és el composable que representa una acció que l'usuari pot executar amb un toc. Equival al `Button` o `MaterialButton` del sistema de Views, però igual que la resta de components de Compose, tot el seu comportament i aparença es defineix mitjançant paràmetres de la funció, no amb atributs XML.

Documentació oficial: https://developer.android.com/develop/ui/compose/components/button

## 1. Ús bàsic

Un `Button` necessita, com a mínim, un paràmetre `onClick` amb l'acció a executar, i un contingut (*content*) que sol ser un `Text`:

```kotlin
Button(onClick = { /* acció a executar */ }) {
    Text("Continuar")
}
```

El fet que el contingut del botó sigui una lambda composable (i no simplement un `text: String`) és el que permet posar-hi qualsevol combinació de composables a dins: text, icones, o tots dos alhora, com es veu al punt 3.

## 2. Jerarquia visual: variants de botó

Material Design defineix diverses variants de botó, pensades per expressar visualment la importància relativa de cada acció dins d'una mateixa pantalla. Totes comparteixen la mateixa API (`onClick`, `enabled`, contingut lambda); només canvia el seu aspecte per defecte:

```kotlin
Button(onClick = { }) { Text("Acció principal") }            // Fons ple: màxima èmfasi
FilledTonalButton(onClick = { }) { Text("Acció secundària") } // Fons tonal: èmfasi mitjà
OutlinedButton(onClick = { }) { Text("Acció alternativa") }   // Vora, sense fons: èmfasi mitjà-baix
TextButton(onClick = { }) { Text("Cancel·lar") }              // Sense fons ni vora: èmfasi mínim
ElevatedButton(onClick = { }) { Text("Amb ombra") }           // Fons ple amb elevació
```

Com a criteri general: en una mateixa pantalla només hi hauria d'haver **un** `Button` (l'acció principal); la resta d'accions relacionades (cancel·lar, opcions secundàries) haurien de fer servir `OutlinedButton` o `TextButton`, per no competir visualment amb l'acció principal.
Consulteu la documentació oficial, per veure on utilitzar cada tipus de botó.

## 3. Botó amb icona

Com que el contingut d'un `Button` és una lambda composable, es pot combinar una `Icon` i un `Text` dins d'una `Row`, seguint la convenció habitual de Material Design (icona a l'esquerra, separada del text amb un `Spacer` petit):

```kotlin
Button(onClick = { }) {
    Icon(
        imageVector = Icons.Default.Add,
        contentDescription = null,
        modifier = Modifier.size(ButtonDefaults.IconSize)
    )
    Spacer(Modifier.size(ButtonDefaults.IconSpacing))
    Text("Afegir")
}
```

`ButtonDefaults.IconSize` i `ButtonDefaults.IconSpacing` proporcionen les mides recomanades per Material Design, en lloc d'haver d'inventar valors arbitraris de `dp`.

## 4. Botó només amb icona: `IconButton`

Quan l'acció es representa únicament amb una icona (sense text), com per exemple una icona de "favorit" o de "tancar", no s'ha de fer servir `Button` amb només una `Icon` a dins: existeix un composable específic, `IconButton`, pensat per a aquest cas, que ja gestiona correctament la mida mínima d'àrea tocable que exigeix l'accessibilitat (48dp), encara que la icona visual sigui més petita.

```kotlin
IconButton(onClick = { /* tancar */ }) {
    Icon(Icons.Default.Close, contentDescription = "Tancar")
}
```

En aquest cas, com que no hi ha cap `Text` que expliqui l'acció, el `contentDescription` de la icona deixa de ser opcional: és l'única informació que tindrà un lector de pantalla per descriure què fa el botó.

## 5. Estat `enabled`

Tots els botons accepten un paràmetre `enabled` per desactivar-los (per exemple, mentre no s'han omplert tots els camps d'un formulari). Un botó desactivat es mostra automàticament amb un estil atenuat i deixa de respondre als tocs:

```kotlin
Button(
    onClick = { },
    enabled = formulariValid
) {
    Text("Enviar")
}
```

## 6. Colors del botó

Per defecte, un `Button` no porta cap color escrit al codi: els seus colors (fons i contingut) surten del `MaterialTheme.colorScheme` actiu de l'aplicació. Això és el que fa que, si l'app canvia de tema (per exemple, entre mode clar i mode fosc, o entre diferents esquemes de color dinàmic), tots els botons s'actualitzin automàticament sense haver-los de tocar un per un.

Quan cal sortir-se d'aquest comportament per defecte, es fa servir el paràmetre `colors`, passant-hi el resultat d'una funció `ButtonDefaults`:

```kotlin
Button(
    onClick = { },
    colors = ButtonDefaults.buttonColors(
        containerColor = Color.Red,
        contentColor = Color.White,
        disabledContainerColor = Color.LightGray,
        disabledContentColor = Color.DarkGray
    )
) {
    Text("Eliminar")
}
```

`ButtonDefaults.buttonColors(...)` accepta quatre colors, tots opcionals; els que no s'indiquin es continuen agafant del tema:

- **`containerColor`**: el color de fons del botó quan està actiu (`enabled = true`).
- **`contentColor`**: el color del contingut (text i icones) quan està actiu. Sol triar-se un color amb prou contrast sobre `containerColor`.
- **`disabledContainerColor`** / **`disabledContentColor`**: els colors quan `enabled = false`. Si no s'indiquen, Compose aplica una atenuació per defecte sobre els colors actius.

!!! info "Un `ButtonDefaults` diferent per a cada variant"
    Cada variant de botó té la seva pròpia funció de colors, amb els mateixos quatre paràmetres però anomenats de manera coherent amb el seu disseny: `ButtonDefaults.filledTonalButtonColors(...)` per a `FilledTonalButton`, `ButtonDefaults.outlinedButtonColors(...)` per a `OutlinedButton`, `ButtonDefaults.textButtonColors(...)` per a `TextButton`, i `ButtonDefaults.elevatedButtonColors(...)` per a `ElevatedButton`. Cal fer servir sempre la funció corresponent a la variant que s'estigui personalitzant.

En general, es recomana no personalitzar els colors llevat que hi hagi un motiu clar de disseny (per exemple, un botó d'eliminar en vermell): dependre del `MaterialTheme` és el que garanteix consistència visual a tota l'aplicació i compatibilitat automàtica amb el mode fosc.
