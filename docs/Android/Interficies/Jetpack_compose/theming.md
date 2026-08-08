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

## 6. Formes (`Shapes`)

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

## 7. Mode fosc

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
