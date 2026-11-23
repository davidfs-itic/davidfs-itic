# Afegir i eliminar ítems al RecyclerView

Un RecyclerView no detecta per si sol els canvis a la llista de dades. Cada vegada que afegim o eliminem un element, hem de fer dues coses: modificar la llista i **notificar l'adapter** del canvi perquè actualitzi la pantalla.

Documentació oficial:

- [RecyclerView.Adapter](https://developer.android.com/reference/androidx/recyclerview/widget/RecyclerView.Adapter)

Partim de l'exemple de [RecyclerView amb llista en memòria](./recyclerview.md), on les dades són a `DataSource.items` (una `MutableList<MyItem>`).

!!! info "Per què funciona modificar `DataSource.items` des de l'Activity?"
    A l'exemple base, l'Activity crea l'adapter amb `MyAdapter(items = DataSource.items, ...)`. L'adapter no rep una còpia, sinó una referència a **la mateixa llista**. Per tant, si afegim o eliminem elements a `DataSource.items`, l'adapter veu el canvi quan torna a llegir la llista.

    El que l'adapter no sap és **quan** ha canviat. Per això cal cridar els mètodes `notify...`.

## 1. Afegir un ítem

Per afegir un element nou a la llista i que es mostri al RecyclerView:

```kotlin linenums="1"
// A l'Activity o Fragment

// 1. Crear el nou element
val nouItem = MyItem("Nou element", "Subtítol nou")

// 2. Afegir-lo a la llista de dades
DataSource.items.add(nouItem)

// 3. Notificar l'adapter que s'ha inserit un element
adapter.notifyItemInserted(DataSource.items.size - 1)
```

### Explicació

- `DataSource.items.add(nouItem)` → afegeix l'element al final de la llista.
- `notifyItemInserted(posició)` → indica al RecyclerView que s'ha afegit un element a la posició indicada. Això permet que el RecyclerView faci l'animació d'inserció.
- La posició és `size - 1` perquè, després d'afegir, l'últim element és a l'índex `size - 1`.

### Afegir en una posició concreta

```kotlin linenums="1"
val nouItem = MyItem("Element inserit", "A la posició 2")

// Inserir a la posició 2 (l'índex comença a 0)
DataSource.items.add(2, nouItem)
adapter.notifyItemInserted(2)
```

Els elements que hi havia a partir de la posició 2 es desplacen una posició avall. El RecyclerView ho té en compte i en fa l'animació.

## 2. Eliminar un ítem

### Eliminar per posició

```kotlin linenums="1"
// A l'Activity o Fragment

// Suposem que volem eliminar el segon element
val position = 1

// 1. Eliminar de la llista de dades
DataSource.items.removeAt(position)

// 2. Notificar l'adapter que s'ha eliminat un element
adapter.notifyItemRemoved(position)
```

### Eliminar per objecte

Sovint no tenim la posició, sinó l'objecte. Per exemple, al callback `onItemClick` de l'exemple base rebem el `MyItem` que s'ha clicat. En aquest cas, busquem la seva posició **actual** a la llista:

```kotlin linenums="1"
adapter = MyAdapter(
    items = DataSource.items,
    onItemClick = { item ->
        // Eliminar l'element clicat
        val index = DataSource.items.indexOf(item)

        if (index != -1) {
            DataSource.items.removeAt(index)
            adapter.notifyItemRemoved(index)
        }
    }
)
```

- `indexOf(item)` retorna la posició de l'element a la llista, o `-1` si no hi és.
- Buscar la posició en el moment de l'eliminació és més segur que guardar-la abans: si s'han afegit o eliminat altres elements, una posició guardada pot ser incorrecta. Vegeu la mateixa idea explicada amb `bindingAdapterPosition` a [Swipe to Delete](./recyclerviewswipetodelete.md#explicacio-pas-a-pas).

!!! warning "Elements repetits"
    `indexOf` compara amb `equals()`. Com que `MyItem` és una `data class`, dos elements amb el mateix `title` i `subtitle` es consideren iguals, i `indexOf` retornarà el primer que trobi.

## 3. Exemple complet a l'Activity

Exemple d'una Activity amb dos botons: un per afegir i un per eliminar l'últim element.

### Layout (activity_main.xml)

```xml
<!-- Fitxer: res/layout/activity_main.xml -->
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:padding="8dp">

        <Button
            android:id="@+id/btnAfegir"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="Afegir" />

        <Button
            android:id="@+id/btnEliminar"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="Eliminar últim" />

    </LinearLayout>

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1" />

</LinearLayout>
```

El RecyclerView té `layout_height="0dp"` i `layout_weight="1"` perquè ocupi tot l'espai que deixen lliures els botons. Vegeu [LinearLayout](./linearlayout.md).

### Activity (MainActivity.kt)

```kotlin linenums="1"
// MainActivity.kt
import android.os.Bundle
import android.widget.Button
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView

class MainActivity : AppCompatActivity() {

    private lateinit var recyclerView: RecyclerView
    private lateinit var adapter: MyAdapter

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // 1. Configurar el RecyclerView (igual que a l'exemple base)
        recyclerView = findViewById(R.id.recyclerView)
        recyclerView.layoutManager = LinearLayoutManager(this)

        adapter = MyAdapter(
            items = DataSource.items,
            onItemClick = { item ->
                Toast.makeText(this, "Has clicat: ${item.title}", Toast.LENGTH_SHORT).show()
            }
        )
        recyclerView.adapter = adapter

        // 2. Botó per afegir
        findViewById<Button>(R.id.btnAfegir).setOnClickListener {
            afegirItem()
        }

        // 3. Botó per eliminar
        findViewById<Button>(R.id.btnEliminar).setOnClickListener {
            eliminarUltimItem()
        }
    }

    private fun afegirItem() {
        val nouItem = MyItem("Element ${DataSource.items.size + 1}", "Subtítol nou")

        DataSource.items.add(nouItem)
        adapter.notifyItemInserted(DataSource.items.size - 1)

        // Fer scroll fins al nou element
        recyclerView.scrollToPosition(DataSource.items.size - 1)
    }

    private fun eliminarUltimItem() {
        if (DataSource.items.isNotEmpty()) {
            val ultimaPosicio = DataSource.items.size - 1
            DataSource.items.removeAt(ultimaPosicio)
            adapter.notifyItemRemoved(ultimaPosicio)
        } else {
            Toast.makeText(this, "La llista és buida", Toast.LENGTH_SHORT).show()
        }
    }
}
```

### Explicació

- `scrollToPosition(posició)` → desplaça la llista fins a l'element indicat. Sense això, si la llista és llarga, l'element nou s'afegiria a baix de tot i l'usuari no el veuria.
- Abans d'eliminar comprovem `isNotEmpty()`: si la llista és buida, `removeAt(-1)` llançaria una `IndexOutOfBoundsException`.

!!! info "Alternativa: posar la lògica a l'adapter"
    En aquest exemple, l'Activity modifica la llista i també crida `notify...`. Una alternativa és afegir a l'adapter mètodes com `addItem(item)` o `removeAt(position)` que facin les dues coses alhora. Així l'Activity no pot oblidar-se de notificar. És el que es fa a [Swipe to Delete](./recyclerviewswipetodelete.md#3-metode-per-eliminar-a-ladapter).

## 4. Mètodes de notificació de l'Adapter

El `RecyclerView.Adapter` proporciona diversos mètodes per notificar canvis:

| Mètode | Ús |
|--------|-----|
| `notifyItemInserted(position)` | S'ha afegit un element a la posició |
| `notifyItemRemoved(position)` | S'ha eliminat l'element de la posició |
| `notifyItemChanged(position)` | L'element a la posició s'ha modificat |
| `notifyItemRangeInserted(start, count)` | S'han afegit `count` elements a partir de `start` |
| `notifyItemRangeRemoved(start, count)` | S'han eliminat `count` elements a partir de `start` |
| `notifyDataSetChanged()` | Tota la llista ha canviat (menys eficient) |

**Recomanació:** Utilitzeu els mètodes específics (`notifyItemInserted`, `notifyItemRemoved`...) en lloc de `notifyDataSetChanged()`, ja que permeten animacions i són més eficients.

`notifyDataSetChanged()` té sentit quan substituïm tota la llista per una altra, com quan apliquem un filtre. Vegeu [Com aplicar filtres de categoria a un RecyclerView](./recyclerviewfilter.md#2-adapter-amb-una-funcio-per-actualitzar-la-llista).

## 5. Resum

1. Després de modificar la llista, **sempre** cal notificar l'adapter.
2. Utilitzar el mètode de notificació adequat (`notifyItemInserted`, `notifyItemRemoved`...) per obtenir animacions.
3. Si eliminem a partir d'un objecte, buscar-ne la posició actual amb `indexOf` en el moment d'eliminar.
4. Si s'utilitza un ViewModel, la llista la gestiona el ViewModel i no l'Activity. Vegeu un exemple a [DialogFragment amb ViewModel](../Arquitectura/dialogfragmentviewmodel.md).
