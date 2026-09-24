# LinearLayout

`LinearLayout` és un `ViewGroup` que col·loca els seus fills un darrere l'altre en una sola direcció: en **vertical** (un a sota de l'altre) o en **horitzontal** (un al costat de l'altre). És el layout més senzill d'entendre i és molt adequat per a formularis, barres de botons o qualsevol grup d'elements alineats.

Els conceptes comuns a tots els layouts (`layout_width`, `dp`, padding, margin, `gravity`...) estan explicats a [Layouts](./layouts.md).

Documentació oficial: [https://developer.android.com/develop/ui/views/layout/linear](https://developer.android.com/develop/ui/views/layout/linear)

## 1. Orientació

L'atribut `android:orientation` indica la direcció:

- `vertical`: els fills es col·loquen en columna.
- `horizontal`: els fills es col·loquen en fila. És el valor per defecte si no s'indica res.

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Primer" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Segon" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Tercer" />

</LinearLayout>
```

!!! warning "Oblidar l'orientació"
    Si s'oblida `android:orientation`, el `LinearLayout` és horitzontal. Un error molt habitual és posar diversos elements amb `layout_width="match_parent"` en un `LinearLayout` sense orientació: el primer ocupa tota l'amplada i la resta queden fora de la pantalla.

Un `LinearLayout` **no fa scroll**. Si el contingut no hi cap, la part que sobra no es veu. Per poder desplaçar-lo cal posar-lo dins d'un `ScrollView` (vegeu l'exemple de la [secció 6](#6-exemple-complet-pantalla-de-preferencies)).

## 2. Mida dels fills

Cada fill indica la seva mida en l'eix de l'orientació i en l'eix perpendicular:

- En un `LinearLayout` **vertical**, l'alçada dels fills sol ser `wrap_content` (cada element ocupa el que necessita) i l'amplada sol ser `match_parent` o `wrap_content`.
- En un `LinearLayout` **horitzontal**, és al revés: l'amplada sol ser `wrap_content` i l'alçada `match_parent` o `wrap_content`.

## 3. Pesos: layout_weight

L'atribut `android:layout_weight` permet repartir l'**espai sobrant** del `LinearLayout` entre els fills, de manera proporcional al pes de cadascun.

Funciona així:

1. El `LinearLayout` mesura tots els fills amb la mida que tenen indicada.
2. Calcula l'espai que sobra.
3. Reparteix aquest espai entre els fills que tenen pes, segons la proporció dels pesos.

### Repartir l'espai en parts iguals

Per repartir tot l'espai en parts iguals, es posa la mida del fill en l'eix de l'orientació a **`0dp`** i el mateix pes a tots. Així l'espai que sobra és tot l'espai del pare, i es reparteix només segons els pesos:

```xml
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal">

    <Button
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:text="Cancel·lar" />

    <Button
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:text="Acceptar" />

</LinearLayout>
```

Els dos botons ocupen exactament la meitat de l'amplada cadascun, encara que els textos tinguin longituds diferents.

!!! info "Per què `0dp`?"
    Si es posa `wrap_content` en lloc de `0dp`, primer es dona a cada fill l'amplada del seu contingut i només es reparteix la resta. El resultat és que un botó amb un text més llarg acaba sent més ample. Amb `0dp`, tot l'espai es reparteix segons els pesos.

### Proporcions diferents

Amb pesos diferents, cada fill rep una part proporcional. En aquest exemple, l'`EditText` ocupa 3/4 de l'amplada i el botó 1/4:

```xml
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal">

    <EditText
        android:id="@+id/edtCerca"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="3"
        android:hint="Cerca..." />

    <Button
        android:id="@+id/btnCerca"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:text="OK" />

</LinearLayout>
```

### Un element que omple l'espai restant

Un cas molt habitual és tenir un element que ocupa tot l'espai que deixen els altres. Només cal donar pes a aquest element:

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:padding="16dp"
        android:text="Capçalera" />

    <!-- Ocupa tot l'espai entre la capçalera i el botó -->
    <TextView
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:text="Contingut" />

    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Continuar" />

</LinearLayout>
```

!!! warning "`0dp` depèn del layout pare"
    `0dp` no vol dir "omple l'espai", sinó "no tinc mida, que la calculi el pare". En un `LinearLayout`, qui calcula la mida és el pes: un fill amb `0dp` **sense** `layout_weight` fa literalment 0 i no es veu. A més, el pes només actua en l'eix de l'orientació: en un `LinearLayout` vertical, un `layout_width="0dp"` deixa la vista amb amplada 0 encara que tingui pes.

    En un `ConstraintLayout`, en canvi, `0dp` sí que fa que la vista s'estiri, però entre les seves restriccions, i `layout_weight` no hi té cap efecte (vegeu [ConstraintLayout: Mida de les vistes](./constraintlayout.md#4-mida-de-les-vistes)).

### weightSum

Per defecte, el total de pesos és la suma dels pesos dels fills. Amb `android:weightSum` al `LinearLayout` es pot fixar un total diferent. Per exemple, per fer que un botó ocupi la meitat de l'amplada encara que estigui sol:

```xml
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal"
    android:weightSum="2">

    <Button
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:text="Meitat" />

</LinearLayout>
```

## 4. Gravity i layout_gravity

La diferència general entre `gravity` i `layout_gravity` s'explica a [Layouts](./layouts.md#5-gravity-i-layout_gravity). En un `LinearLayout` s'apliquen així:

- **`android:gravity`** al `LinearLayout`: alinea **tots els fills en bloc**. Per exemple, `center` agrupa tots els fills al centre de la pantalla.
- **`android:layout_gravity`** a un fill: alinea **només aquell fill**, i només en l'eix **perpendicular** a l'orientació. En un `LinearLayout` vertical es pot moure un fill a l'esquerra, al centre o a la dreta, però no amunt o avall (la posició vertical la decideix l'ordre dels fills).

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center_vertical"
    android:orientation="vertical"
    android:padding="16dp">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="start"
        android:text="A l'esquerra" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal"
        android:text="Al centre" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="end"
        android:text="A la dreta" />

</LinearLayout>
```

Els tres textos queden agrupats al centre vertical de la pantalla (`gravity` del pare) i cadascun amb una alineació horitzontal diferent (`layout_gravity` de cada fill).

!!! warning "layout_gravity i match_parent"
    `layout_gravity` no té cap efecte visible si el fill té `match_parent` en aquell eix, perquè ja ocupa tot l'espai i no hi ha on moure'l.

## 5. Separadors

Un `LinearLayout` pot dibuixar una línia o un espai entre els seus fills amb els atributs `divider` i `showDividers`, sense haver d'afegir vistes buides:

```xml
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:divider="?android:attr/listDivider"
    android:orientation="vertical"
    android:showDividers="middle">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:padding="12dp"
        android:text="Opció 1" />

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:padding="12dp"
        android:text="Opció 2" />

</LinearLayout>
```

`showDividers` accepta `beginning`, `middle`, `end` i `none`, i es poden combinar amb `|`.

Si només es vol espai (sense línia) entre dos elements, n'hi ha prou amb un `layout_marginTop` al segon element, o amb una `Space`:

```xml
<Space
    android:layout_width="match_parent"
    android:layout_height="24dp" />
```

## 6. Exemple complet: pantalla de preferències

Aquest exemple és una pantalla de preferències típica. Cada opció és una fila amb un títol, una descripció i un control a la dreta. Tot el contingut va dins d'un `ScrollView` perquè es pugui desplaçar si hi ha més opcions de les que caben a la pantalla.

L'exemple fa servir gairebé tot el que s'ha vist en aquest document:

- Un `LinearLayout` vertical que conté totes les seccions.
- Un `LinearLayout` horitzontal per a cada fila.
- `layout_weight` perquè els textos de cada fila ocupin tot l'espai que no ocupa el control.
- `gravity` i `layout_gravity` per alinear els elements.
- Separadors entre les opcions amb `divider` i `showDividers`.

```xml
<?xml version="1.0" encoding="utf-8"?>
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/mainScroll"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:padding="16dp">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginBottom="16dp"
            android:text="Preferències"
            android:textSize="28sp"
            android:textStyle="bold" />

        <!-- Secció: General -->
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="8dp"
            android:text="General"
            android:textColor="?attr/colorPrimary"
            android:textSize="14sp"
            android:textStyle="bold" />

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:divider="?android:attr/listDivider"
            android:orientation="vertical"
            android:showDividers="middle">

            <!-- Fila: textos a l'esquerra (amb pes) i switch a la dreta -->
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:gravity="center_vertical"
                android:orientation="horizontal"
                android:paddingVertical="12dp">

                <LinearLayout
                    android:layout_width="0dp"
                    android:layout_height="wrap_content"
                    android:layout_weight="1"
                    android:orientation="vertical">

                    <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="Notificacions"
                        android:textSize="16sp" />

                    <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="Rebre avisos de missatges nous"
                        android:textSize="14sp" />
                </LinearLayout>

                <com.google.android.material.materialswitch.MaterialSwitch
                    android:id="@+id/swNotificacions"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content" />
            </LinearLayout>

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:gravity="center_vertical"
                android:orientation="horizontal"
                android:paddingVertical="12dp">

                <LinearLayout
                    android:layout_width="0dp"
                    android:layout_height="wrap_content"
                    android:layout_weight="1"
                    android:orientation="vertical">

                    <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="Mode fosc"
                        android:textSize="16sp" />

                    <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="Utilitzar colors foscos a tota l'aplicació"
                        android:textSize="14sp" />
                </LinearLayout>

                <com.google.android.material.materialswitch.MaterialSwitch
                    android:id="@+id/swModeFosc"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content" />
            </LinearLayout>

        </LinearLayout>

        <!-- Secció: Pantalla -->
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="24dp"
            android:text="Pantalla"
            android:textColor="?attr/colorPrimary"
            android:textSize="14sp"
            android:textStyle="bold" />

        <!-- Fila: etiqueta a l'esquerra i valor a la dreta -->
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:gravity="center_vertical"
            android:orientation="horizontal"
            android:paddingTop="12dp">

            <TextView
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="Mida del text"
                android:textSize="16sp" />

            <TextView
                android:id="@+id/txtMidaText"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="16sp"
                android:textSize="14sp" />
        </LinearLayout>

        <SeekBar
            android:id="@+id/skMidaText"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="8dp"
            android:max="10"
            android:progress="4" />

        <Button
            android:id="@+id/btnRestablir"
            style="?attr/materialButtonOutlinedStyle"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="end"
            android:layout_marginTop="32dp"
            android:text="Restablir valors" />

    </LinearLayout>

</ScrollView>
```

Observacions:

- El `ScrollView` només té un fill directe: el `LinearLayout` vertical principal.
- L'alçada del `LinearLayout` principal és `wrap_content`, i no `match_parent`. El `ScrollView` té l'alçada de la pantalla, i el seu fill ha de poder ser **més alt** que ella perquè hi hagi alguna cosa a desplaçar. Amb `wrap_content`, el `LinearLayout` fa l'alçada de tot el seu contingut; amb `match_parent`, estaria demanant fer exactament l'alçada de la pantalla, cosa que no té sentit dins d'un `ScrollView` (Android Studio ho marca amb un avís). Si es vol que el contingut ompli com a mínim tota la pantalla, cal afegir `android:fillViewport="true"` al `ScrollView`.
- A cada fila, el bloc de textos té `layout_width="0dp"` i `layout_weight="1"`. Per això ocupa tot l'espai que deixa el switch, i el switch sempre queda enganxat a la dreta, encara que la descripció sigui llarga.
- `gravity="center_vertical"` a la fila centra verticalment el switch respecte als dos textos.
- Els separadors entre les opcions de la secció *General* no són vistes afegides a mà: els dibuixa el mateix `LinearLayout` amb `showDividers="middle"`.
- El botó *Restablir valors* fa servir `layout_gravity="end"` per col·locar-se a la dreta dins del `LinearLayout` vertical.
- L'arrel té `android:id="@+id/main"`, que és l'`id` que fa servir el codi d'[edge-to-edge](./layoutactivities.md) de les plantilles.

!!! info "Molts nivells de niuament"
    Cada fila té dos nivells de `LinearLayout` (la fila i el bloc de textos) dins de la secció, que està dins del `LinearLayout` principal. Per a una pantalla petita no és cap problema, però mostra per què, en pantalles més complexes, és millor fer servir un `ConstraintLayout` (vegeu la [secció 7](#7-limitacions)).

Els textos estan escrits directament per simplificar l'exemple; en una aplicació real s'haurien de posar a `strings.xml` (vegeu [Layouts](./layouts.md#10-leditor-de-layouts-dandroid-studio)).

## 7. Limitacions

`LinearLayout` és molt pràctic per a disposicions en fila o en columna, però quan la pantalla és més complexa obliga a niar molts `LinearLayout` els uns dins dels altres. Això fa l'XML més difícil de llegir i empitjora el rendiment, sobretot si es fan servir pesos en diversos nivells (vegeu [Layouts](./layouts.md#9-rendiment-evitar-el-niuament-excessiu)).

En aquests casos és millor fer servir un [ConstraintLayout](./constraintlayout.md), que permet posicionar tots els elements en un sol nivell.

En Jetpack Compose, l'equivalent de `LinearLayout` són els composables `Column` i `Row` (vegeu [Layouts en Jetpack Compose](./Jetpack_compose/layouts.md#2-column)).
