## 1. La Toolbar com a AppBar (Fonaments)
### Concepte i Funció
L'AppBar (Barra d'Aplicacions) és el component estructural més important a la part superior de qualsevol pantalla Android. 

El seu propòsit és:
- Identitat: Mostrar el títol de l'aplicació o la pantalla actual.
- Navegació: Allotjar el botó "Up" (fletxa enrere) o la icona del Navigation Drawer.
- Accions: Contenir les accions més importants de la pantalla (menús).

La Toolbar (proporcionada per la biblioteca AndroidX Material Components) és la implementació moderna i flexible de l'AppBar. A diferència de l'antiga ActionBar nativa, la Toolbar pot ser personalitzada amb facilitat.

La utilitzarem quan vulguem personalitzar la appbar que apareix per defecte amb els temes.

### Implementació Bàsica
Per utilitzar una Toolbar, primer s'ha d'afegir al fitxer XML del layout de l'Activity.
Exemple: XML del Layout (activity_main.xml)
```XML
<LinearLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <androidx.appcompat.widget.Toolbar
        android:id="@+id/my_toolbar"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        android:background="?attr/colorPrimary"
        android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar" /> 

    <TextView 
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="El contingut de l'Activity va aquí"
        android:padding="16dp" />

</LinearLayout>
```
Després, s'ha de configurar aquesta Toolbar com l'Action Bar de l'Activity en el codi Kotlin/Java, després d'ocultar la barra nativa al tema de l'aplicació (normalment, s'usa un tema NoActionBar), amb la funció setSupportActionBar

Exemple: Configuració a l'Activity (Kotlin)
```Kotlin
import androidx.appcompat.app.AppCompatActivity

class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val toolbar = findViewById<Toolbar>(R.id.my_toolbar)
        // 1. Configura la Toolbar com l'Action Bar del sistema
        setSupportActionBar(toolbar) 
        
        // Opcional: Estableix un títol
        supportActionBar?.title = "La Meva App" 
    }
}
```

## 2. Afegir Menús a la Toolbar
### Definició del Menú
Els ítems del menú es defineixen en un fitxer XML separat a res/menu/.

Exemple: Recurs de Menú (res/menu/main_menu.xml)
```XML
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <item
        android:id="@+id/action_cercar"
        android:icon="@drawable/ic_search" 
        android:title="Cercar"
        app:showAsAction="ifRoom" />

    <item
        android:id="@+id/action_configuracio"
        android:title="Configuració"
        app:showAsAction="never" />

</menu>
```
### Inflació
El procés d'inflar (carregar) el menú XML a la Toolbar es fa sobreescrivint el mètode onCreateOptionsMenu() de l'Activity.
Exemple: Inflació del Menú (Kotlin)
```Kotlin
class MainActivity : AppCompatActivity() {
    // ... onCreate ...
    
    override fun onCreateOptionsMenu(menu: Menu?): Boolean {
        // 1. Carrega el recurs de menú a l'Activity
        menuInflater.inflate(R.menu.main_menu, menu)
        return true // Retorna true per mostrar el menú
    }
    // ...
}
```

### Gestió de Clics i Prioritat

Per respondre a les interaccions de l'usuari amb els ítems del menú, s'ha de sobreescriure el mètode onOptionsItemSelected().
Exemple 2.3: Gestió de Clics (Kotlin)
```Kotlin
    // ...
    override fun onOptionsItemSelected(item: MenuItem): Boolean {
        return when (item.itemId) {
            R.id.action_cercar -> {
                // Lògica per a l'acció de cercar
                Toast.makeText(this, "S'ha premut Cerca", Toast.LENGTH_SHORT).show()
                true
            }
            R.id.action_configuracio -> {
                // Lògica per anar a Configuració
                Toast.makeText(this, "Obrint Configuració...", Toast.LENGTH_SHORT).show()
                true
            }
            else -> super.onOptionsItemSelected(item)
        }
    }
}
```
## 3. Gestió de la Navegació a l'AppBar
### El botó "Up"
El botó "Up" és la fletxa cap a l'esquerra que apareix a l'inici de la Toolbar. Permet a l'usuari pujar en la jerarquia de l'aplicació, és a dir, tornar a la pantalla pare de la pantalla actual. Per exemple, des de la pantalla de detall d'un producte es puja a la llista de productes.

És diferent del botó "Enrere" del sistema:

