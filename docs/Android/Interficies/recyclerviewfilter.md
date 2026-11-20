# Com aplicar filtres de categoria a un RecyclerView

Filtrar per categoria vol dir mostrar al RecyclerView només els elements que pertanyen a una categoria concreta (p. ex. només els llibres), triada per l'usuari des d'un menú. A diferència de la [cerca per text](./recyclersearchview.md), aquí l'usuari no escriu res: tria una opció d'una llista tancada.

Veurem dues maneres de mostrar les categories:

- **Amb un submenú** de la barra superior: la manera més senzilla, suficient en la majoria de casos.
- **Amb un PopupMenu**: un desplegable que podem obrir des de qualsevol View.

Documentació oficial:

- [Menus](https://developer.android.com/develop/ui/views/components/menus)
- [PopupMenu](https://developer.android.com/reference/androidx/appcompat/widget/PopupMenu)

Partim de l'exemple de [RecyclerView amb llista en memòria](./recyclerview.md).

!!! warning "Cal una barra superior"
    `onCreateOptionsMenu` només es crida si l'Activity té una ActionBar o una Toolbar configurada amb `setSupportActionBar`. Si el tema és `NoActionBar` i no hi ha Toolbar, el menú no apareixerà. Vegeu [La Toolbar com a AppBar](./toolbar.md#1-la-toolbar-com-a-appbar-fonaments).

## 1. Model de dades

Les categories són un conjunt tancat de valors, per això les representem amb una `enum class`. Així evitem escriure el nom de la categoria com a text a diversos llocs: si ens equivoquem en una lletra ("Pelicules" en lloc de "Pel·lícules"), el compilador no ens avisaria i el filtre no trobaria cap element.

```kotlin
// Fitxer: Categoria.kt
enum class Categoria(val nom: String) {
    PELICULES("Pel·lícules"),
    LLIBRES("Llibres"),
    MUSICA("Música")
}
```

Afegim la categoria a cada element:

```kotlin
// Fitxer: MyItem.kt
data class MyItem(
    val title: String,
    val subtitle: String,
    val categoria: Categoria
)
```

I la llista de dades:

```kotlin
// Fitxer: DataSource.kt
object DataSource {
    val items = listOf(
        MyItem("Interstellar", "Christopher Nolan", Categoria.PELICULES),
        MyItem("El senyor dels anells", "J. R. R. Tolkien", Categoria.LLIBRES),
        MyItem("Abbey Road", "The Beatles", Categoria.MUSICA),
        MyItem("Matilda", "Roald Dahl", Categoria.LLIBRES),
        MyItem("Coco", "Pixar", Categoria.PELICULES)
    )
}
```

## 2. Adapter amb una funció per actualitzar la llista

L'adapter necessita una funció que li permeti rebre una llista nova i tornar a pintar el RecyclerView. Per això `items` és `var` i no `val`.

```kotlin
class MyAdapter(
    private var items: List<MyItem>,
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

## 3. Filtrar amb un submenú

### Flux

- L'usuari prem el botó "Categories" de la barra superior i Android mostra el submenú.
- L'usuari tria una categoria i Android crida `onOptionsItemSelected`.
- `applyCategoryFilter` filtra la llista completa.
- `adapter.updateList` passa la llista filtrada a l'adapter i el RecyclerView es redibuixa.

### Definició del menú (XML)

Fitxer `res/menu/menu_cerca.xml`. Dins de l'`<item>` de la barra niem un altre `<menu>` amb les categories: Android el mostra automàticament com a submenú.

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <item
        android:id="@+id/action_category_button"
        android:title="Categories"
        app:showAsAction="ifRoom">
        <menu>
            <group android:checkableBehavior="single">
                <item
                    android:id="@+id/cat_totes"
                    android:title="Totes les categories"
                    android:checked="true" />
                <item
                    android:id="@+id/cat_pelicules"
                    android:title="Pel·lícules" />
                <item
                    android:id="@+id/cat_llibres"
                    android:title="Llibres" />
                <item
                    android:id="@+id/cat_musica"
                    android:title="Música" />
            </group>
        </menu>
    </item>
</menu>
```

- `app:showAsAction="ifRoom"`: el botó es mostra a la barra si hi ha espai; si no, va al menú de desbordament (els tres punts). En tots dos casos el submenú funciona.
- `<group android:checkableBehavior="single">`: els elements del grup es mostren amb un botó de selecció (radio button) i només un pot estar marcat alhora. Així l'usuari veu quin filtre està aplicat.
- `android:checked="true"`: en obrir l'app, l'opció "Totes les categories" apareix marcada.

### Activity

A l'Activity guardem `llistaCompleta`, la llista original sense filtrar.

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var adapter: MyAdapter

    // Llista original: MAI la modifiquem, sempre filtrem a partir d'aquesta
    private val llistaCompleta = DataSource.items

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val recyclerView = findViewById<RecyclerView>(R.id.recyclerView)
        recyclerView.layoutManager = LinearLayoutManager(this)

        adapter = MyAdapter(llistaCompleta) { item ->
            Toast.makeText(this, "Has clicat: ${item.title}", Toast.LENGTH_SHORT).show()
        }
        recyclerView.adapter = adapter
    }
}
```

### Inflar el menú

Al mètode `onCreateOptionsMenu` convertim l'XML en el menú de la barra superior:

```kotlin
override fun onCreateOptionsMenu(menu: Menu?): Boolean {
    menuInflater.inflate(R.menu.menu_cerca, menu)
    return true // true indica que el menú s'ha de mostrar
}
```

### Capturar l'opció triada

Les opcions del submenú també arriben a `onOptionsItemSelected`, igual que els elements de la barra:

```kotlin
override fun onOptionsItemSelected(item: MenuItem): Boolean {
    val categoria = when (item.itemId) {
        R.id.cat_totes -> null // null = sense filtre
        R.id.cat_pelicules -> Categoria.PELICULES
        R.id.cat_llibres -> Categoria.LLIBRES
        R.id.cat_musica -> Categoria.MUSICA
        else -> return super.onOptionsItemSelected(item)
    }
    item.isChecked = true // marquem l'opció triada
    applyCategoryFilter(categoria)
    return true
}
```

- Fem servir `null` per representar "Totes les categories", ja que no és cap valor de l'enum.
- El `when` retorna la categoria triada. Si l'id no és cap de les categories (p. ex. el botó "Categories" mateix), deixem que el gestioni `super`.
- Cal posar `isChecked = true` manualment: Android dibuixa els botons de selecció però no els marca sol.

#### Què vol dir retornar `true` o `false`?

- `true`: "ja he gestionat aquest clic", Android no fa res més.
- `false` (o cridar `super`): "no l'he gestionat", Android continua buscant qui el pot gestionar (p. ex. el botó Up o un Fragment).

### Aplicar el filtre

```kotlin
private fun applyCategoryFilter(categoria: Categoria?) {
    val filtrats = if (categoria == null) {
        llistaCompleta
    } else {
        llistaCompleta.filter { it.categoria == categoria }
    }
    adapter.updateList(filtrats)

    val text = categoria?.nom ?: "Totes"
    Toast.makeText(this, "Filtrat per: $text", Toast.LENGTH_SHORT).show()
}
```

!!! warning "Filtra sempre a partir de la llista completa"
    Un error habitual és filtrar la llista que té l'adapter en aquell moment. Si l'usuari filtra per "Llibres" i després per "Música", la segona vegada només es buscaria entre els llibres i el resultat seria buit. I en triar "Totes" ja no hi hauria manera de recuperar els elements eliminats. Per això guardem `llistaCompleta` a part i no la modifiquem mai.

## 4. Filtrar amb un PopupMenu

El submenú sempre penja de la barra superior. Hi ha casos en què no n'hi ha prou:

- Volem obrir el desplegable des d'una View qualsevol: un botó del layout, un element de la llista...
- Volem decidir en temps d'execució quines opcions es mostren.

En aquests casos fem servir un `PopupMenu`: un desplegable que creem des del codi i que apareix just a sota de la View que li indiquem (l'**ancoratge**).

El model de dades, l'adapter, `llistaCompleta` i `applyCategoryFilter` són exactament els mateixos de l'apartat anterior. Només canvia com es mostren les categories i com es captura l'opció triada.

### Flux

- L'usuari prem el botó "Categories" de la barra superior.
- Android crida `onOptionsItemSelected`.
- Creem i mostrem un `PopupMenu` ancorat al botó.
- L'usuari tria una categoria i s'executa el listener del `PopupMenu`.
- Cridem `applyCategoryFilter`, igual que abans.

### Definició dels menús (XML)

Ara tenim dos menús separats. El de la barra (`res/menu/menu_cerca.xml`) només té el botó, sense submenú:

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <item
        android:id="@+id/action_category_button"
        android:title="Categories"
        app:showAsAction="always" />
</menu>
```

