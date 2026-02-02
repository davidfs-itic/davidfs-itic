# Afegir i eliminar ítems al RecyclerView



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

### Afegir en una posició concreta

```kotlin linenums="1"
val nouItem = MyItem("Element inserit", "A la posició 2")

// Inserir a la posició 2 (índex comença a 0)
DataSource.items.add(2, nouItem)
adapter.notifyItemInserted(2)
```

## 2. Eliminar un ítem

Per eliminar un element de la llista:

```kotlin linenums="1"
// A l'Activity o Fragment

// Suposem que volem eliminar l'element a la posició 'position'
val position = 1  // per exemple, el segon element

// 1. Eliminar de la llista de dades
DataSource.items.removeAt(position)

// 2. Notificar l'adapter que s'ha eliminat un element
adapter.notifyItemRemoved(position)
```

### Eliminar per objecte

Si tenim la referència a l'objecte que volem eliminar:

```kotlin linenums="1"
val itemAEliminar = DataSource.items[position]

// Trobar la posició de l'element
val index = DataSource.items.indexOf(itemAEliminar)

if (index != -1) {
    DataSource.items.removeAt(index)
    adapter.notifyItemRemoved(index)
}
```

## 3. Exemple complet a l'Activity

Exemple d'una Activity amb dos botons: un per afegir i un per eliminar l'últim element.

### Layout (activity_main.xml)

```xml
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

        //....

        // Botó per afegir
        findViewById<Button>(R.id.btnAfegir).setOnClickListener {
            afegirItem()
        }

        // Botó per eliminar
        findViewById<Button>(R.id.btnEliminar).setOnClickListener {
            eliminarUltimItem()
        }
    }

    private fun afegirItem() {
        val nouItem = MyItem("Element Nou", "Subtítol Nou")
        
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

## 5. Mètodes de notificació de l'Adapter

El `RecyclerView.Adapter` proporciona diversos mètodes per notificar canvis:

| Mètode | Ús |
|--------|-----|
| `notifyItemInserted(position)` | S'ha afegit un element a la posició |
| `notifyItemRemoved(position)` | S'ha eliminat l'element de la posició |
| `notifyItemChanged(position)` | L'element a la posició s'ha modificat |
| `notifyDataSetChanged()` | Tota la llista ha canviat (menys eficient) |

**Recomanació:** Utilitzeu els mètodes específics (`notifyItemInserted`, `notifyItemRemoved`) en lloc de `notifyDataSetChanged()`, ja que permeten animacions i són més eficients.

## 6. Resum

1. Després de modificar la llista, **sempre** notificar l'adapter.
2. Utilitzar el mètode de notificació adequat per obtenir animacions.
3. Adaptar el codi en cas que s'estigui utlitzant ViewModel. 
