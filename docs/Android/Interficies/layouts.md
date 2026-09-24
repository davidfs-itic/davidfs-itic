# Layouts

Un layout defineix l'estructura visual d'una pantalla: quins elements hi ha (textos, botons, imatges...) i com es col·loquen els uns respecte als altres. En el sistema de Views, els layouts es declaren normalment en fitxers XML, separats del codi Kotlin, i després es carreguen des d'una Activity o un Fragment.

Aquest document recull els conceptes comuns a tots els layouts. Els dos layouts més utilitzats tenen el seu propi document:

- [LinearLayout](./linearlayout.md)
- [ConstraintLayout i Chains](./constraintlayout.md)

Documentació oficial: [https://developer.android.com/develop/ui/views/layout/declaring-layout](https://developer.android.com/develop/ui/views/layout/declaring-layout)

## 1. View i ViewGroup

Tots els elements d'una interfície amb Views són objectes de dues classes base:

- **`View`**: un element que es dibuixa a la pantalla i amb el qual l'usuari pot interactuar. Per exemple `TextView`, `Button`, `ImageView` o `EditText`.
- **`ViewGroup`**: un contenidor invisible que conté altres `View` (o altres `ViewGroup`) i decideix on es col·loquen. Els layouts (`LinearLayout`, `ConstraintLayout`, `FrameLayout`...) són subclasses de `ViewGroup`.

Això forma una **jerarquia de vistes** en forma d'arbre: un `ViewGroup` arrel que conté fills, que al seu torn poden ser altres `ViewGroup` amb més fills.

```
ConstraintLayout (arrel)
├── TextView
├── LinearLayout
│   ├── EditText
│   └── Button
└── ImageView
```

## 2. Fitxers de layout

Els layouts es guarden a la carpeta `res/layout/` del projecte, amb noms en minúscules i guions baixos (per exemple `activity_main.xml`, `fragment_login.xml`, `item_producte.xml`).

Un fitxer de layout té **un únic element arrel**, que ha de declarar el namespace `android`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

</androidx.constraintlayout.widget.ConstraintLayout>
```

Per mostrar el layout, l'Activity el carrega amb `setContentView()`, indicant-li el recurs del layout (`R.layout.nom_del_fitxer`). Un cop carregat, es pot obtenir qualsevol vista del layout a partir del seu `id` amb `findViewById()`.

Per exemple, si el layout contingués un `TextView` amb `android:id="@+id/txtTitol"`:

```kotlin
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val txtTitol = findViewById<TextView>(R.id.txtTitol)
        txtTitol.text = "Benvinguts"
    }
}
```

!!! warning "Cridar `findViewById()` després de `setContentView()`"
    `findViewById()` busca la vista dins del layout que s'ha carregat. Si es crida abans de `setContentView()`, encara no hi ha cap layout i retorna `null`, cosa que provoca un error en executar l'app.

!!! info "Edge-to-edge"
    Les plantilles noves d'Android Studio criden `enableEdgeToEdge()` i apliquen uns paddings a la vista arrel (amb `id` `main`) perquè el contingut no quedi sota les barres del sistema. Vegeu [Edge-to-edge i System Bars](./layoutactivities.md).

### Namespaces `app` i `tools`

A més de `android`, sovint apareixen dos namespaces més a l'element arrel:

```xml
xmlns:app="http://schemas.android.com/apk/res-auto"
xmlns:tools="http://schemas.android.com/tools"
```

- **`app:`**: atributs definits per llibreries (AndroidX, Material...) i no pel sistema. Per exemple, tots els atributs de `ConstraintLayout` (`app:layout_constraintTop_toTopOf`...).
- **`tools:`**: atributs que només fa servir l'editor d'Android Studio i que **no arriben a l'aplicació**. Són útils per veure dades d'exemple a la vista de disseny sense posar-les al layout real:

```xml
<TextView
    android:id="@+id/txtNom"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    tools:text="Nom d'exemple" />
```

## 3. Atributs comuns

### Identificador (`id`)

L'atribut `android:id` dona un nom a la vista per poder-hi accedir des del codi (amb ViewBinding o `findViewById`) i perquè altres vistes s'hi puguin referenciar (per exemple, en un `ConstraintLayout`).

```xml
android:id="@+id/btnEnviar"
```

El `+` indica que es crea un identificador nou. Quan es fa referència a un `id` ja existent, s'escriu sense el `+`: `@id/btnEnviar`.

!!! warning "Convenció de noms"
    ViewBinding converteix els `id` a *camelCase* (`btn_enviar` passa a ser `binding.btnEnviar`). Tant és fer servir `btn_enviar` com `btnEnviar`, però cal ser coherent en tot el projecte.

### Amplada i alçada

Totes les vistes **han de tenir** `android:layout_width` i `android:layout_height`. Els valors possibles són:

| Valor | Significat |
|---|---|
| `match_parent` | La vista ocupa tot l'espai disponible del pare en aquella dimensió. |
| `wrap_content` | La vista ocupa només l'espai que necessita el seu contingut. |
| `120dp` | Mida fixa. |
| `0dp` | Mida calculada pel pare: amb pesos al `LinearLayout` o amb restriccions al `ConstraintLayout`. |

!!! info "`fill_parent`"
    En codi antic es pot trobar `fill_parent`. És el nom antic de `match_parent` i està obsolet.

### Unitats de mesura

- **`dp`** (*density-independent pixels*): unitat per a mides, marges i paddings. Un `dp` ocupa aproximadament el mateix espai físic en qualsevol pantalla, independentment de la seva densitat de píxels.
- **`sp`** (*scale-independent pixels*): com `dp`, però a més s'escala segons la mida de text que l'usuari ha triat a la configuració del dispositiu. S'utilitza **només per a mides de text** (`android:textSize`).
- **`px`**: píxels reals de la pantalla. No s'ha de fer servir, perquè el resultat canvia molt d'un dispositiu a un altre.

```xml
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:padding="16dp"
    android:textSize="18sp"
    android:text="Text amb mida accessible" />
