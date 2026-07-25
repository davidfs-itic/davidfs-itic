# El composable Text

`Text` és el composable encarregat de mostrar text a la pantalla. Equival al `TextView` del sistema de Views, però en comptes de configurar-se amb atributs XML o crides posteriors (`setText`, `setTextColor`...), tots els seus paràmetres es passen directament a la funció composable.

Documentació oficial: https://developer.android.com/develop/ui/compose/text

## 1. Ús bàsic i paràmetres principals

En la seva forma més senzilla, `Text` només necessita la cadena a mostrar:

```kotlin
Text("Hola, Compose!")
```

A partir d'aquí, la mateixa funció accepta un bon nombre de paràmetres opcionals per controlar-ne l'aparença i el comportament. Els més habituals:

```kotlin
Text(
    text = "Hola, Compose!",
    modifier = Modifier.padding(8.dp),
    color = Color.DarkGray,
    fontSize = 20.sp,
    fontWeight = FontWeight.Bold,
    fontStyle = FontStyle.Italic,
    textDecoration = TextDecoration.Underline,
    textAlign = TextAlign.Center,
    lineHeight = 24.sp,
    maxLines = 2,
    overflow = TextOverflow.Ellipsis
)
```

- **`color`**: color del text. Es fa servir un `Color` de Compose, no un recurs de color XML.
- **`fontSize`**: mida del text, sempre en `sp` (*scale-independent pixels*), l'unitat pensada perquè respecti la mida de lletra que l'usuari ha configurat al sistema.
- **`fontWeight`**: gruix del text (`FontWeight.Normal`, `FontWeight.Bold`, `FontWeight.Light`...).
- **`fontStyle`**: `FontStyle.Italic` o `FontStyle.Normal`.
- **`textDecoration`**: subratllat (`TextDecoration.Underline`), ratllat (`TextDecoration.LineThrough`), o cap (`TextDecoration.None`).
- **`textAlign`**: alineació horitzontal del text dins de l'espai que ocupa (`TextAlign.Center`, `TextAlign.Start`, `TextAlign.End`, `TextAlign.Justify`). Cal tenir en compte que això només té efecte visible si el `Text` ocupa més amplada que la que necessita el seu contingut (per exemple, amb `Modifier.fillMaxWidth()`).
- **`lineHeight`**: alçada de línia, útil quan el text ocupa diverses línies.
- **`maxLines`** i **`overflow`**: limiten el nombre de línies visibles. Quan el text no hi cap, `TextOverflow.Ellipsis` afegeix "..." al final; `TextOverflow.Clip` el retalla sense indicador.

!!! info "`sp` vs `dp`"
    Per a mides de text sempre s'utilitza `sp`, mai `dp`. La diferència és que `sp` escala també amb la configuració d'accessibilitat de mida de lletra del sistema, mentre que `dp` només escala amb la densitat de la pantalla. Fer servir `dp` en text és un error comú que trenca l'accessibilitat de la app.

## 2. Estil amb `TextStyle` i `MaterialTheme.typography`

Quan cal aplicar la mateixa combinació d'estils (mida, gruix, família tipogràfica...) a molts `Text` diferents, repetir tots els paràmetres a cada crida és poc pràctic i fàcil de desincronitzar. Per això `Text` també accepta un paràmetre `style` de tipus `TextStyle`, que agrupa totes aquestes propietats en un sol objecte:

```kotlin
val titolEstil = TextStyle(
    fontSize = 22.sp,
    fontWeight = FontWeight.Bold,
    color = Color.Black
)

Text("Títol de la pantalla", style = titolEstil)
```