- **Enrere** torna a la pantalla anterior, sigui quina sigui, seguint l'historial (back stack).
- **Up** torna sempre a la pantalla pare, encara que l'usuari hagi arribat a la pantalla actual des d'un altre lloc (per exemple, des d'una notificació).

En una aplicació senzilla, on sempre s'arriba a la pantalla de detall des de la llista, els dos botons acaben fent el mateix.

Hi ha dues maneres de fer funcionar el botó "Up": deixar que Android ho faci automàticament a partir del manifest, o gestionar el clic nosaltres.

### Opció 1: Activity pare al manifest (automàtic)

Al fitxer `AndroidManifest.xml`, indiquem quina és l'Activity pare amb l'atribut `android:parentActivityName`:

```xml
<activity
    android:name=".DetailActivity"
    android:parentActivityName=".MainActivity" />
```

A l'Activity, només cal mostrar la fletxa amb `setDisplayHomeAsUpEnabled(true)`. No cal escriure cap codi per gestionar el clic: `AppCompatActivity` ja sap a quina Activity ha de tornar.

```kotlin
class DetailActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_detail)

        setSupportActionBar(findViewById(R.id.my_toolbar))

        // Mostra la fletxa "Up" a la Toolbar
        supportActionBar?.setDisplayHomeAsUpEnabled(true)
    }
}
```

!!! warning "L'Activity pare es torna a crear"
    Amb aquesta opció, Android tanca les Activities que hi ha a sobre de la pare i, per defecte, també destrueix la pare i la torna a crear. Es perd el seu estat: la posició de l'scroll, el text escrit en un camp, etc. Si l'usuari ha de trobar la pantalla tal com la va deixar, feu servir l'opció 2.

### Opció 2: Gestionar el clic manualment

Si no declarem `parentActivityName` al manifest, hem de gestionar el clic de la fletxa nosaltres. El botó "Up" arriba a `onOptionsItemSelected()` com qualsevol altre element del menú, amb l'id `android.R.id.home`.

El més habitual és tancar l'Activity actual amb `finish()`. Així tornem a la pantalla anterior, que es manté tal com estava:

```kotlin
class DetailActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_detail)

        setSupportActionBar(findViewById(R.id.my_toolbar))

        // Mostra la fletxa "Up" a la Toolbar
        supportActionBar?.setDisplayHomeAsUpEnabled(true)
    }

    // Gestió manual del clic a la fletxa "Up"
    override fun onOptionsItemSelected(item: MenuItem): Boolean {
        return when (item.itemId) {
            android.R.id.home -> {
                // Tanca aquesta Activity i torna a l'anterior
                finish()
                true
            }
            else -> super.onOptionsItemSelected(item)
        }
    }
}
```

Fixeu-vos que l'id és `android.R.id.home` (amb el prefix `android.`), no `R.id.home`. Aquest id el defineix Android, no nosaltres, i no apareix a cap fitxer de menú.

!!! info "Cal configurar la Toolbar abans"
    `supportActionBar` només existeix després de cridar `setSupportActionBar()`. Si la crida no s'ha fet abans, `supportActionBar` és `null` i la fletxa no apareix (gràcies a l'operador `?.`, no peta, però tampoc fa res).

### Quina opció triar

| | Opció 1: manifest | Opció 2: `finish()` |
|---|---|---|
| Codi a l'Activity | Només `setDisplayHomeAsUpEnabled` | També `onOptionsItemSelected` |
| On torna | Sempre a l'Activity pare | A la pantalla anterior |
| Estat de la pantalla on torna | Es perd (es torna a crear) | Es conserva |

!!! info "Botó Enrere amb Fragments"
    Tot el que hem vist en aquesta secció serveix per navegar entre Activities. Si l'aplicació té una sola Activity i canvia de pantalla amb Fragments, la fletxa s'ha de mostrar o amagar segons els Fragments que hi ha al back stack. Vegeu [Toolbar amb Fragments i botó Enrere](./toolbarnavback.md).
## 4. Actualització Dinàmica dels Menús
De vegades, l'estat d'un ítem de menú (p. ex., canviar una icona de "No Favorit" a "Favorit") ha de canviar mentre l'Activity està oberta.
### Invalidació del Menú
Per forçar que el sistema torni a cridar a onCreateOptionsMenu() i recreï completament el menú, s'utilitza invalidateOptionsMenu(). 