```

## 4. Padding i margin

Tots dos creen espai, però en llocs diferents:

- **`padding`**: espai **interior**, entre la vora de la vista i el seu contingut. Forma part de la vista (per exemple, el color de fons també s'hi pinta).
- **`layout_margin`**: espai **exterior**, entre la vora de la vista i els elements que l'envolten. El gestiona el layout pare.

```
┌────────────── margin ──────────────┐
│  ┌─────────── vora ─────────────┐  │
│  │  ┌──────── padding ───────┐  │  │
│  │  │        contingut       │  │  │
│  │  └────────────────────────┘  │  │
│  └──────────────────────────────┘  │
└────────────────────────────────────┘
```

Es poden indicar per a tots els costats alhora o per a cadascun per separat:

```xml
android:padding="16dp"
android:paddingHorizontal="16dp"
android:paddingVertical="8dp"
android:paddingStart="16dp"

android:layout_margin="8dp"
android:layout_marginTop="24dp"
android:layout_marginEnd="16dp"
```

!!! info "`start`/`end` en lloc de `left`/`right`"
    És preferible fer servir `Start` i `End` en lloc de `Left` i `Right`. En idiomes que s'escriuen de dreta a esquerra (àrab, hebreu), Android intercanvia automàticament `start` i `end`, i la interfície es veu correctament sense haver de fer cap canvi.

## 5. Gravity i layout_gravity

Són dos atributs que es confonen sovint:

- **`android:gravity`**: com es col·loca el **contingut dins de la vista**. Per exemple, el text dins d'un `TextView`, o els fills dins d'un `LinearLayout`.
- **`android:layout_gravity`**: com es col·loca la **vista dins del seu pare**. Només té efecte si el pare el suporta (`LinearLayout` i `FrameLayout`). En un `ConstraintLayout` no fa res.

```xml
<TextView
    android:layout_width="match_parent"
    android:layout_height="100dp"
    android:gravity="center"
    android:text="Text centrat dins del TextView" />
```

Els valors es poden combinar amb `|`: `android:gravity="center_vertical|end"`.

L'ús de `layout_gravity` s'explica amb detall a [LinearLayout](./linearlayout.md#4-gravity-i-layout_gravity).

## 6. Atributs amb prefix `layout_`

Els atributs d'una vista es divideixen en dos grups, i el nom indica a quin pertanyen:

- **Atributs que comencen per `layout_`**: són instruccions per al **pare**. Indiquen al layout que conté la vista com l'ha de col·locar i quina mida li ha de donar. Per exemple `layout_width`, `layout_height`, `layout_margin` o `layout_gravity`.
- **Atributs sense `layout_`**: són propietats de la **vista mateixa**. Per exemple `text`, `textSize`, `padding`, `gravity` o `background`.

Com que els atributs `layout_` els interpreta el pare, cada tipus de layout accepta els seus. `layout_weight` només té efecte dins d'un `LinearLayout`, i els atributs `layout_constraint...` només dins d'un `ConstraintLayout`. Si es posen en una vista que té un altre tipus de pare, s'ignoren.

Aquesta regla explica la diferència entre les parelles d'atributs que s'han vist a les seccions anteriors ([Padding i margin](#4-padding-i-margin) i [Gravity i layout_gravity](#5-gravity-i-layout_gravity)), i també entre `layout_height` i `height`:

| Atribut | Qui el fa servir | Significat |
|---|---|---|
| `layout_margin` | El pare | Espai **fora** de la vista, respecte als elements del voltant. |
| `padding` | La vista | Espai **dins** de la vista, entre la vora i el contingut. |
| `layout_gravity` | El pare | On es col·loca la vista dins del pare. |
| `gravity` | La vista | On es col·loca el contingut dins de la vista. |
| `layout_height` | El pare | L'alçada que el pare dona a la vista. Obligatori a totes les vistes. |
| `height` | La vista | L'alçada exacta d'un `TextView` (i subclasses com `Button` o `EditText`). Només té efecte si `layout_height` és `wrap_content`. |

```xml
<!-- Fa 100dp d'alt: amb wrap_content el TextView decideix la seva alçada, i height la fixa -->
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:height="100dp"
    android:text="Hola" />

