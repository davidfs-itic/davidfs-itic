## Com aplicar filtres a un RecyclerVew

## 1. Cal fer una funció per actualitzar el Recyclerview

```kotlin linenum="1"
class MyAdapter(
    private val items: List<MyItem>,
    private val onItemClick: (MyItem) -> Unit
) : RecyclerView.Adapter<MyViewHolder>() {

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): MyViewHolder {
        val inflater = LayoutInflater.from(parent.context)
        val view = inflater.inflate(R.layout.item_row, parent, false)
        return MyViewHolder(view, onItemClick)
    }

    override fun getItemCount(): Int = items.size

    override fun onBindViewHolder(holder: MyViewHolder, position: Int) {
        val item = items[position]
        holder.bind(item)
    }

    fun updateList(newList: List<MyItem>) {
        items = newList
        notifyDataSetChanged()
    }
}
```

## 2. Des de la Activity:

### Fem un menú per buscar (per exemple un PopUp per categories)

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:id="@+id/cat_totes"
        android:title="Totes les Categories" />
    <item
        android:id="@+id/cat_peliculas"
        android:title="Pel·lícules" />
    <item
        android:id="@+id/cat_llibres"
        android:title="Llibres" />
    <item
        android:id="@+id/cat_musica"
        android:title="Música" />
</menu>
```

### Creem el menú a partir del xml
En l'event on onCreateOptionsMenu.

```kotlin
override fun onCreateOptionsMenu(menu: Menu?): Boolean {

        menuInflater.inflate(R.menu.menu_cerca, menu)
        
        // 3. (OPCIONAL) **Configuració del SearchView:**
        // Aquí és on continuaries per obtenir les referències del SearchView
        // i configurar els seus listeners.
        
        // Exemple d'obtenció del SearchView:
        // val searchItem: MenuItem? = menu?.findItem(R.id.action_cerca)
        // val searchView = searchItem?.actionView as? SearchView
        // Incialitzar el searchView amb les opcions i listeners.

        return true // Retorna 'true' per indicar que el menú s'ha mostrat
    }
```


### Sobrescrivim el onOptionsItemSelected

Per capturar quan l'usuari seleccioni el item del menú

```kotlin
override fun onOptionsItemSelected(item: MenuItem): Boolean {
    return when (item.itemId) {
        R.id.action_category_button -> {
            // 1. Mostrar el PopupMenu
            showCategoryPopupMenu(findViewById(R.id.action_category_button))
            true
        }
        else -> super.onOptionsItemSelected(item)
    }
}
```

### Creem el PopUpMenú:

```kotlin
private fun showCategoryPopupMenu(view: View) {

        // Inicialitza el PopupMenu amb el context de l'Activitat i el view del menú
        val popup = PopupMenu(this, view)
        
        // Infla el menú definit a popup_categories.xml
        popup.menuInflater.inflate(R.menu.popup_categories, popup.menu)
        
        // Defineix el Listener per gestionar les seleccions
        popup.setOnMenuItemClickListener { menuItem ->
            when (menuItem.itemId) {
                R.id.cat_totes -> {
                    applyCategoryFilter("Totes")
                    true
                }
                R.id.cat_peliculas -> {
                    applyCategoryFilter("Pel·lícules")
                    true
                }
                R.id.cat_llibres -> {
                    applyCategoryFilter("Llibres")
                    true
                }
                R.id.cat_musica -> {
                    applyCategoryFilter("Música")
                    true
                }
                else -> false
            }
        }
        
        // Mostrar el menú
        popup.show()
    }

    private fun applyCategoryFilter(category: String) {
        Toast.makeText(this, "Filtrat per: $category", Toast.LENGTH_SHORT).show()
        // Aquí s'aplica la lògica de filtratge a la RecyclerView o Llista.
    }
    ```