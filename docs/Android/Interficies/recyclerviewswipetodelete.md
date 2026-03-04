# Swipe to Delete al RecyclerView

## 1. Què és el Swipe to Delete?

És un patró d'interfície que permet a l'usuari eliminar un element de la llista fent lliscar el dit cap a l'esquerra o la dreta sobre la fila.

Android proporciona la classe `ItemTouchHelper` que detecta gestos de swipe (i drag) sobre un RecyclerView sense necessitat de biblioteques externes.

## 2. Com funciona?

El flux és el següent:

1. Creem un `ItemTouchHelper.SimpleCallback` que detecta el gest de swipe.
2. Quan l'usuari fa swipe, es crida el mètode `onSwiped()` amb la posició de la fila.
3. Dins `onSwiped()`, demanem a l'adapter que elimini l'element d'aquella posició.
4. Connectem l'`ItemTouchHelper` al RecyclerView.

## 3. Mètode per eliminar a l'Adapter

Cal afegir un mètode `removeAt(position)` a l'adapter que elimini l'element de la llista i notifiqui el canvi.

```kotlin linenums="1"
// MyAdapter.kt
class MyAdapter(
    private val items: MutableList<MyItem>,
    private val onItemClick: (MyItem) -> Unit
) : RecyclerView.Adapter<MyViewHolder>() {

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): MyViewHolder {
        val inflater = LayoutInflater.from(parent.context)
        val view = inflater.inflate(R.layout.item_row, parent, false)
        return MyViewHolder(view, onItemClick)
    }

    override fun getItemCount(): Int = items.size

    override fun onBindViewHolder(holder: MyViewHolder, position: Int) {
        holder.bind(items[position])
    }

    /**
     * Elimina l'element a la posició indicada
     * i notifica el RecyclerView perquè faci l'animació.
     */
    fun removeAt(position: Int) {
        items.removeAt(position)
        notifyItemRemoved(position)
    }
}
```

### Explicació

- La llista `items` ha de ser `MutableList` perquè es pugui modificar.
- `removeAt(position)` elimina l'element de la llista i crida `notifyItemRemoved(position)` perquè el RecyclerView actualitzi la vista amb animació.


## 4. Configurar l'ItemTouchHelper a l'Activity

```kotlin linenums="1"
// MainActivity.kt
import android.os.Bundle
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity
import androidx.recyclerview.widget.ItemTouchHelper
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView

class MainActivity : AppCompatActivity() {

    private lateinit var recyclerView: RecyclerView
    private lateinit var adapter: MyAdapter

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // 1. Obtenir referència al RecyclerView
        recyclerView = findViewById(R.id.recyclerView)
        recyclerView.layoutManager = LinearLayoutManager(this)

        // 2. Crear l'Adapter
        adapter = MyAdapter(
            items = DataSource.items,
            onItemClick = { item ->
                Toast.makeText(this, "Has clicat: ${item.title}", Toast.LENGTH_SHORT).show()
            }
        )
        recyclerView.adapter = adapter

        // 3. Configurar el swipe to delete
        val swipeHandler = ItemTouchHelper(swipeToDeleteCallback)
        swipeHandler.attachToRecyclerView(recyclerView)
    }

    /**
     * Callback que detecta el gest de swipe i elimina l'element.
     *
     * Paràmetres del constructor SimpleCallback(dragDirs, swipeDirs):
     *   - dragDirs = 0 → no volem drag & drop
     *   - swipeDirs = LEFT or RIGHT → detectem swipe en ambdues direccions
     */
    private val swipeToDeleteCallback = object : ItemTouchHelper.SimpleCallback(
        0,
        ItemTouchHelper.LEFT or ItemTouchHelper.RIGHT
    ) {
        // No volem drag & drop, retornem false
        override fun onMove(
            recyclerView: RecyclerView,
            viewHolder: RecyclerView.ViewHolder,
            target: RecyclerView.ViewHolder
        ): Boolean = false

        // Quan l'usuari fa swipe, eliminem l'element
        override fun onSwiped(viewHolder: RecyclerView.ViewHolder, direction: Int) {
            val position = viewHolder.adapterPosition
            adapter.removeAt(position)
        }
    }
}
```

### Explicació pas a pas

**`ItemTouchHelper.SimpleCallback(dragDirs, swipeDirs)`**

- `dragDirs = 0` → No activem el drag & drop (arrossegar per reordenar).
- `swipeDirs = ItemTouchHelper.LEFT or ItemTouchHelper.RIGHT` → Detectem swipe cap a l'esquerra i cap a la dreta. Si només volem un sentit, posem només `LEFT` o `RIGHT`.

**`onMove(...)`**

- Es crida quan es detecta un gest de drag & drop. Com que no el volem, retornem `false`.

**`onSwiped(viewHolder, direction)`**

- Es crida quan l'usuari completa el gest de swipe.
- `viewHolder.adapterPosition` ens dóna la posició de la fila dins la llista.
- Cridem `adapter.removeAt(position)` per eliminar l'element.

**`attachToRecyclerView(recyclerView)`**

- Connecta l'`ItemTouchHelper` al RecyclerView perquè comenci a detectar gestos.

## 5. Limitar la direcció del swipe

Si volem que només funcioni cap a l'esquerra (com és habitual en moltes apps):

```kotlin
// Canviar el segon paràmetre del SimpleCallback
ItemTouchHelper.SimpleCallback(0, ItemTouchHelper.LEFT)
```

## 6. Resum

1. Afegir un mètode `removeAt(position)` a l'adapter que elimini de la llista i cridi `notifyItemRemoved`.
2. Crear un `ItemTouchHelper.SimpleCallback` amb `onSwiped()` que cridi `adapter.removeAt()`.
3. Connectar l'`ItemTouchHelper` al RecyclerView amb `attachToRecyclerView()`.