<!-- Fa 50dp d'alt: layout_height fixa la mida i height s'ignora -->
<TextView
    android:layout_width="wrap_content"
    android:layout_height="50dp"
    android:height="100dp"
    android:text="Hola" />
```

A la pràctica, `android:height` i `android:width` gairebé no es fan servir: una mida fixa es posa directament a `layout_height` o `layout_width`. Si es vol una mida mínima, hi ha `android:minHeight` i `android:minWidth`, que funcionen a qualsevol vista.

## 7. Visibilitat

L'atribut `android:visibility` controla si una vista es mostra:

| Valor | Es veu? | Ocupa espai? |
|---|---|---|
| `visible` | Sí | Sí |
| `invisible` | No | Sí (deixa el forat) |
| `gone` | No | No (la resta d'elements es recol·loquen) |

Des del codi es canvia amb la propietat `visibility`, o amb les funcions d'extensió de `core-ktx`:

```kotlin
val progressBar = findViewById<ProgressBar>(R.id.progressBar)
progressBar.visibility = View.GONE

// Equivalent amb core-ktx
progressBar.isVisible = false
```

## 8. Tipus de layouts

| Layout | Ús |
|---|---|
| `LinearLayout` | Col·loca els fills en una sola fila o columna. Senzill i ideal per a formularis i llistes curtes d'elements. Vegeu [LinearLayout](./linearlayout.md). |
| `ConstraintLayout` | Posiciona cada fill amb restriccions respecte al pare o a altres vistes. Permet interfícies complexes sense niar layouts. És el layout per defecte de les plantilles d'Android Studio. Vegeu [ConstraintLayout i Chains](./constraintlayout.md). |
| `FrameLayout` | Apila els fills un sobre l'altre. S'utilitza com a contenidor d'un sol element (per exemple, per als Fragments) o per superposar vistes. |
| `ScrollView` / `NestedScrollView` | Permet desplaçar un contingut més gran que la pantalla. Només pot tenir **un fill directe** (normalment un `LinearLayout`). |
| `RelativeLayout` | Posiciona els fills respecte als altres. Està en desús: `ConstraintLayout` fa el mateix amb més possibilitats. |

Per a llistes llargues o dinàmiques no es fa servir un `ScrollView` amb molts elements, sinó un [RecyclerView](./recyclerview.md).

### Equivalències amb Jetpack Compose

| Views (XML) | Jetpack Compose |
|---|---|
| `LinearLayout` vertical | `Column` |
| `LinearLayout` horitzontal | `Row` |
| `FrameLayout` | `Box` |
| `ConstraintLayout` | `ConstraintLayout` (compose) |

Vegeu [Layouts en Jetpack Compose](./Jetpack_compose/layouts.md).

## 9. Rendiment: evitar el niuament excessiu

Cada `ViewGroup` que s'afegeix a la jerarquia s'ha de mesurar i col·locar. Si es nien molts layouts (un `LinearLayout` dins d'un altre, dins d'un altre...), la pantalla triga més a dibuixar-se, sobretot si s'utilitzen pesos (`layout_weight`), que obliguen a mesurar els fills dues vegades.

Recomanacions:

- Mantenir la jerarquia tan plana com sigui possible.
- Per a pantalles complexes, fer servir un `ConstraintLayout` en lloc de nivells de `LinearLayout`.
- Reutilitzar fragments de layout repetits amb `<include>`.

### Reutilitzar layouts amb `<include>`

Si un bloc d'interfície es repeteix en diverses pantalles (una capçalera, un peu...), es pot definir en un fitxer propi i incloure'l:

```xml
<!-- res/layout/capcalera.xml -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal"
    android:padding="16dp">

    <ImageView
        android:layout_width="40dp"
        android:layout_height="40dp"
        android:src="@mipmap/ic_launcher" />

    <TextView
        android:id="@+id/txtNomApp"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginStart="12dp"
        android:text="@string/app_name"
        android:textSize="20sp" />
</LinearLayout>
```

```xml
<!-- A qualsevol altre layout -->
<include
    android:id="@+id/capcalera"
    layout="@layout/capcalera" />
```

Les vistes del layout inclòs s'obtenen igual que la resta, amb `findViewById()`, perquè passen a formar part de la jerarquia del layout que les inclou:

```kotlin
val txtNomApp = findViewById<TextView>(R.id.txtNomApp)
```

## 10. L'editor de layouts d'Android Studio

En obrir un fitxer de layout, Android Studio ofereix tres modes (a la cantonada superior dreta):

- **Code**: només l'XML.
- **Split**: l'XML i la previsualització alhora. És el mode més recomanable per aprendre, perquè es veu l'efecte de cada atribut.
- **Design**: editor visual amb la paleta de components, l'arbre de components (*Component Tree*) i el panell d'atributs.

!!! info "Textos a `strings.xml`"
    Els textos no s'haurien d'escriure directament al layout (`android:text="Enviar"`), sinó a `res/values/strings.xml` i referenciar-los (`android:text="@string/enviar"`). Així es poden traduir a altres idiomes. Android Studio avisa d'aquest cas amb un *warning* de *hardcoded string*.