!!! info "Per què `always` i no `ifRoom`?"
    Necessitem la View del botó per ancorar-hi el `PopupMenu`. Amb `ifRoom`, si no hi ha espai, Android mou l'element al menú de desbordament i aleshores no existeix cap View amb aquest id: `findViewById` retornaria `null` i l'aplicació petaria. Amb `always` el botó sempre és visible a la barra.

I les categories passen a un fitxer propi, `res/menu/popup_categories.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <group android:checkableBehavior="single">
        <item
            android:id="@+id/cat_totes"
            android:title="Totes les categories" />
        <item
            android:id="@+id/cat_pelicules"
            android:title="Pel·lícules" />
        <item
            android:id="@+id/cat_llibres"
            android:title="Llibres" />
        <item
            android:id="@+id/cat_musica"
            android:title="Música" />
    </group>
</menu>
```

### Obrir el PopupMenu

`onCreateOptionsMenu` és igual que abans. A `onOptionsItemSelected` ara només capturem el botó de la barra i obrim el desplegable:

```kotlin
override fun onOptionsItemSelected(item: MenuItem): Boolean {
    return when (item.itemId) {
        R.id.action_category_button -> {
            // La View del botó serveix d'ancoratge per al desplegable
            showCategoryPopupMenu(findViewById(R.id.action_category_button))
            true
        }
        else -> super.onOptionsItemSelected(item)
    }
}
```

