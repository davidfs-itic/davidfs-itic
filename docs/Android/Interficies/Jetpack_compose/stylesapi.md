# Styles API

La Styles API és una eina experimental de Jetpack Compose per definir l'aparença d'un component (mides, colors, formes, ombres, tipografia...) de manera declarativa i reutilitzable, agrupant en un sol objecte `Style` propietats que fins ara calia repetir com a paràmetres solts o com a cadenes de `Modifier`. No substitueix els `Modifier` (que continuen sent l'eina per a interaccions i dibuix personalitzat), sinó que substitueix els **paràmetres d'estil** dispersos (`color =`, `padding =`, `shape =`...) per un únic bloc coherent, semblant a un `style` de CSS.

Documentació oficial: https://developer.android.com/develop/ui/compose/styles

Més info: https://www.youtube.com/watch?v=3oJs2i_os1Y

!!! warning "API experimental"
    La Styles API és `@Experimental` i encara pot canviar. Requereix una versió alpha de Compose (per exemple `androidx.compose.foundation:foundation:1.12.0-alpha03` o posterior) i, de moment, els components estàndard de Material **no** l'admeten directament: la seva utilitat pràctica actual és sobretot per a components propis d'un sistema de disseny personalitzat, no com a substitut de [MaterialTheme](./theming.md) en una app corrent.

## 1. Per què una nova API si ja hi ha `Modifier`?

Definir l'aparença d'un component sol implicar dues coses barrejades: paràmetres individuals (`color = MaterialTheme.colorScheme.primary`, `shape = RoundedCornerShape(8.dp)`) i cadenes de `Modifier` (`.padding(16.dp).background(Color.Blue)`). Cap dels dos mecanismes és pensat per compartir-se ni combinar-se fàcilment entre components diferents.

`Style` agrupa aquestes propietats en un sol objecte reutilitzable:

```kotlin
val estilBoto = Style {
    background(Color.Blue)
    contentPadding(16.dp)
    shape(RoundedCornerShape(8.dp))
}
```

A més d'oferir reutilització, els `Style` s'executen a les fases de *Layout* i *Draw*, sense passar per la fase de *Composició*, cosa que en redueix el cost respecte a canviar paràmetres normals (que sí que provoquen recomposició).

## 2. Aplicar un `Style`

Hi ha dues maneres d'aplicar un `Style` a un composable, segons si aquest exposa o no un paràmetre `style`:

**a) Component amb paràmetre `style`** (típic en components d'un sistema de disseny propi):

```kotlin
BaseButton(
    onClick = { },
    style = estilBoto
) {
    BaseText("Fes clic")
}
```

**b) `Modifier.styleable`**, per a qualsevol composable que no exposi `style` (per exemple, un `Row` o un `Box` normal):

```kotlin
Row(
    modifier = Modifier.styleable {
        background(Color.Blue)
        contentPadding(16.dp)
    }
) {
    BaseText("Contingut")
}
```

## 3. Propietats disponibles

Un `Style` agrupa propietats de quatre grans blocs:

- **Mida i posició**: `contentPadding()`, `externalPadding()`, `width()`, `height()`, `size()`, `fillWidth()`, `fillHeight()`...
- **Aparença visual**: `background()`, `foreground()`, `borderWidth()`, `borderColor()`, `shape()`, `dropShadow()`, `innerShadow()`.
- **Transformacions**: `scaleX()`/`scaleY()`, `rotationZ()`, `translationX()`/`translationY()`, `alpha()`.
- **Tipografia (heretable pels fills)**: `textStyle()`, `fontSize()`, `fontWeight()`, `contentColor()` (també s'aplica a icones), `lineHeight()`, `textAlign()`...

Dins d'un mateix `Style`, les propietats **no són additives**: si se'n defineix la mateixa dues vegades, guanya l'última:

```kotlin
val estil = Style {
    background(Color.Red)
    background(Color.Teal)       // sobreescriu el vermell
    contentPadding(64.dp)
    contentPaddingTop(16.dp)     // sobreescriu només el padding superior
}
```

Aquest comportament és diferent del d'un `Modifier`, on cada crida s'afegeix a les anteriors en lloc de sobreescriure-les.

## 4. Combinar estils amb `then`

Diversos `Style` es poden combinar amb l'operador `then`, igual que es fa amb `Modifier`:

```kotlin
val paddingAtomic = Style { contentPadding(16.dp) }
val cantonadesAtomic = Style { shape(RoundedCornerShape(8.dp)) }
val fonsAtomic = Style { background(Color.Blue) }

val estilBoto = paddingAtomic then cantonadesAtomic then fonsAtomic
```

Aquest patró d'**estils atòmics** (petits, d'una sola responsabilitat, combinats amb `then`) és el recomanat per a un sistema de disseny, en lloc d'un `Style` monolític amb totes les propietats juntes: permet reutilitzar `paddingAtomic` o `cantonadesAtomic` en altres components sense repetir codi.

## 5. Estats: `hovered`, `pressed`, `focused`, `disabled`

Un dels punts forts de la Styles API és definir com canvia l'aparença d'un component segons el seu estat d'interacció, sense haver de gestionar aquest estat a mà (a diferència del que cal fer amb [InteractionSource](./interactionsource.md) i `Modifier` normals):

```kotlin
val estilAmbOmbra = Style {
    hovered {
        animate {
            dropShadow(
                Shadow(
                    offset = DpOffset(0.dp, 0.dp),
                    radius = 2.dp,
                    color = Color.Blue
                )
            )
        }
    }
}
```

El bloc `animate { ... }` anima automàticament la transició entre estats (per exemple, en aparèixer o desaparèixer l'ombra en passar el punter per sobre), sense necessitat d'`animateColorAsState` ni de gestionar l'estat manualment.

Per fer servir estats amb `Modifier.styleable` cal, a més, un `StyleState`, normalment construït a partir d'un `MutableInteractionSource`:

```kotlin
@Composable
fun BotoPersonalitzat(text: String, onClick: () -> Unit) {
    val interactionSource = remember { MutableInteractionSource() }
    val styleState = remember(interactionSource) { MutableStyleState(interactionSource) }

    Box(
        modifier = Modifier
            .clickable(interactionSource = interactionSource, indication = null, onClick = onClick)
            .styleable(styleState, estilAmbOmbra)
    ) {
        Text(text)
    }
}
```

## 6. Relació amb `MaterialTheme`

De moment, els components estàndard de Material (`Button`, `Card`, `TextField`...) no accepten un paràmetre `style`, i Material encara no ha adoptat oficialment la Styles API. Això vol dir que, avui dia, la Styles API té sentit sobretot en dos casos:

- Components **propis**, fora de Material, que formen part d'un sistema de disseny a mida (per exemple, exposant un paràmetre `style` a `BaseButton`, `BaseText`, etc.).
- Aplicar `Modifier.styleable` puntualment sobre composables bàsics (`Box`, `Row`, `Column`) que no depenen de Material.

Per a una app que utilitza els components estàndard de Material 3, la via recomanada continua sent [Theming amb MaterialTheme](./theming.md) (colors, tipografia i formes del tema), reservant la Styles API per a components personalitzats o per experimentar-hi mentre l'API madura.
