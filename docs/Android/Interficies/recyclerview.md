# RecyclerView amb llista en memòria (Kotlin)

Documentació oficial:
https://developer.android.com/develop/ui/views/layout/recyclerview

Exemple:
https://github.com/davidfs-itic/RecyclerView


## 1. Què és un RecyclerView?
Un RecyclerView és un component que permet mostrar llistes (o graelles) d’elements reciclant les vistes per millorar el rendiment.
Substitueix l’antic ListView i ofereix més flexibilitat (layout managers, animacions, etc.).

Per utilitzar un RecyclerView cal:

- Una llista de dades (en memòria, en aquest exemple).
- Un layout XML per a la pantalla que conté el RecyclerView.
- Un layout XML per a cada fila (ítem) de la llista.
- Un Adapter amb un ViewHolder per “pintar” les dades a cada fila.

## 2. Model de dades (data class i llista)

### Data Class

Creem una classe senzilla per representar cada element de la llista.

```kotlin  linenums="1"
// Fitxer: MyItem.kt
data class MyItem(
    val title: String,
    val subtitle: String
)
```

### Llista de dades en un object (Kotlin)

#### Què és un object a Kotlin?

En Kotlin, un object és una construcció que crea una única instància (patró Singleton) de manera automàtica.
S’utilitza quan es vol un únic punt d’accés compartit, per exemple una llista de dades comuna per a diferents pantalles.

#### Característiques principals:

- No cal fer new ni cridar cap constructor; s’accedeix pel nom (DataProvider.items).
- Es crea la instància la primera vegada que es fa servir.
- És útil per guardar dades en memòria mentre l’app està en execució.


Objecte que conté la llista: DataProvider

```kotlin  linenums="1"
// Fitxer: DataProvider.kt

/**
 * Objecte singleton que proporciona dades en memòria
 * per ser utilitzades al RecyclerView.
 */
object DataProvider {

    // Llista de dades en memòria (només de lectura)
    val items: List<MyItem> = listOf(
        MyItem("Element 1", "Subtítol 1"),
        MyItem("Element 2", "Subtítol 2"),
        MyItem("Element 3", "Subtítol 3"),
        MyItem("Element 4", "Subtítol 4")
    )
}
```
**Què fà aquest codi?**



```kotlin  linenums="1"
object DataProvider
```
- Declara un objecte únic anomenat DataProvider.
- No s’instancia amb DataProvider(), sinó que s’utilitza directament pel nom.

```kotlin  linenums="1"
val items: List<MyItem>
```
- És una propietat pública que conté la llista en memòria.
- El tipus és List<MyItem>, per tant és immutable (no es poden afegir/eliminar elements).
- Es pot canviar a MutableList<MyItem> si es vol modificar la llista durant l’execució.

L’objecte DataProvider actua com un “mini repositori de dades” senzill, sense base de dades ni API.

(Per a veure com s'ha de crear un repositori de dades, veure l'apartat repositori a Arquitectura)

## 3. Layout de cada element (item_row.xml)
Aquest layout defineix com es veu una fila de la llista.

```xml
<!-- Fitxer: res/layout/item_row.xml -->
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:padding="16dp">

    <TextView
        android:id="@+id/tvTitle"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Títol"
        android:textStyle="bold"
        android:textSize="18sp" />

    <TextView
        android:id="@+id/tvSubtitle"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Subtítol"
        android:textSize="14sp" />

</LinearLayout>
```

## 4. Layout amb el RecyclerView (activity_main.xml)
Aquest layout conté el RecyclerView que ocuparà tota la pantalla.

```xml
<!-- Fitxer: res/layout/activity_main.xml -->
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

## 5. Adapter i ViewHolder (MyAdapter.kt)

En un RecyclerView aquestes classes sempre treballen juntes:

#### El Adapter sap:
- Quantes files hi ha.
- Quin layout s’utilitza per a cada fila.
- Quines dades s’han de mostrar a cada posició.

#### El ViewHolder sap:

- Quines vistes (TextView, ImageView, etc.) té una fila.
- On escriure les dades quan l’Adapter li digui “pinta l’element X”.

Sense Adapter i ViewHolder, el RecyclerView no sap ni quantes files mostrar, ni com dibuixar-les.

### Exemple codi Holder
```kotlin linenums="1"
// MyViewHolder.kt
import android.view.View
import android.widget.TextView
import androidx.recyclerview.widget.RecyclerView

/**
 * ViewHolder responsable d'una fila del RecyclerView.
 * Encapsula la vista de la fila i sap com "pintar-hi" un MyItem.
 */
