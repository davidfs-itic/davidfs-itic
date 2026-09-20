# Theming a Jetpack Compose

En el sistema de Views, el tema d'una app es defineix amb recursos XML: `colors.xml`, `styles.xml` i `themes.xml`. En Jetpack Compose aquest paper el fa el composable `MaterialTheme`, que centralitza els tres subsistemes de Material Design 3 —**esquema de colors**, **tipografia** i **formes**— en objectes de Kotlin normals (habitualment definits a `Color.kt`, `Type.kt` i `Theme.kt`), en lloc de recursos.

Documentació oficial: https://developer.android.com/develop/ui/compose/designsystems/material3

## 1. Concepte: `MaterialTheme` embolcalla tota l'app

Un projecte de Compose crea, per defecte, un composable arrel (per exemple `MeuaAppTheme`) que embolcalla tot el contingut amb `MaterialTheme`, passant-li els tres subsistemes:

```kotlin
MaterialTheme(
    colorScheme = colorScheme,
    typography = typography,
    shapes = shapes
) {
    // Tota la UI de l'app
}
```

Qualsevol composable definit dins d'aquest bloc pot llegir aquests valors amb `MaterialTheme.colorScheme`, `MaterialTheme.typography` i `MaterialTheme.shapes`, de la mateixa manera que en XML es feia referència a `?attr/colorPrimary` o `@style/AppTheme`. La diferència clau és que aquí no hi ha cap resolució de recursos: són simplement propietats d'un objecte Kotlin.

## 2. Esquema de colors (`ColorScheme`)

Material 3 defineix un `ColorScheme` amb un esquema clar (`lightColorScheme`) i un de fosc (`darkColorScheme`), normalment declarats a `Theme.kt` a partir de colors bàsics definits a `Color.kt`:

```kotlin
// Color.kt
val md_theme_light_primary = Color(0xFF476810)
val md_theme_light_onPrimary = Color(0xFFFFFFFF)
val md_theme_light_primaryContainer = Color(0xFFC7F089)

val md_theme_dark_primary = Color(0xFFACD370)
val md_theme_dark_onPrimary = Color(0xFF213600)
val md_theme_dark_primaryContainer = Color(0xFF324F00)
```

```kotlin
// Theme.kt
private val LightColorScheme = lightColorScheme(
    primary = md_theme_light_primary,
    onPrimary = md_theme_light_onPrimary,
    primaryContainer = md_theme_light_primaryContainer
)

private val DarkColorScheme = darkColorScheme(
    primary = md_theme_dark_primary,
    onPrimary = md_theme_dark_onPrimary,
    primaryContainer = md_theme_dark_primaryContainer
)
```

No cal escriure aquests colors a mà: l'eina [Material Theme Builder](https://m3.material.io/theme-builder) genera automàticament tot el `ColorScheme` (clar i fosc) a partir d'un sol color base.

## 3. Ús dels colors: `MaterialTheme.colorScheme`

Els colors no s'apliquen mai amb un valor fix (`Color(0xFF...)`) directament al codi de la pantalla, sinó llegint-los del tema:

```kotlin
Text(
    text = "Hola theming",
    color = MaterialTheme.colorScheme.primary
)
```

Material 3 associa a cada color principal (`primary`, `secondary`, `tertiary`, `surface`...) un color "on-" pensat perquè hi tingui prou contrast a sobre (`onPrimary`, `onSecondary`, `onSurface`...). La regla pràctica és **utilitzar sempre la parella corresponent**:

```kotlin
Card(
    colors = CardDefaults.cardColors(
        containerColor = if (seleccionat) MaterialTheme.colorScheme.primaryContainer
                          else MaterialTheme.colorScheme.surfaceVariant
    )
) {
    Text(
        text = "Sopar en grup",
        style = MaterialTheme.typography.bodyLarge,
        color = if (seleccionat) MaterialTheme.colorScheme.onPrimaryContainer
                else MaterialTheme.colorScheme.onSurface
    )
}
```

Si es combina, per exemple, `primaryContainer` com a fons amb `primary` com a color de text (en lloc del seu "on-" corresponent), el contrast no queda garantit i l'accessibilitat de la pantalla se'n ressent.

## 4. Cada component usa un rol de color concret per defecte

Els components de Material 3 no trien el seu aspecte de forma arbitrària: cadascun té assignat, per defecte, un rol específic del `ColorScheme` (no necessàriament `primary` o `surface`). Per exemple, `ElevatedCard` utilitza per defecte `surfaceContainerLow` com a color de fons:

```kotlin
ElevatedCard {
    Text("El fons d'aquesta targeta és surfaceContainerLow, no surface")
}
```

Això explica per què, en alguns casos, canviar un color del tema "no fa res": si es redefineix `primary` o `surface` però el component en qüestió llegeix `surfaceContainerLow`, el seu aspecte no varia. Per personalitzar-lo cal definir explícitament aquest rol concret en el `lightColorScheme` / `darkColorScheme`:

