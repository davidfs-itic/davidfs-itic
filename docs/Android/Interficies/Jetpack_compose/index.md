# Introducció a Jetpack Compose

Jetpack Compose és el toolkit modern d'Android per construir interfícies d'usuari de forma **declarativa**. En comptes de dissenyar la interfície en XML i manipular-la des del codi (Views, `findViewById`, ViewBinding...), amb Compose la UI es defineix directament amb funcions Kotlin que descriuen "com ha de ser la pantalla" en cada moment, en funció del seu estat.

Documentació oficial: https://developer.android.com/develop/ui/compose/documentation

Exemples i diferències entre Material i Material3: https://www.jetpackcompose.pro/home/guide/

## 1. Paradigma declaratiu vs. sistema de Views

En el sistema tradicional (XML + Views), l'interfície s'infla una vegada i després cal anar-la actualitzant manualment: `text.setText(...)`, `imageView.setImageResource(...)`, etc. El programador és responsable de mantenir la vista sincronitzada amb les dades.

En Compose, es descriu la interfície com una funció de l'estat: `UI = f(estat)`. Quan l'estat canvia, Compose torna a executar (recompon) les funcions afectades i actualitza automàticament només allò que cal, sense que calgui buscar vistes ni actualitzar-les a mà. Aquest mecanisme es coneix com **recomposició**.

Aquest canvi de paradigma és el mateix que trobem en altres frameworks moderns d'UI declarativa (SwiftUI a iOS, React o Flutter), i és important interioritzar-lo perquè afecta a tota l'arquitectura de la app.

## 2. Arquitectura: una sola Activity

En una app basada en Compose, deixem de tenir una Activity per pantalla. En lloc d'això, la app sol tenir **una única Activity** (normalment `MainActivity`), que actua com a contenidor de tota la interfície. Les diferents "pantalles" de la app deixen de ser Activities o Fragments, i passen a ser simplement **funcions composables** que es criden les unes a les altres.

Aquest enfocament s'anomena *Single Activity Architecture*. Els avantatges principals són:

- No cal gestionar el cicle de vida de múltiples Activities ni el pas de dades entre elles amb Intents i Bundles.
- La navegació entre pantalles passa a ser responsabilitat d'una llibreria de navegació (Navigation Compose), que simplement canvia quin composable es mostra.
- Compartir estat entre pantalles és molt més senzill, ja que totes viuen dins del mateix arbre de composició.

A la pràctica, un projecte nou de Compose ja ve preparat amb aquesta estructura: una `MainActivity` que crida un composable arrel, i a partir d'aquí anem afegint noves funcions `@Composable` per a cada pantalla o component.

!!! info "Colors i estils"
    Als projectes generats per defecte, els colors i els estils tipogràfics (Type) es defineixen en dos llocs diferents segons si es fan servir Views (XML, `colors.xml`, `styles.xml`) o Compose (arxius Kotlin `Color.kt`, `Type.kt`, `Theme.kt`). És important no confondre'ls: en un projecte purament Compose, els colors i tipografies es defineixen com a valors de Kotlin, no com a recursos XML.

## 3. Anotadors `@Composable` i `@Preview`

### `@Composable`

Qualsevol funció que vulgui descriure una part de la interfície s'ha de marcar amb l'anotador `@Composable`. Aquest anotador li indica al compilador de Compose que aquesta funció pot cridar altres funcions composables i que participa en el mecanisme de recomposició.

```kotlin
@Composable
fun Greeting(name: String) {
    Text(text = "Hola, $name!")
}
```

Una funció composable no retorna cap View: la seva "sortida" és l'efecte d'emetre components a l'arbre d'interfície que gestiona Compose internament.

### `@Preview`

L'anotador `@Preview` permet visualitzar un composable directament des de l'editor d'Android Studio, sense necessitat d'executar la app en un emulador o dispositiu. És una de les grans millores de productivitat de Compose respecte al sistema anterior.

```kotlin
@Preview(showBackground = true, showSystemUi = true)
@Composable
fun GreetingPreview() {
    Greeting(name = "Món")
}
```

Paràmetres més habituals del `@Preview`:

