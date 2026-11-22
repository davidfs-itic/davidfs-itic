# Menús

Un menú és una llista d'opcions que l'usuari pot triar per executar una acció o per navegar a una altra pantalla. A Android Views, tots els menús es defineixen de la mateixa manera: amb un recurs XML a la carpeta `res/menu/`. El que canvia d'un tipus de menú a un altre és on es mostra i com es gestiona el clic.

Aquest document fa de punt de partida: presenta tots els tipus de menú, explica l'estructura comuna del recurs XML i desenvolupa els casos que no estan explicats en altres documents.

Documentació oficial:

- [Menus](https://developer.android.com/develop/ui/views/components/menus)
- [MenuProvider](https://developer.android.com/reference/androidx/core/view/MenuProvider)
- [ActionMode](https://developer.android.com/reference/androidx/appcompat/view/ActionMode)

## 1. Tipus de menús

| Tipus | Quan es fa servir | On s'explica |
|---|---|---|
| Menú d'opcions | Accions generals de la pantalla, a la barra superior | [Toolbar](./toolbar.md#2-afegir-menus-a-la-toolbar) |
| Submenú | Agrupar opcions dins d'un element de la barra | [RecyclerView filtrat](./recyclerviewfilter.md#3-filtrar-amb-un-submenu) |
| PopupMenu | Desplegable enganxat a qualsevol View, obert des del codi | [RecyclerView filtrat](./recyclerviewfilter.md#4-filtrar-amb-un-popupmenu) |
| Menú contextual flotant | Accions sobre una View concreta, amb pulsació llarga | [Secció 4](#4-menu-contextual-flotant) |
| Mode d'acció contextual | Accions sobre elements seleccionats, en una barra temporal | [Secció 5](#5-mode-daccio-contextual) |
| Navegació inferior | Canviar entre les 3-5 pantalles principals | [Bottom Menu](./bottommenu.md) |
| Navigation Drawer | Panell lateral amb moltes destinacions | [Navigation Drawer](./drawermenu.md) |

!!! info "Menús d'acció i menús de navegació"
    Els cinc primers tipus són **menús d'acció**: l'opció triada executa alguna cosa (esborrar, compartir, filtrar...). La navegació inferior i el Navigation Drawer són **menús de navegació**: l'opció triada canvia la pantalla que es mostra. Tots fan servir el mateix format XML, però s'associen a components diferents.

## 2. Anatomia del recurs XML de menú

Un recurs de menú té tres elements:

- `<menu>`: l'arrel del fitxer. També es pot niar dins d'un `<item>` per crear un submenú.
- `<item>`: cada opció del menú.
- `<group>`: agrupa diversos `<item>` per aplicar-los propietats comunes (visibilitat, selecció...).

### Atributs principals de `<item>`

| Atribut | Funció |
|---|---|
| `android:id` | Identificador per saber quina opció s'ha triat |
| `android:title` | Text de l'opció |
| `android:icon` | Icona (només es veu si l'element surt a la barra) |
| `app:showAsAction` | Si l'element surt a la barra o al menú de desbordament (vegeu més avall) |
| `android:orderInCategory` | Ordre de l'element: els números més baixos van primer |
| `android:checkable` | L'element es pot marcar i desmarcar (checkbox) |
| `android:checked` | Estat inicial de la marca |
| `android:enabled` | Si és `false`, l'element es veu en gris i no es pot prémer |
| `android:visible` | Si és `false`, l'element no es mostra |

Valors de `app:showAsAction` (només té efecte al menú d'opcions de la barra):

- `always`: sempre a la barra.
- `ifRoom`: a la barra si hi ha espai; si no, al menú de desbordament (els tres punts).
- `never`: sempre al menú de desbordament.
- `withText`: a la barra, mostra el text al costat de la icona si hi ha espai. Es combina amb un altre valor: `ifRoom|withText`.
- `collapseActionView`: l'element té una [action view](./toolbar.md#5-action-views) que es pot plegar. Es combina amb un altre valor: `ifRoom|collapseActionView`.

!!! warning "El namespace `app:`"
    Amb AppCompat (que és el que fem servir sempre), `showAsAction` va amb el prefix `app:`, no `android:`. Cal declarar `xmlns:app="http://schemas.android.com/apk/res-auto"` a l'element `<menu>`. Si poseu `android:showAsAction`, s'ignora i l'element va al menú de desbordament.

### Exemple complet

Fitxer `res/menu/menu_nota.xml`:

```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <item
        android:id="@+id/action_compartir"
        android:icon="@drawable/ic_share"
        android:title="Compartir"
        android:orderInCategory="1"
        app:showAsAction="ifRoom" />

    <item
        android:id="@+id/action_important"
        android:title="Important"
        android:checkable="true"
        android:checked="false"
        android:orderInCategory="2"
        app:showAsAction="never" />

    <item
        android:id="@+id/action_esborrar"
        android:title="Esborrar"
        android:orderInCategory="3"
        app:showAsAction="never" />

</menu>
```

Per a grups amb una sola opció marcada (`<group android:checkableBehavior="single">`), vegeu l'exemple del [submenú de categories](./recyclerviewfilter.md#3-filtrar-amb-un-submenu).

### Elements marcables

Android dibuixa la marca d'un element `checkable`, però **no la canvia sol** quan l'usuari el prem. L'hem de canviar nosaltres en gestionar el clic:

```kotlin
override fun onOptionsItemSelected(item: MenuItem): Boolean {
    return when (item.itemId) {
        R.id.action_important -> {
            // Inverteix l'estat de la marca
            item.isChecked = !item.isChecked
            Toast.makeText(this, "Important: ${item.isChecked}", Toast.LENGTH_SHORT).show()
            true
        }
        else -> super.onOptionsItemSelected(item)
    }
}
```

## 3. Menús en Fragments: MenuProvider

A [Toolbar](./toolbar.md#2-afegir-menus-a-la-toolbar) hem vist com afegir el menú des d'una Activity amb `onCreateOptionsMenu()` i `onOptionsItemSelected()`. Quan l'aplicació fa servir Fragments (per exemple, amb una [navegació inferior](./bottommenu.md)), sovint cada Fragment vol posar les seves pròpies opcions a la barra.

### Per què no fer servir `setHasOptionsMenu()`

Abans, els Fragments cridaven `setHasOptionsMenu(true)` i sobreescrivien `onCreateOptionsMenu()`. Aquests mètodes estan obsolets (*deprecated*). Ara es fa servir la interfície `MenuProvider`, que s'afegeix a l'Activity i queda lligada al cicle de vida del Fragment.

### Els mètodes de MenuProvider

Són els equivalents dels mètodes de l'Activity:

| Activity | MenuProvider |
|---|---|
| `onCreateOptionsMenu()` | `onCreateMenu()` |
| `onOptionsItemSelected()` | `onMenuItemSelected()` |
| `onPrepareOptionsMenu()` | `onPrepareMenu()` (opcional) |
| `invalidateOptionsMenu()` | `requireActivity().invalidateMenu()` |

### Exemple

```kotlin
import androidx.core.view.MenuProvider
import androidx.lifecycle.Lifecycle

class HomeFragment : Fragment(R.layout.fragment_home) {

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        requireActivity().addMenuProvider(object : MenuProvider {

            // Infla el menú del Fragment
            override fun onCreateMenu(menu: Menu, menuInflater: MenuInflater) {
                menuInflater.inflate(R.menu.menu_home, menu)
            }

            // Gestiona els clics
            override fun onMenuItemSelected(menuItem: MenuItem): Boolean {
                return when (menuItem.itemId) {
                    R.id.action_compartir -> {
                        Toast.makeText(requireContext(), "Compartir", Toast.LENGTH_SHORT).show()
                        true
                    }
                    else -> false // No l'hem gestionat: que el gestioni un altre
                }
            }

        }, viewLifecycleOwner, Lifecycle.State.RESUMED)
    }
}
```

Els dos últims paràmetres de `addMenuProvider` són els importants:

- `viewLifecycleOwner`: el menú queda lligat a la vista del Fragment. Quan el Fragment es substitueix per un altre, el menú s'elimina automàticament. No cal fer res per treure'l.
- `Lifecycle.State.RESUMED`: el menú només es mostra mentre el Fragment és visible i actiu.

!!! info "Menú de l'Activity i menú del Fragment"
    Si l'Activity també té un menú, Android combina els dos: a la barra es veuen les opcions de l'Activity i les del Fragment actiu. Per això, a `onMenuItemSelected` cal retornar `false` per als elements que no són del Fragment, perquè l'Activity tingui l'oportunitat de gestionar-los.

!!! warning "Cal una barra superior"
    Igual que a l'Activity, el menú només apareix si l'Activity té una Toolbar configurada amb `setSupportActionBar`. Vegeu [La Toolbar com a AppBar](./toolbar.md#1-la-toolbar-com-a-appbar-fonaments).

## 4. Menú contextual flotant

El menú contextual flotant apareix quan l'usuari fa una **pulsació llarga** sobre una View. Es mostra com una llista al centre de la pantalla, per sobre del contingut, i serveix per oferir accions sobre aquella View concreta (copiar un text, esborrar una nota...).

### Com funciona

Normalment, una pulsació llarga no fa res. Amb `registerForContextMenu(view)` li diem a Android que, quan l'usuari mantingui premuda aquella View, ha d'obrir un menú contextual. A partir d'aquí, Android crida dos mètodes que hem de sobreescriure:

| Menú d'opcions (barra) | Menú contextual |
|---|---|
| `onCreateOptionsMenu()` | `onCreateContextMenu()` |
| `onOptionsItemSelected()` | `onContextItemSelected()` |

A diferència del menú d'opcions, `onCreateContextMenu()` es crida **cada vegada** que s'obre el menú.

### Definició del menú (XML)

Fitxer `res/menu/menu_context.xml`:

```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:id="@+id/action_copiar"
        android:title="Copiar" />
    <item
        android:id="@+id/action_esborrar"
        android:title="Esborrar" />
</menu>
```

Els menús contextuals no mostren icones ni fan servir `showAsAction`.

### Activity

```kotlin
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val text = findViewById<TextView>(R.id.text_nota)
        // A partir d'ara, una pulsació llarga sobre el TextView obre el menú
        registerForContextMenu(text)
    }

    // Es crida cada vegada que s'obre el menú (pulsació llarga)
    override fun onCreateContextMenu(
        menu: ContextMenu,
        v: View,
        menuInfo: ContextMenu.ContextMenuInfo?
    ) {
        super.onCreateContextMenu(menu, v, menuInfo)
        menuInflater.inflate(R.menu.menu_context, menu)
    }

    // Es crida quan l'usuari tria una opció
    override fun onContextItemSelected(item: MenuItem): Boolean {
        return when (item.itemId) {
            R.id.action_copiar -> {
                Toast.makeText(this, "Copiat", Toast.LENGTH_SHORT).show()
                true
            }
            R.id.action_esborrar -> {
                Toast.makeText(this, "Esborrat", Toast.LENGTH_SHORT).show()
                true
            }
            else -> super.onContextItemSelected(item)
        }
    }
}
```

### Diverses Views amb menús diferents

Es poden registrar tantes Views com calgui. El paràmetre `v` de `onCreateContextMenu` indica quina View s'ha premut, i així podem inflar un menú diferent per a cadascuna:

```kotlin
override fun onCreateContextMenu(
    menu: ContextMenu,
    v: View,
    menuInfo: ContextMenu.ContextMenuInfo?
) {
    super.onCreateContextMenu(menu, v, menuInfo)
    when (v.id) {
        R.id.text_nota -> menuInflater.inflate(R.menu.menu_context, menu)
        R.id.image_foto -> menuInflater.inflate(R.menu.menu_context_foto, menu)
    }
}
```

### Menú contextual o PopupMenu?

| | Menú contextual | PopupMenu |
|---|---|---|
| Qui l'obre | Android, amb la pulsació llarga | Nosaltres, des del codi |
| On apareix | Al centre de la pantalla | Enganxat a la View |
| On es captura l'opció | `onContextItemSelected` | Listener del `PopupMenu` |

!!! warning "Menús contextuals en un RecyclerView"
    Si cridem `registerForContextMenu(recyclerView)`, el menú s'obre, però a `onContextItemSelected` no sabem sobre quin element s'ha fet la pulsació llarga, i esbrinar-ho requereix força feina extra. És més senzill posar un `setOnLongClickListener` al ViewHolder i obrir-hi un [PopupMenu](./recyclerviewfilter.md#4-filtrar-amb-un-popupmenu): allà ja tenim la posició de l'element.

## 5. Mode d'acció contextual

El mode d'acció contextual (*Contextual Action Mode*) mostra una barra temporal que **cobreix la Toolbar** amb accions sobre el que l'usuari ha seleccionat. És el que veiem, per exemple, a l'aplicació de missatges quan mantenim premut un missatge: la barra canvia de color i mostra icones per esborrar, copiar o reenviar.

La barra té un botó per tancar-la (a l'esquerra) i desapareix quan l'usuari tria una acció o prem el botó Enrere.

### ActionMode.Callback

El mode d'acció es gestiona amb un objecte `ActionMode.Callback`, que té quatre mètodes:

- `onCreateActionMode()`: es crida en obrir el mode d'acció. Hi inflem el menú.
- `onPrepareActionMode()`: es crida després de crear-lo i cada vegada que s'invalida. Retornem `false` si no hem canviat res.
- `onActionItemClicked()`: es crida quan l'usuari prem una acció.
- `onDestroyActionMode()`: es crida en tancar el mode d'acció. Hi netegem l'estat (per exemple, desmarcar la selecció).

### Definició del menú (XML)

Fitxer `res/menu/menu_action_mode.xml`. Aquí sí que fem servir icones i `showAsAction`, perquè les accions es mostren a la barra:

```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <item
        android:id="@+id/action_copiar"
        android:icon="@drawable/ic_copy"
        android:title="Copiar"
        app:showAsAction="ifRoom" />

    <item
        android:id="@+id/action_esborrar"
        android:icon="@drawable/ic_delete"
        android:title="Esborrar"
        app:showAsAction="ifRoom" />

</menu>
```

### Activity

En aquest exemple, una pulsació llarga sobre un TextView el selecciona (canvia el color de fons) i obre el mode d'acció:

```kotlin
import androidx.appcompat.view.ActionMode

class MainActivity : AppCompatActivity() {

    // Guardem el mode d'acció actiu per no obrir-ne dos alhora
    private var actionMode: ActionMode? = null
    private lateinit var text: TextView

    private val actionModeCallback = object : ActionMode.Callback {

        override fun onCreateActionMode(mode: ActionMode, menu: Menu): Boolean {
            mode.menuInflater.inflate(R.menu.menu_action_mode, menu)
            mode.title = "1 seleccionat"
            return true
        }

        override fun onPrepareActionMode(mode: ActionMode, menu: Menu): Boolean {
            return false // No hem canviat res
        }

        override fun onActionItemClicked(mode: ActionMode, item: MenuItem): Boolean {
            return when (item.itemId) {
                R.id.action_copiar -> {
                    Toast.makeText(this@MainActivity, "Copiat", Toast.LENGTH_SHORT).show()
                    mode.finish() // Tanca el mode d'acció
                    true
                }
                R.id.action_esborrar -> {
                    text.text = ""
                    mode.finish()
                    true
                }
                else -> false
            }
        }

        override fun onDestroyActionMode(mode: ActionMode) {
            // Treu la selecció i oblida el mode d'acció
            text.setBackgroundColor(Color.TRANSPARENT)
            actionMode = null
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        setSupportActionBar(findViewById(R.id.my_toolbar))

        text = findViewById(R.id.text_nota)
        text.setOnLongClickListener {
            if (actionMode == null) {
                text.setBackgroundColor(Color.LTGRAY) // Marca la selecció
                actionMode = startSupportActionMode(actionModeCallback)
            }
            true // Hem gestionat la pulsació llarga
        }
    }
}
```

Punts a destacar:

- `startSupportActionMode(callback)` obre el mode d'acció i retorna l'objecte `ActionMode`. L'hem d'importar de `androidx.appcompat.view`, no de `android.view`.
- `mode.finish()` tanca el mode d'acció des del codi. Després, Android crida `onDestroyActionMode`.
- `mode.title` permet mostrar un text a la barra, per exemple quants elements hi ha seleccionats.
- Guardem `actionMode` per no obrir-ne un de nou si l'usuari torna a fer una pulsació llarga mentre ja està obert.

!!! info "La barra apareix a sobre de la Toolbar"
    Quan fem servir una Toolbar pròpia (tema `NoActionBar` + `setSupportActionBar`), pot passar que la barra del mode d'acció aparegui **a sobre** de la Toolbar i empenyi el contingut cap avall, en lloc de cobrir-la. Per evitar-ho, afegiu aquest element al tema de l'aplicació (`res/values/themes.xml`):

    ```xml
    <item name="windowActionModeOverlay">true</item>
    ```

### Menú contextual o mode d'acció?

- **Menú contextual flotant**: accions puntuals sobre un sol element, sense selecció visible.
- **Mode d'acció contextual**: quan l'usuari selecciona un o més elements i després hi aplica una acció. És el patró recomanat per Material Design per a la selecció múltiple.

## 6. Resum

| Tipus de menú | Com es crea | On es gestiona el clic |
|---|---|---|
| Menú d'opcions (Activity) | `onCreateOptionsMenu()` | `onOptionsItemSelected()` |
| Menú d'opcions (Fragment) | `addMenuProvider()` → `onCreateMenu()` | `onMenuItemSelected()` |
| Submenú | `<menu>` niat dins d'un `<item>` | `onOptionsItemSelected()` |
| PopupMenu | `PopupMenu(context, view)` + `show()` | `setOnMenuItemClickListener` |
| Menú contextual flotant | `registerForContextMenu()` + `onCreateContextMenu()` | `onContextItemSelected()` |
| Mode d'acció contextual | `startSupportActionMode(callback)` | `onActionItemClicked()` |
| Navegació inferior | `app:menu` al `BottomNavigationView` | `setOnItemSelectedListener` |
| Navigation Drawer | `app:menu` al `NavigationView` | `setNavigationItemSelectedListener` |