```kotlin
private val LightColorScheme = lightColorScheme(
    primary = md_theme_light_primary,
    onPrimary = md_theme_light_onPrimary,
    surfaceContainerLow = Color(0xFFF5F0E6)
)
```

Un cop definit, **tots** els components que utilitzen `surfaceContainerLow` (no només `ElevatedCard`) adopten el nou color automàticament, ja que en cap lloc del codi de pantalla s'ha escrit un valor fix.

!!! tip "Com saber quin rol de color usa cada component"
    La [documentació de components de Material 3](https://m3.material.io/components) mostra, per a cada component i cadascuna de les seves variants (per exemple `Card` amb `filled`, `elevated` i `outlined`), l'especificació ("specs") amb el rol de color exacte que s'aplica a cada part: fons, contorn, text, icona... Consultar-la abans de sobreescriure un color evita haver d'esbrinar-ho per prova i error.

## 5. Tipografia (`Typography`)

De la mateixa manera que els colors, la tipografia es defineix com un objecte `Typography` amb 15 estils amb nom (`displayLarge`, `headlineMedium`, `titleLarge`, `bodyMedium`, `labelSmall`...), cadascun amb la seva mida, alçada de línia i pes per defecte:

```kotlin
// Type.kt
val meuaTypography = Typography(
    titleLarge = TextStyle(
        fontWeight = FontWeight.SemiBold,
        fontSize = 22.sp,
        lineHeight = 28.sp,
        letterSpacing = 0.sp
    ),
    bodyLarge = TextStyle(
        fontWeight = FontWeight.Normal,
        fontFamily = FontFamily.SansSerif,
        fontSize = 16.sp,
        lineHeight = 24.sp,
        letterSpacing = 0.15.sp
    )
)
```

Un cop passada a `MaterialTheme(typography = meuaTypography)`, s'utilitza igual que els colors:

```kotlin
Text(text = "Títol de la pantalla", style = MaterialTheme.typography.titleLarge)
Text(text = "Text de cos", style = MaterialTheme.typography.bodyMedium)
```

## 6. Fonts personalitzades (arxius `.ttf`)

Els estils de `Typography` no estan limitats a les famílies tipogràfiques del sistema (`FontFamily.SansSerif`, `FontFamily.Serif`...). També es pot utilitzar una font pròpia descarregada en format `.ttf` (o `.otf`).

Documentació oficial: https://developer.android.com/develop/ui/compose/text/fonts

### Pas 1: afegir l'arxiu a `res/font`

Cal copiar l'arxiu `.ttf` dins la carpeta `res/font` del projecte (si no existeix, es crea amb clic dret sobre `res` → **New → Android Resource Directory**, tipus `font`). El nom de l'arxiu ha de seguir les mateixes normes que qualsevol recurs Android: només minúscules, xifres i `_`, sense espais ni majúscules.

```
res/
└── font/
    ├── montserrat_regular.ttf
    ├── montserrat_medium.ttf
    └── montserrat_bold.ttf
```

### Pas 2: crear un `FontFamily`

A `Type.kt` es defineix un `FontFamily` que associa cada arxiu al seu `FontWeight` corresponent:

```kotlin
// Type.kt
import androidx.compose.ui.text.font.Font
import androidx.compose.ui.text.font.FontFamily
import androidx.compose.ui.text.font.FontWeight
import com.exemple.meuaapp.R // el R generat del propi projecte, no android.R

val Montserrat = FontFamily(
    Font(R.font.montserrat_regular, FontWeight.Normal),
    Font(R.font.montserrat_medium, FontWeight.Medium),
    Font(R.font.montserrat_bold, FontWeight.Bold)
)
```

Si només es té un únic pes de la font (per exemple, només `Regular`), n'hi ha prou amb una sola línia dins el `FontFamily`.

### Fonts variables (variable fonts)

Una font variable és un **únic arxiu `.ttf`** que conté tots els pesos (i de vegades també l'ample o la inclinació) codificats com a eixos interpolables, en lloc de tenir un arxiu independent per a cada pes. Per tant, només cal copiar-la **una vegada** a `res/font/`.

Per obtenir diversos pesos a partir d'aquest únic arxiu, cal referenciar el mateix `resId` diverses vegades dins el `FontFamily`, indicant a cadascuna quin punt de l'eix de pes s'ha de fer servir amb `variationSettings`:

```kotlin
import androidx.compose.ui.text.font.FontVariation

val Montserrat = FontFamily(
    Font(
        resId = R.font.montserrat_variable,
        weight = FontWeight.Normal,
        variationSettings = FontVariation.Settings(FontVariation.weight(400))
    ),
    Font(
        resId = R.font.montserrat_variable,
        weight = FontWeight.Bold,
        variationSettings = FontVariation.Settings(FontVariation.weight(700))
    )
)
```

!!! warning "Sense variationSettings no hi ha diferència de pes"
    Si es referencia el mateix `resId` tres vegades amb `FontWeight` diferents però **sense** `variationSettings`, Compose pinta sempre la mateixa instància per defecte de la font: visualment no hi haurà cap diferència entre "Normal" i "Bold". Cal `variationSettings` per triar realment un punt concret de l'eix.

    Aquesta API requereix `minSdk 26` i Compose UI 1.2 o superior; en dispositius més antics es farà servir la instància estàtica per defecte de la font, independentment del pes sol·licitat.

!!! warning "R.font no troba la font"
    Si Android Studio no reconeix `R.font.montserrat_regular`, comprovar:

    - Que l'arxiu és realment dins `res/font/` (no `res/raw/`), amb extensió `.ttf` o `.otf` (a vegades una descàrrega deixa una doble extensió com `.ttf.zip`).
    - Que el nom segueix les normes de recursos Android: minúscules, xifres i `_`, sense guions ni majúscules.
    - Que s'ha fet **Build → Rebuild Project** (o *Sync Project with Gradle Files*) després d'afegir l'arxiu, ja que la classe `R` només es regenera en compilar.
    - Que l'import de `R` és el del propi paquet de l'app i no `android.R`, que Android Studio a vegades suggereix per error si ja hi havia una altra referència a `android.R` al mateix arxiu.

### Pas 3: assignar-la a la `Typography`

El `FontFamily` s'utilitza igual que `FontFamily.SansSerif` a l'exemple anterior, assignant-lo al camp `fontFamily` de cada `TextStyle` que ha d'utilitzar la font personalitzada:

```kotlin
val meuaTypography = Typography(
    titleLarge = TextStyle(
        fontFamily = Montserrat,
        fontWeight = FontWeight.SemiBold,
        fontSize = 22.sp,
        lineHeight = 28.sp
    ),
    bodyLarge = TextStyle(
        fontFamily = Montserrat,
        fontWeight = FontWeight.Normal,
        fontSize = 16.sp,
        lineHeight = 24.sp,
        letterSpacing = 0.15.sp
    )
)
```

!!! tip "Aplicar la font a tots els estils alhora"
    Si es vol que **tots** els estils de `Typography` utilitzin la mateixa font, es pot partir de la tipografia per defecte de Material 3 i sobreescriure únicament el `fontFamily` de cadascun amb `.copy(fontFamily = Montserrat)`, en lloc de redefinir els 15 estils sencers.

## 7. Formes (`Shapes`)

L'últim subsistema és `Shapes`: cinc mides estàndard (`extraSmall`, `small`, `medium`, `large`, `extraLarge`) que determinen l'arrodoniment de cantonades de components com `Card`, `Button` o `FloatingActionButton`:

```kotlin
val meuaShapes = Shapes(
    extraSmall = RoundedCornerShape(4.dp),
    small = RoundedCornerShape(8.dp),
    medium = RoundedCornerShape(12.dp),
    large = RoundedCornerShape(16.dp),
    extraLarge = RoundedCornerShape(24.dp)
)
```

```kotlin
Card(shape = MaterialTheme.shapes.medium) { /* contingut */ }
FloatingActionButton(shape = MaterialTheme.shapes.large, onClick = { }) { /* contingut */ }
```

## 8. Mode fosc

El composable arrel del tema rep normalment un paràmetre `darkTheme` amb valor per defecte `isSystemInDarkTheme()`, que consulta la preferència del sistema, i tria l'esquema de colors corresponent:

```kotlin
@Composable
fun MeuaAppTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    content: @Composable () -> Unit
) {
    val colorScheme = if (darkTheme) DarkColorScheme else LightColorScheme

    MaterialTheme(
        colorScheme = colorScheme,
        content = content
    )
}
```

A partir d'Android 12 (API 31) es pot anar més enllà amb el **color dinàmic**, que genera l'esquema de colors a partir del fons de pantalla de l'usuari:

```kotlin
val dynamicColor = Build.VERSION.SDK_INT >= Build.VERSION_CODES.S
val colorScheme = when {
    dynamicColor && darkTheme -> dynamicDarkColorScheme(LocalContext.current)
    dynamicColor && !darkTheme -> dynamicLightColorScheme(LocalContext.current)
    darkTheme -> DarkColorScheme
    else -> LightColorScheme
}
```

Com que tota la UI llegeix els colors sempre via `MaterialTheme.colorScheme`, i mai amb un valor fix, canviar entre mode clar i fosc (o activar el color dinàmic) no requereix tocar cap pantalla: n'hi ha prou canviant quin `ColorScheme` es passa a `MaterialTheme` a l'arrel de l'app.

!!! info "I si vull anar més enllà de MaterialTheme?"
    `MaterialTheme` cobreix el cas estàndard d'una app basada en Material Design. Quan es vol crear un sistema de disseny propi, amb estils reutilitzables per a components personalitzats, Compose ofereix una eina complementària: vegeu [Styles API](./stylesapi.md).
