# ConstraintLayout i Chains

`ConstraintLayout` és un `ViewGroup` que posiciona cada fill mitjançant **restriccions** (*constraints*): regles que lliguen una vora de la vista a una vora del pare o d'una altra vista. Per exemple: "la part superior d'aquest botó va 16dp per sota del text del títol" o "aquesta imatge està centrada horitzontalment a la pantalla".

El seu gran avantatge és que permet construir interfícies complexes amb **una jerarquia plana**, sense haver de niar layouts. Per això és el layout que fan servir per defecte les plantilles d'Android Studio.

Els conceptes comuns a tots els layouts (`layout_width`, `dp`, padding, margin...) estan explicats a [Layouts](./layouts.md).

Documentació oficial: [https://developer.android.com/develop/ui/views/layout/constraint-layout](https://developer.android.com/develop/ui/views/layout/constraint-layout)

## 1. Dependència

`ConstraintLayout` forma part d'AndroidX i no del sistema. Els projectes nous ja inclouen la dependència; si no hi és, cal afegir-la al `build.gradle.kts` del mòdul:

```kotlin
dependencies {
    implementation("androidx.constraintlayout:constraintlayout:2.2.1")
}
```

Tots els atributs de restricció fan servir el namespace `app:`, que cal declarar a l'element arrel:

```xml
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <!-- fills -->

</androidx.constraintlayout.widget.ConstraintLayout>
```

## 2. Restriccions bàsiques

Cada vista té quatre vores principals: `Top`, `Bottom`, `Start` i `End` (més `Baseline`, la línia base del text). Una restricció lliga una vora de la vista a una vora d'un altre element, amb atributs que segueixen el patró:

```
app:layout_constraint<VoraPròpia>_to<VoraDestí>Of="<destí>"
```

El destí pot ser `parent` o l'`id` d'una altra vista del mateix `ConstraintLayout`.

```xml
<TextView
    android:id="@+id/txtTitol"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_marginTop="32dp"
    android:text="Títol"
    android:textSize="24sp"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toTopOf="parent"
    android:layout_marginStart="16dp" />

<TextView
    android:id="@+id/txtSubtitol"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_marginTop="8dp"
    android:text="Subtítol"
    app:layout_constraintStart_toStartOf="@id/txtTitol"
    app:layout_constraintTop_toBottomOf="@id/txtTitol" />
```

- `txtTitol`: la seva part superior està lligada a la part superior del pare (amb 32dp de marge), i el seu inici a l'inici del pare (amb 16dp de marge).
- `txtSubtitol`: la seva part superior està lligada a la part **inferior** del títol, i el seu inici està alineat amb l'inici del títol.

!!! warning "Cada vista necessita una restricció horitzontal i una de vertical"
    Si una vista no té cap restricció horitzontal, es col·loca a l'esquerra del tot; si no en té cap de vertical, a dalt de tot. A l'editor de disseny pot semblar que està ben col·locada (perquè l'editor la mostra on l'has deixat anar), però en executar l'app apareixerà a la cantonada. Android Studio avisa d'aquest error amb el missatge *This view is not constrained*.

### Margins

Els marges (`android:layout_marginTop`, `layout_marginStart`...) només tenen efecte en la direcció en què hi ha una restricció. Un `layout_marginTop` en una vista sense restricció `Top` s'ignora.


### Alineació de textos: Baseline

Per alinear dos textos de mides diferents per la línia on s'escriuen les lletres (i no per la vora de la vista), es fa servir `Baseline`:

```xml
app:layout_constraintBaseline_toBaselineOf="@id/txtTitol"
```

## 3. Centrar i bias

Si una vista té restriccions **a totes dues bandes** d'un eix (per exemple `Start` i `End`), el `ConstraintLayout` la centra entre elles, com si dues molles tiressin de la vista amb la mateixa força:

```xml
<Button
    android:id="@+id/btnCentrat"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Centrat"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toTopOf="parent" />
```

Aquest botó queda al centre exacte de la pantalla.

El **bias** permet desplaçar la vista cap a una de les dues bandes. Va de `0` (tot a l'inici o a dalt) a `1` (tot al final o a baix); el valor per defecte és `0.5`:

```xml
app:layout_constraintHorizontal_bias="0.25"
app:layout_constraintVertical_bias="0.8"
```

Amb aquests valors, el botó queda al 25% de l'amplada i al 80% de l'alçada disponibles.

## 4. Mida de les vistes

Els valors generals de `layout_width` i `layout_height` (`wrap_content`, mida fixa...) funcionen igual que a qualsevol layout (vegeu [Amplada i alçada](./layouts.md#amplada-i-alcada)). En un `ConstraintLayout`, però, hi ha dos valors amb un comportament propi:

| Valor | Comportament |
|---|---|
| `0dp` (*match constraint*) | La vista s'estira per ocupar tot l'espai entre les seves restriccions. |
| `match_parent` | **No s'ha de fer servir** dins d'un `ConstraintLayout`. S'ha de substituir per `0dp` amb restriccions a totes dues bandes. |

Exemple d'un `EditText` que ocupa tota l'amplada amb 16dp de marge a cada banda:

```xml
<EditText
    android:id="@+id/edtNom"
    android:layout_width="0dp"
    android:layout_height="wrap_content"
    android:layout_marginStart="16dp"
    android:layout_marginEnd="16dp"
    android:hint="Nom"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toBottomOf="@id/txtSubtitol" />
```

!!! warning "`0dp` depèn del layout pare"
    `0dp` no vol dir "omple l'espai", sinó "no tinc mida, que la calculi el pare". En un `ConstraintLayout`, la mida la calculen les restriccions, i per això cal que la vista estigui lligada **a les dues bandes** de l'eix (`Start` i `End` per a l'amplada, `Top` i `Bottom` per a l'alçada). Si només té una restricció, no hi ha cap espai entre restriccions per omplir i la vista es comporta com amb `wrap_content`.

    Aquí no cal `layout_weight`, i de fet no hi té cap efecte: és un atribut del `LinearLayout`. En un `LinearLayout`, en canvi, un fill amb `0dp` sense pes fa literalment 0 i no es veu (vegeu [LinearLayout: Un element que omple l'espai restant](./linearlayout.md#un-element-que-omple-lespai-restant)). L'únic cas en què un `ConstraintLayout` fa servir pesos és dins d'una chain (vegeu [Chains amb pes](#chains-amb-pes-weighted-chain)).

### Textos llargs amb wrap_content

Una vista amb `wrap_content` pot créixer més enllà de les seves restriccions si el contingut és molt llarg (per exemple, un text que no cap). Per evitar-ho i fer que el text salti de línia dins l'espai permès:

```xml
android:layout_width="wrap_content"
app:layout_constrainedWidth="true"
```

### Proporció (dimensionRatio)

Si una de les dues dimensions és `0dp`, es pot calcular a partir de l'altra amb una proporció. Molt útil per a imatges:

```xml
<ImageView
    android:id="@+id/imgPortada"
    android:layout_width="0dp"
    android:layout_height="0dp"
    android:scaleType="centerCrop"
    android:src="@drawable/portada"
    app:layout_constraintDimensionRatio="16:9"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toTopOf="parent" />
```

La imatge ocupa tota l'amplada i la seva alçada es calcula perquè mantingui la proporció 16:9.

### Percentatges

Amb `0dp` també es pot indicar la mida com un percentatge de l'espai del pare:

```xml
android:layout_width="0dp"
app:layout_constraintWidth_percent="0.6"
```

## 5. Guidelines

Una `Guideline` és una línia invisible, horitzontal o vertical, que serveix com a punt de referència per a altres restriccions. No es mostra a l'aplicació.

La seva posició es pot indicar amb una distància des de l'inici (`layout_constraintGuide_begin`), des del final (`layout_constraintGuide_end`) o en percentatge (`layout_constraintGuide_percent`).

```xml
<androidx.constraintlayout.widget.Guideline
    android:id="@+id/guideMeitat"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    app:layout_constraintGuide_percent="0.5" />

<Button
    android:id="@+id/btnEsquerra"
    android:layout_width="0dp"
    android:layout_height="wrap_content"
    android:text="Esquerra"
    app:layout_constraintEnd_toStartOf="@id/guideMeitat"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toTopOf="parent" />

<Button
    android:id="@+id/btnDreta"
    android:layout_width="0dp"
    android:layout_height="wrap_content"
    android:text="Dreta"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toEndOf="@id/guideMeitat"
    app:layout_constraintTop_toTopOf="parent" />
```

Cada botó ocupa la meitat de l'amplada: un a l'esquerra i l'altre a la dreta de la guia vertical situada al 50%.

## 6. Barriers

Una `Barrier` també és una línia invisible, però la seva posició no és fixa: es col·loca automàticament a la vora de la vista **més gran** d'un grup. És útil quan les mides de les vistes depenen del contingut (per exemple, textos traduïts a idiomes diferents).

Cas típic: un formulari amb etiquetes a l'esquerra i camps a la dreta. Els camps han de començar just després de l'etiqueta més llarga, sigui quina sigui:

```xml
<TextView
    android:id="@+id/lblNom"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Nom:"
    app:layout_constraintBaseline_toBaselineOf="@id/edtNomForm"
    app:layout_constraintStart_toStartOf="parent" />

<TextView
    android:id="@+id/lblCorreu"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Correu electrònic:"
    app:layout_constraintBaseline_toBaselineOf="@id/edtCorreuForm"
    app:layout_constraintStart_toStartOf="parent" />

<androidx.constraintlayout.widget.Barrier
    android:id="@+id/barrierEtiquetes"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:barrierDirection="end"
    app:constraint_referenced_ids="lblNom,lblCorreu" />

<EditText
    android:id="@+id/edtNomForm"
    android:layout_width="0dp"
    android:layout_height="wrap_content"
    android:layout_marginStart="8dp"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toEndOf="@id/barrierEtiquetes"
    app:layout_constraintTop_toTopOf="parent" />

<EditText
    android:id="@+id/edtCorreuForm"
    android:layout_width="0dp"
    android:layout_height="wrap_content"
    android:layout_marginStart="8dp"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toEndOf="@id/barrierEtiquetes"
    app:layout_constraintTop_toBottomOf="@id/edtNomForm" />
```

- `barrierDirection="end"`: la barrera es col·loca al final (dreta) de l'etiqueta més ampla.
- `constraint_referenced_ids`: les vistes que controlen la posició de la barrera, separades per comes i **sense** `@id/`.

## 7. Group

Un `Group` permet canviar la visibilitat de diverses vistes alhora, sense que deixin de ser fills directes del `ConstraintLayout`:

```xml
<androidx.constraintlayout.widget.Group
    android:id="@+id/grupFormulari"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:constraint_referenced_ids="lblNom,lblCorreu,edtNomForm,edtCorreuForm" />
```

```kotlin
// Amaga totes les vistes del grup
binding.grupFormulari.isVisible = false
```

## 8. Chains

Una **chain** (cadena) és un grup de vistes lligades **entre elles en totes dues direccions** al llarg d'un eix. El `ConstraintLayout` tracta la cadena com un bloc i reparteix l'espai entre els seus elements segons l'estil de la cadena. És l'equivalent, dins d'un `ConstraintLayout`, del que es fa amb un [LinearLayout](./linearlayout.md), però sense niar layouts.

### Crear una chain

Perquè hi hagi una cadena horitzontal entre A, B i C, cal que:

- A estigui lligada al pare per l'inici i a B pel final.
- B estigui lligada a A per l'inici i a C pel final.
- C estigui lligada a B per l'inici i al pare pel final.

```
parent ◄── A ◄──► B ◄──► C ──► parent
```

```xml
<Button
    android:id="@+id/btnA"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="A"
    app:layout_constraintEnd_toStartOf="@id/btnB"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toTopOf="parent" />

<Button
    android:id="@+id/btnB"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="B"
    app:layout_constraintEnd_toStartOf="@id/btnC"
    app:layout_constraintStart_toEndOf="@id/btnA"
    app:layout_constraintTop_toTopOf="parent" />

<Button
    android:id="@+id/btnC"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="C"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toEndOf="@id/btnB"
    app:layout_constraintTop_toTopOf="parent" />
```

!!! info "Crear chains des de l'editor"
    A la vista **Design** d'Android Studio, es poden seleccionar diverses vistes, fer clic amb el botó dret i triar **Chains > Create Horizontal Chain** (o *Vertical*). L'editor afegeix automàticament totes les restriccions bidireccionals.

### Cap de la cadena (head)

El primer element de la cadena (el de més a l'esquerra en una horitzontal, o el de més amunt en una vertical) és el **cap** (*head*). Els atributs que configuren tota la cadena, com l'estil, **només es posen al cap**.

### Estils de chain

L'estil s'indica al cap de la cadena amb `app:layout_constraintHorizontal_chainStyle` (o `Vertical_chainStyle`):

```xml
<Button
    android:id="@+id/btnA"
    ...
    app:layout_constraintHorizontal_chainStyle="spread_inside" />
```

Els valors possibles són:

| Estil | Comportament |
|---|---|
| `spread` | Valor per defecte. Reparteix l'espai de manera igual abans, entre i després dels elements. |
| `spread_inside` | Els elements dels extrems queden enganxats a les vores i l'espai es reparteix només entre els elements. |
| `packed` | Tots els elements queden junts al centre. La posició del bloc es pot desplaçar amb el bias del cap. |

```
spread:         |   A    B    C   |
spread_inside:  |A      B       C|
packed:         |      ABC       |
```

Amb `packed`, el bias del cap desplaça tot el bloc. Per exemple, per tenir els tres botons junts i alineats a l'esquerra:

```xml
app:layout_constraintHorizontal_chainStyle="packed"
app:layout_constraintHorizontal_bias="0"
```

### Chains amb pes (weighted chain)

Si els elements de la cadena tenen `0dp` en l'eix de la cadena, s'estiren per omplir tot l'espai, igual que amb `layout_weight` a un `LinearLayout`. Per defecte es reparteixen l'espai a parts iguals; amb `layout_constraintHorizontal_weight` (o `Vertical_weight`) s'indica la proporció:

```xml
<EditText
    android:id="@+id/edtCerca"
    android:layout_width="0dp"
    android:layout_height="wrap_content"
    android:hint="Cerca..."
    app:layout_constraintEnd_toStartOf="@id/btnCerca"
    app:layout_constraintHorizontal_weight="3"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toTopOf="parent" />

<Button
    android:id="@+id/btnCerca"
    android:layout_width="0dp"
    android:layout_height="wrap_content"
    android:text="OK"
    app:layout_constraintBaseline_toBaselineOf="@id/edtCerca"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintHorizontal_weight="1"
    app:layout_constraintStart_toEndOf="@id/edtCerca" />
```

L'`EditText` ocupa 3/4 de l'amplada i el botó 1/4, el mateix resultat que l'exemple de pesos de [LinearLayout](./linearlayout.md#proporcions-diferents).

### Chains verticals

Funcionen exactament igual, però amb `Top` i `Bottom`. Un ús habitual és centrar verticalment un bloc d'elements a la pantalla amb una cadena `packed`:

```xml
<ImageView
    android:id="@+id/imgLogo"
    android:layout_width="120dp"
    android:layout_height="120dp"
    android:src="@mipmap/ic_launcher"
    app:layout_constraintBottom_toTopOf="@id/txtBenvinguda"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toTopOf="parent"
    app:layout_constraintVertical_chainStyle="packed" />

<TextView
    android:id="@+id/txtBenvinguda"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_marginTop="16dp"
    android:text="Benvinguts"
    android:textSize="24sp"
    app:layout_constraintBottom_toTopOf="@id/btnComencar"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toBottomOf="@id/imgLogo" />

<Button
    android:id="@+id/btnComencar"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_marginTop="24dp"
    android:text="Començar"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toBottomOf="@id/txtBenvinguda" />
```

Els marges entre els elements de la cadena es respecten, i el bloc sencer queda centrat verticalment.

## 9. Exemple complet: pantalla de login

Aquest exemple combina restriccions bàsiques, una guia, mides `0dp` i una chain horitzontal amb pes, tot en un sol nivell de jerarquia:

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="24dp">

    <androidx.constraintlayout.widget.Guideline
        android:id="@+id/guideSuperior"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        app:layout_constraintGuide_percent="0.15" />

    <ImageView
        android:id="@+id/imgLogo"
        android:layout_width="96dp"
        android:layout_height="96dp"
        android:src="@mipmap/ic_launcher"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="@id/guideSuperior" />

    <TextView
        android:id="@+id/txtTitol"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:text="Inicia sessió"
        android:textSize="24sp"
        android:textStyle="bold"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/imgLogo" />

    <EditText
        android:id="@+id/edtUsuari"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="32dp"
        android:hint="Usuari"
        android:inputType="text"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/txtTitol" />

    <EditText
        android:id="@+id/edtPassword"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="12dp"
        android:hint="Contrasenya"
        android:inputType="textPassword"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/edtUsuari" />

    <TextView
        android:id="@+id/txtOblit"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="8dp"
        android:text="Has oblidat la contrasenya?"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintTop_toBottomOf="@id/edtPassword" />

    <!-- Chain horitzontal amb pes: els dos botons es reparteixen l'amplada -->
    <Button
        android:id="@+id/btnRegistre"
        style="?attr/materialButtonOutlinedStyle"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="32dp"
        android:layout_marginEnd="8dp"
        android:text="Registre"
        app:layout_constraintEnd_toStartOf="@id/btnEntrar"
        app:layout_constraintHorizontal_weight="1"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/txtOblit" />

    <Button
        android:id="@+id/btnEntrar"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginStart="8dp"
        android:text="Entrar"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_weight="1"
        app:layout_constraintStart_toEndOf="@id/btnRegistre"
        app:layout_constraintTop_toTopOf="@id/btnRegistre" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

Observacions:

- Totes les vistes són fills directes del `ConstraintLayout`: no hi ha cap layout niat.
- El logo no queda enganxat a dalt, sinó a la guia del 15% de l'alçada, de manera que la posició s'adapta a la mida de la pantalla.
- Els camps de text fan servir `0dp` amb restriccions a totes dues bandes en lloc de `match_parent`.
- El text *Has oblidat la contrasenya?* només té restricció a `End`, per això queda alineat a la dreta.
- Els dos botons formen una chain amb pes i es reparteixen l'amplada a parts iguals.

## 10. ConstraintLayout a Jetpack Compose

Jetpack Compose també té un `ConstraintLayout`, amb els mateixos conceptes (restriccions, guies, barreres i chains) però declarats en Kotlin. Vegeu [Layouts en Jetpack Compose](./Jetpack_compose/layouts.md#6-constraintlayout).
