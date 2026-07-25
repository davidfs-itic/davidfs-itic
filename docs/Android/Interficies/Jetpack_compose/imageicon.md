# Els composables Image i Icon

`Image` i `Icon` són els dos composables per mostrar contingut gràfic a la pantalla. Encara que tots dos acaben dibuixant un gràfic, tenen propòsits diferents: `Image` mostra contingut visual (fotografies, il·lustracions), mentre que `Icon` mostra símbols petits i monocroms que representen accions o estats (com les icones de les barres d'eines o dels botons).

Documentació oficial: https://developer.android.com/develop/ui/compose/graphics/images/customize

## 1. Image

`Image` es fa servir per mostrar un recurs gràfic complet, respectant els seus colors originals. La font de la imatge sol ser un recurs local carregat amb `painterResource`:

```kotlin
Image(
    painter = painterResource(id = R.drawable.paisatge),
    contentDescription = "Fotografia d'un paisatge de muntanya",
    modifier = Modifier
        .fillMaxWidth()
        .height(200.dp),
    contentScale = ContentScale.Crop
)
```

- **`painter`**: descriu què s'ha de dibuixar. `painterResource` el crea a partir d'un recurs de la carpeta `res/drawable`, però `Image` també pot rebre directament un `ImageBitmap` o un `ImageVector`.
- **`contentDescription`**: descripció textual de la imatge per a lectors de pantalla (vegeu el punt 4). Si la imatge és purament decorativa i no aporta informació, s'ha de passar explícitament `null`.
- **`contentScale`**: com s'ajusta la imatge quan la seva mida original no coincideix amb l'espai disponible. Els valors més habituals:
    - `ContentScale.Crop`: omple tot l'espai disponible, retallant l'excedent si cal (manté la proporció).
    - `ContentScale.Fit`: mostra la imatge sencera dins l'espai disponible, sense retallar-la (pot deixar espai buit).
    - `ContentScale.FillBounds`: estira la imatge perquè ocupi exactament l'espai, sense respectar la proporció original (pot deformar-la).

## 2. Icon

`Icon` és una versió simplificada d'`Image`, pensada específicament per a icones d'un sol color (*tint*). En comptes de mostrar els colors propis del recurs, `Icon` dibuixa la forma de la icona amb el color que se li indiqui, cosa que permet que s'adapti automàticament al tema (per exemple, canviant de color en mode fosc):

```kotlin
Icon(
    imageVector = Icons.Default.Favorite,
    contentDescription = "Afegir a preferits",
    tint = Color.Red,
    modifier = Modifier.size(24.dp)
)
```

- **`imageVector`**: la majoria d'icones es fan servir a partir del paquet `Icons` (`Icons.Default`, `Icons.Filled`, `Icons.Outlined`, `Icons.Rounded`...), que proporciona centenars d'icones estàndard de Material Design ja preparades per fer servir, sense necessitat d'afegir cap recurs `drawable` propi.
- **`tint`**: color amb què es pinta la icona. Per defecte, si no s'indica, `Icon` fa servir el color de contingut del tema (`LocalContentColor`), cosa que sol ser el comportament desitjat dins de botons o barres.

## 3. Quan fer servir cada un

La distinció no és només tècnica, també és semàntica:

- **`Icon`**: per a símbols d'interfície (accions, estats, indicadors), normalment petits, que han de respectar el color del tema. Exemple: la icona d'una `IconButton`, la icona d'un `TextField`, l'estrella de "preferit".
- **`Image`**: per a contingut que forma part de la informació que es mostra a l'usuari: fotografies de perfil, il·lustracions, captures, logotips amb els seus propis colors.

Fer servir `Image` per a una icona d'acció (o a l'inrevés) sol donar problemes: una `Image` no s'adapta al canvi de tema perquè mostra sempre els seus propis colors, i una `Icon` no és adequada per a fotografies perquè només es pinta amb un sol color pla.

## 4. Accessibilitat: `contentDescription`

El paràmetre `contentDescription` és el text que un lector de pantalla (TalkBack) llegirà en veu alta quan l'usuari navegui fins a aquest element. És obligatori raonar-lo en cada cas:

- Si la imatge o icona **aporta informació** (una fotografia de producte, una icona sense text que representa una acció), cal descriure-la de manera breu i útil: `contentDescription = "Afegir a preferits"`.
- Si la imatge és **purament decorativa** i ja hi ha un text al costat que explica el mateix (per exemple, una icona seguida d'un `Text` amb la mateixa etiqueta), s'ha de passar `contentDescription = null` explícitament, perquè el lector de pantalla no la llegeixi dues vegades ni afegeixi soroll innecessari.

## 5. Imatges des d'una font remota

Ni `Image` ni `Icon` saben, per si sols, descarregar una imatge d'Internet: `painterResource` només funciona amb recursos locals de l'app. Per carregar imatges des d'una URL (per exemple, l'avatar d'un usuari des d'un servidor), cal una llibreria externa especialitzada, com **Coil**, que ofereix el seu propi composable `AsyncImage` amb una API molt semblant a `Image`:

```kotlin
AsyncImage(
    model = "https://exemple.cat/imatges/avatar.png",
    contentDescription = "Avatar de l'usuari",
    contentScale = ContentScale.Crop,
    modifier = Modifier.size(64.dp).clip(CircleShape)
)
```

Documentació oficial per càrrega d'imatges online: https://developer.android.com/develop/ui/compose/graphics/images/loading

Caldrà afegir alguna llibreria com glide o coil