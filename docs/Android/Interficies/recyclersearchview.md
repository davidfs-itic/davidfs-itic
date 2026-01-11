## Com aplicar filtres de búsqueda a un RecyclerVew

A través dels ítems del menú i del ActioViewClass, podem posar directament una barra de búsqueda (SearchView en un element del menú)

### En el menú
Al fitxer, per exemple, res/menu/menu_cerca.xml, afegeix un element <item> utilitzant l'atribut app:actionViewClass amb la classe androidx.appcompat.widget.SearchView.

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <item
        android:id="@+id/action_cerca"
        android:title="Cerca"
        android:icon="@drawable/ic_search"
        app:showAsAction="ifRoom|collapseActionView"
        app:actionViewClass="androidx.appcompat.widget.SearchView" />
</menu>
```
- android:icon: Utilitza una icona de cerca (p. ex., @android:drawable/ic_menu_search o una de Material Design).
- app:showAsAction="ifRoom|collapseActionView": Indica que es mostri com a icona a la barra si hi ha espai (ifRoom) i que es col·lapsi a una icona quan no s'utilitza (collapseActionView).
- app:actionViewClass: Estableix el widget SearchView que apareixerà.

### En l'activity (fragment)

Recuperem la referència al Searchview

```kotlin

class MainActivity : AppCompatActivity() {
    //...
    override fun onCreateOptionsMenu(menu: Menu?): Boolean {
            // 1. Infla el menú definit a l'XML
            menuInflater.inflate(R.menu.menu_cerca, menu)

            // 2. Troba l'element de menú de cerca
            val searchItem: MenuItem? = menu?.findItem(R.id.action_cerca)

            // 3. Obté la referència al SearchView
            // Si ens demana la llibreria, busquem la appcompat
            // (coincideix amb el xml)
            val searchView = searchItem?.actionView as? SearchView
            //...
```

### I programem la lògica:

Continuant en el onCreateOptionsMenu...

```kotlin

            // 4. Posa el text d'ajuda (hint)
            searchView.queryHint = "Introdueix el text a cercar..."

            //5. Necessitem un objecte que implementi OnQueryTextListener:
            val listener = object : SearchView.OnQueryTextListener {
                override fun onQueryTextSubmit(query: String?): Boolean {
                    Log.d("MainActivity", "Query submitted: $query")
                    return true
                }

                override fun onQueryTextChange(newText: String?): Boolean {
                    Log.d("MainActivity", "Query changed: $newText")
                    return true
                }
            }

            //6. Assignem aquest objecte al listener.
            searchView.setOnQueryTextListener(listener)

```

Perquè creem un object, i no simplement li passem una funció lambda, com la majoria de listener?

La resposta és perque el OnQueryTextListener no demana només una funció, sino dues, i per tant, cal especificar quina o quines funcions implementem.

En el cas dels botons, per exemple, només hi ha una funció, i kotlin es capaç de decidir, que, si una interficie només té un métode, (i és una interficie de java), es pot passar directament una lambda. 

Aixó s'anomena **SAM conversion**. 

Si es vol fer en kotlin, cal crear una **functional interface**, i el codi seria equivalent al de java.