- **`showBackground`**: mostra un fons (per defecte transparent) perquè es vegi bé el component.
- **`showSystemUi`**: mostra la barra d'estat i la barra de navegació del sistema, simulant una pantalla completa.
- **`device`**: permet especificar un dispositiu concret de referència (mida, densitat...) per veure com es renderitza el composable, per exemple `device = Devices.PIXEL_4`.

Es poden combinar diversos `@Preview` sobre la mateixa funció per veure-la en diferents condicions (mode clar/fosc, mides de pantalla diferents, etc.).

## 4. Modificadors (`Modifier`)

Els `Modifier` són l'eina amb què es controla l'aspecte i el comportament d'un composable: mida, posició, padding, fons, capacitat de fer scroll, gestos, etc. Tot allò que en el món de les Views es feia amb atributs XML (`layout_width`, `layout_margin`, `background`...) a Compose es fa encadenant funcions sobre un objecte `Modifier`.

```kotlin
Text(
    text = "Hola",
    modifier = Modifier
        .padding(16.dp)
        .background(Color.LightGray)
)
```

Alguns punts importants a tenir en compte:

- **Mida per defecte**: per defecte, els composables ocupen només l'espai que necessita el seu contingut, de manera semblant a `wrap_content` en XML. Si es vol que ocupi tot l'espai disponible del seu contenidor, cal indicar-ho explícitament amb `Modifier.fillMaxSize()` (equivalent aproximat a `match_parent`), o bé `fillMaxWidth()` / `fillMaxHeight()` si només es vol en una dimensió.

- **No existeix `margin`, només `padding`**: en Compose no hi ha un concepte equivalent al `layout_margin` de les Views. Tot l'espaiat es resol amb `padding`, aplicat sobre el mateix component o sobre el seu contenidor. Si es necessita separació entre dos elements, sovint es fa servir un `Spacer` (vegeu [Layouts](./layouts.md)).

- **L'ordre dels modificadors importa**: cada modificador s'aplica embolcallant els anteriors, com si fossin capes. Per tant, `Modifier.padding(16.dp).background(Color.Red)` no és el mateix que `Modifier.background(Color.Red).padding(16.dp)`:

```kotlin
// El padding queda FORA del fons: es veu un marge sense color
Modifier
    .padding(16.dp)
    .background(Color.Red)

// El padding queda DINS del fons: el color ocupa tot l'espai,
// i el contingut es desplaça cap a dins
Modifier
    .background(Color.Red)
    .padding(16.dp)
```

Per aquest motiu, sempre que un component es comporti de manera inesperada (una mida que no és la que s'esperava, un fons que no cobreix el que tocaria...), val la pena revisar l'ordre en què s'han encadenat els modificadors.

## Components bàsics

- [Layouts: Box, Column, Row i ConstraintLayout](./layouts.md)
- [Estats: mutableStateOf, remember i state hoisting](./estats.md)
- [El composable Text](./text.md)
- [El composable TextField](./textfield.md)
- [El composable Button](./button.md)
- [Els composables Image i Icon](./imageicon.md)
- [Els composables de progrés](./progressbar.md)
- [Switch, Checkbox i RadioButton](./switchcheckboxradiobutton.md)
- [El composable Slider](./slider.md)
- [El composable DropdownMenu](./dropdownmenu.md)
- [El composable Scaffold](./scaffold.md)
- [NavigationDrawer](./navigationdrawer.md)
- [Navigation Compose](./navigationcompose.md)
- [AlertDialog](./alertdialog.md)
- [DatePickerDialog](./datepickerdialog.md)
- [Dialog](./dialog.md)

## Gestió avançada de comportaments
- [InteractionSource](./interactionsource.md)
- [LaunchedEffect](./launchedeffect.md)
- [derivedStateOf](./derivedstateof.md)

## Llistats

- [El composable LazyColumn](./lazycolumn.md)
- [LazyColumn: botó Tornar a dalt](./lazycolumnscrolltop.md)
- [LazyColumn: cerca i filtrat](./lazycolumnfilter.md)
- [LazyColumn: swipe to dismiss](./lazycolumnswipetodismiss.md)