### Crear el PopupMenu

A diferència del submenú, el `PopupMenu` es crea de nou cada vegada que s'obre, i per tant no recorda quina opció estava marcada. Per això guardem l'id de l'opció seleccionada en una propietat de l'Activity:

```kotlin
// Opció del desplegable marcada actualment
private var idSeleccionat = R.id.cat_totes
```

```kotlin
private fun showCategoryPopupMenu(view: View) {
    // Crea el PopupMenu ancorat a la View del botó
    val popup = PopupMenu(this, view)

    // Infla el menú definit a popup_categories.xml
    popup.menuInflater.inflate(R.menu.popup_categories, popup.menu)

    // Tornem a marcar l'opció actual
    popup.menu.findItem(idSeleccionat).isChecked = true

    // Listener que s'executa quan l'usuari tria una opció
    popup.setOnMenuItemClickListener { menuItem ->
        val categoria = when (menuItem.itemId) {
            R.id.cat_totes -> null
            R.id.cat_pelicules -> Categoria.PELICULES
            R.id.cat_llibres -> Categoria.LLIBRES
            R.id.cat_musica -> Categoria.MUSICA
            else -> return@setOnMenuItemClickListener false
        }
        idSeleccionat = menuItem.itemId
        applyCategoryFilter(categoria)
        true
    }

    popup.show()
}
```

- El `when` és el mateix que en el submenú. Només canvia on és: dins del listener del `PopupMenu` en lloc de `onOptionsItemSelected`.
- `return@setOnMenuItemClickListener false`: surt només de la lambda (no de la funció) indicant que l'opció no s'ha gestionat.

### Què canvia respecte del submenú

| | Submenú | PopupMenu |
|---|---|---|
| Fitxers XML | Un (submenú niat) | Dos (barra i desplegable) |
| On es captura l'opció | `onOptionsItemSelected` | Listener del `PopupMenu` |
| `showAsAction` | `ifRoom` | `always` (cal la View per ancorar) |
| Recorda l'opció marcada | Sí | No, cal guardar `idSeleccionat` |
| Es pot obrir des de qualsevol View | No | Sí |

## 5. Resum

- Les categories es defineixen amb una `enum class` per evitar errors amb textos repetits.
- L'adapter té una funció `updateList` per rebre una llista nova.
- En triar una categoria, filtrem **sempre** a partir de `llistaCompleta` i passem el resultat a l'adapter.
- Amb `checkableBehavior="single"` l'usuari veu quin filtre està aplicat.
- El **submenú** és l'opció més senzilla quan les categories pengen de la barra superior.
- El **PopupMenu** és útil quan el desplegable s'ha d'obrir des d'una altra View, a canvi d'una mica més de codi.