Compose, però, ja defineix un conjunt d'estils tipogràfics coherents a través del tema de l'aplicació (recordeu l'arxiu `Type.kt` esmentat a la [Introducció](./index.md)), accessibles amb `MaterialTheme.typography`:

```kotlin
Text("Títol de la pantalla", style = MaterialTheme.typography.titleLarge)
Text("Text normal de cos", style = MaterialTheme.typography.bodyMedium)
Text("Text petit, secundari", style = MaterialTheme.typography.labelSmall)
```

Fer servir `MaterialTheme.typography` en lloc de valors solts (`fontSize = 22.sp`, etc.) escampats per tota la app té un avantatge important: si més endavant es vol canviar la tipografia general de l'aplicació, n'hi ha prou de modificar-la en un sol lloc (`Type.kt`), i el canvi es propaga automàticament a tots els `Text` que facin servir el tema.

Si es vol partir d'un estil del tema però modificar-ne només algun detall puntual, `TextStyle` es pot combinar amb `copy()`:

```kotlin
Text(
    text = "Títol destacat",
    style = MaterialTheme.typography.titleLarge.copy(color = Color.Red)
)
```

## 3. Text amb diversos estils: `AnnotatedString`

De vegades cal que **dins d'un mateix `Text`** hi hagi fragments amb estils diferents (per exemple, una paraula en negreta enmig d'una frase normal). Passar-hi directament una `String` no ho permet, ja que els paràmetres com `fontWeight` o `color` s'apliquen a tot el text sencer.

Per a aquests casos, `Text` accepta també un `AnnotatedString`, que es construeix amb la funció `buildAnnotatedString`. Dins d'aquesta funció es pot delimitar quins trossos de text porten quin estil amb `withStyle`:

```kotlin
Text(
    text = buildAnnotatedString {
        append("Aquest text és normal, però ")
        withStyle(style = SpanStyle(fontWeight = FontWeight.Bold, color = Color.Red)) {
            append("aquesta part")
        }
        append(" és en negreta i vermell.")
    }
)
```

Aquest mecanisme és la base per a casos com ressaltar coincidències d'una cerca, marcar camps obligatoris amb un asterisc d'un altre color, o construir text amb enllaços (vegeu el punt següent).

## 4. Text seleccionable i enllaços

### Text seleccionable

Per defecte, el text mostrat amb `Text` **no es pot seleccionar** ni copiar (a diferència del `TextView` tradicional). Si es vol permetre-ho, cal embolcallar el `Text` (o els `Text` que calgui) amb `SelectionContainer`:

```kotlin
SelectionContainer {
    Text("Aquest text es pot seleccionar i copiar.")
}
```

### Enllaços dins del text

Per convertir un fragment de text en un enllaç clicable (per exemple, obrir una URL o executar una acció concreta en tocar només una paraula), es combina un `AnnotatedString` amb una anotació de tipus `LinkAnnotation`, associada al fragment de text corresponent amb `withLink`:

```kotlin
Text(
    text = buildAnnotatedString {
        append("Consulta la ")
        withLink(
            LinkAnnotation.Url(
                url = "https://developer.android.com/develop/ui/compose/text",
                styles = TextLinkStyles(
                    style = SpanStyle(color = Color.Blue, textDecoration = TextDecoration.Underline)
                )
            )
        ) {
            append("documentació oficial")
        }
        append(" per ampliar aquest apartat.")
    }
)
```

Amb això, només el fragment "documentació oficial" reacciona al toc i obre l'enllaç, mentre que la resta del text es queda com a text pla.

## 5. Text a partir de recursos (`stringResource`)

Igual que amb les Views, els textos no s'haurien d'escriure literalment (*hardcoded*) dins del codi, sinó definir-se a `res/values/strings.xml` i recuperar-se amb `stringResource`. Això és el que permet, entre altres coses, traduir la app a diferents idiomes sense tocar cap composable:

```kotlin
Text(text = stringResource(id = R.string.benvinguda))
```

Si la cadena necessita paràmetres, es passen directament a `stringResource`, igual que es faria amb `getString(...)` en una Activity:

```kotlin
// <string name="salutacio">Hola, %1$s!</string>
Text(text = stringResource(id = R.string.salutacio, "Món"))
```