class MyViewHolder(
    itemView: View,
    private val onItemClick: (MyItem) -> Unit
) : RecyclerView.ViewHolder(itemView) {

    private val tvTitle: TextView = itemView.findViewById(R.id.tvTitle)
    private val tvSubtitle: TextView = itemView.findViewById(R.id.tvSubtitle)

    /**
     * Actualitza les vistes de la fila amb les dades de MyItem
     * i configura els listeners d'esdeveniments.
     */
    fun bind(item: MyItem) {
        tvTitle.text = item.title
        tvSubtitle.text = item.subtitle

        // Exemple de gestió de clic sobre tota la fila
        itemView.setOnClickListener {
            onItemClick(item)
        }
    }
}
```

#### Explicació:

**Classe**

```kotlin 
class MyViewHolder(... ) : RecyclerView.ViewHolder(itemView)
```
Hereta de RecyclerView.ViewHolder i rep la vista de la fila (itemView) i una funció de clic (onItemClick).​

**Propietats privades (tvTitle, tvSubtitle):**

El ViewHolder busca una sola vega*da les vistes del layout item_row.xml i s’hi queda la referència.

**Mètode bind(item: MyItem):**

És el punt únic per:
 - Escriure dades a les vistes.
 - Configurar listeners específics d’aquesta fila (clic, long‑click…).

Això treu responsabilitat de “pintar dades” de l’Adapter i la concentra al ViewHolder.

### Exemple codi Adapter
```kotlin linenums="1"
// MyAdapter.kt
import android.view.LayoutInflater
import android.view.ViewGroup
import androidx.recyclerview.widget.RecyclerView

/**
 * Adapter que crea ViewHolder (MyViewHolder),
 * indica quants elements hi ha i demana que es "pintin" amb bind().
 */
class MyAdapter(
    private val items: List<MyItem>,
    private val onItemClick: (MyItem) -> Unit
) : RecyclerView.Adapter<MyViewHolder>() {

    /**
     * Crea (infla) la vista de la fila i construeix el ViewHolder.
     */
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): MyViewHolder {
        val inflater = LayoutInflater.from(parent.context)
        val view = inflater.inflate(R.layout.item_row, parent, false)
        return MyViewHolder(view, onItemClick)
    }

    /**
     * Retorna quants elements hi ha a la llista de dades.
     */
    override fun getItemCount(): Int = items.size

    /**
     * Demana al ViewHolder que mostri les dades de la posició donada.
     */
    override fun onBindViewHolder(holder: MyViewHolder, position: Int) {
        val item = items[position]
        holder.bind(item)
    }
}
```
#### Explicació:
L’Adapter ja no sap com s’actualitzen exactament les vistes; delega aquesta feina a holder.bind(item).

Responsabilitats clares de cada mètode (tal com recomana la documentació oficial):​

- onCreateViewHolder: infla el layout i crea el ViewHolder.
- getItemCount: diu al RecyclerView quantes files han de existir.
- onBindViewHolder: passa el model (MyItem) al ViewHolder perquè l’actualitzi.

## 6. Activity

Exemple de codi en l'activity. 
Els comentaris expliquen els passos que s'han de seguir.

```kotlin linenums="1"
// MainActivity.kt
import android.os.Bundle
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

        // 1. Obtenir referència al RecyclerView del layout
        recyclerView = findViewById(R.id.recyclerView)

        // 2. Configurar LayoutManager (com es col·loquen les files)
        recyclerView.layoutManager = LinearLayoutManager(this)

        // 3. Crear llista de dades (des de DataProvider o directament)
        val items = DataProvider.items

        // 4. Crear l'Adapter passant les dades + funció de callback per clics
        adapter = MyAdapter(
            items = items,
            onItemClick = { item ->
                // AQUÍ gestionem el clic: mostrem un Toast amb el títol
                Toast.makeText(
                    this,
                    "Has clicat: ${item.title}",
                    Toast.LENGTH_SHORT
                ).show()
            }
        )

        // 5. Assignar l'Adapter al RecyclerView
        recyclerView.adapter = adapter
    }
}
```

Tot i que s'utilitza una funció (anomenada callback) per passar la lògica de gestió del click en un ítem (recodem que aixó ho gestiona el ViewHolder), no és una pràctica recomenada. Està aquí per simplicitat.

Veieu "Exemple inversió de dependències amb RecyclerView" a l'apartat Arquitectura



## 7. Resum

### Idea clau:

En un RecyclerView, el ViewHolder encapsula la vista d’una fila, en guarda les referències a les seves sub‑vistes i pot contenir la lògica específica d’aquesta fila (com ara actualitzar les dades o gestionar clics), mentre que l’Adapter crea aquests ViewHolder, els recicla i els alimenta amb les dades correctes de la llista.

- onCreateViewHolder crea la vista i el ViewHolder.
- onBindViewHolder omple aquesta vista amb les dades de la posició concreta.


### Frase “resum” que podeu recordar:


L’Adapter:

- Crea ViewHolder nous (onCreateViewHolder).
- Diu quantes files hi ha (getItemCount).
- Omple les vistes del ViewHolder amb dades (onBindViewHolder).

El ViewHolder 
- Guarda referències a les vistes de la fila (TextView, ImageView, etc.).
- Encapsula la lògica de “bind”: té una funció bind(item: MyItem) que rep el model i actualitza totes les vistes.
- Gestiona esdeveniments de la fila: per exemple, configurar setOnClickListener sobre itemView o sobre algun botó concret, i notifica l’Adapter o un listener extern.