Això és útil si la visibilitat o el títol d'alguns ítems depèn d'un estat global que acaba de canviar.
Exemple: Forçar la Recreació del Menú (Kotlin)
```Kotlin
// Després d'un canvi d'estat a la nostra Activity...
fun onDataUpdated() {
    // ... 
    // Crida a invalidateOptionsMenu() per recrear el menú amb les noves dades
    invalidateOptionsMenu() 
}
```
### Preparació del Menú
El mètode onPrepareOptionsMenu() es crida just abans que es mostri el menú. Aquest és el lloc ideal per canviar la visibilitat o l'aparença dels ítems de menú existents sense recrear tota l'estructura.

Exemple: Preparació Dinàmica (Kotlin)
```Kotlin
// Variable d'estat a l'Activity
private var isUserLoggedIn = false

override fun onPrepareOptionsMenu(menu: Menu?): Boolean {
    val loginItem = menu?.findItem(R.id.action_login)
    val logoutItem = menu?.findItem(R.id.action_logout)
    
    // Mostra l'opció de login si l'usuari no està connectat, i viceversa
    loginItem?.isVisible = !isUserLoggedIn
    logoutItem?.isVisible = isUserLoggedIn
    
    return super.onPrepareOptionsMenu(menu)
}
```
## 5. Action Views
Vegeu documentació per a detalls: https://developer.android.com/develop/ui/views/components/appbar/action-views

### Action Views
Un Action View és un widget personalitzat (per exemple, un EditText, un SearchView, o un ProgressBar) que s'associa a un ítem de menú. En lloc de mostrar la icona i el text habituals, l'ítem de menú mostra el widget de l'Action View directament a la Toolbar.
Característiques:
- Widget Incrustat: El SearchView és l'exemple més comú. Quan l'usuari el selecciona, el widget s'expandeix a la barra d'aplicacions, permetent interaccions complexes (escriure, enviar) sense canviar d'Activity o mostrar un diàleg.

- Implementació: Es defineix mitjançant l'atribut app:actionViewClass o android:actionLayout a l'XML del menú. També es pot inflar manualment en el mètode onCreateOptionsMenu().

- Gestió de Col·lapse: Quan l'Action View s'expandeix, altres accions de la Toolbar es poden col·lapsar (amagar). Pots implementar la interfície MenuItem.OnActionExpandListener per gestionar esdeveniments quan l'acció s'expandeix o es col·lapsa.

Searchview sense focus     |  Searchview amb focus
:-------------------------:|:-------------------------:
![Image 1](./Imatges/actionview1.png) | ![Image 2](./Imatges/actionview2.png)


## Resum de les funcion:

- onCreateOptionsMenu(): Inflar el menú
- onOptionsItemSelected(): Gestionar clics
- onPrepareOptionsMenu(): Actualitzar dinàmicament
- invalidateOptionsMenu(): Forçar recreació del menú

Utilitza AppBarLayout per a comportaments avançats (amagar/mostrar en scroll)
(veure a continuació)

## 6. Estil i Comportament Avançat (Material Design)
### Estil
L'estil de l'AppBar (colors, fons, elevació) es controla principalment a través del Tema de l'aplicació. Les propietats clau que afecten l'AppBar són:

- colorPrimary: El color de fons principal de la barra.

- colorPrimaryVariant: S'utilitza sovint per a la Barra d'Estat (Status Bar).

### Comportament Avançat amb CoordinatorLayout
Per a les aplicacions amb un disseny més sofisticat, la Toolbar s'integra amb un CoordinatorLayout i un AppBarLayout per crear efectes de desplaçament dinàmics.

#### **Exemple d'Efecte de Desplaçament:**
 Si col·loques la Toolbar dins d'un AppBarLayout i el contingut desplaçable (com un RecyclerView) també dins del CoordinatorLayout, pots utilitzar les scroll flags de l'AppBarLayout:

```XML
<androidx.coordinatorlayout.widget.CoordinatorLayout>
    <com.google.android.material.appbar.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
        <androidx.appcompat.widget.Toolbar
            ...
            app:layout_scrollFlags="scroll|enterAlways" /> </com.google.android.material.appbar.AppBarLayout>

    <androidx.recyclerview.widget.RecyclerView
        ...
        app:layout_behavior="@string/appbar_scrolling_view_behavior" />
</androidx.coordinatorlayout.widget.CoordinatorLayout>
```

**scroll:**
Fa que la barra es mogui fora de la pantalla quan l'usuari es desplaça cap avall.

**enterAlways:**
 Fa que la barra torni a aparèixer tan aviat com l'usuari comença a desplaçar-se cap amunt, fins i tot lleugerament.