## 1. Drawer Menú 


Es presenta com un panell vertical que roman ocult i es fa visible lliscant-lo (normalment des de la vora esquerra de la pantalla, depenent de la configuració de l'idioma) o prement la icona de menú/hamburguesa a la barra superior de l'aplicació.

La seva funció principal és estalviar espai a la pantalla i mantenir la interfície neta, ja que agrupa les opcions de navegació menys freqüentment utilitzades o que porten a pantalles de nivell superior, proporcionant un únic punt de salt a qualsevol secció principal de l'aplicació.




## 2. Implementació

### Recurs de Menú (XML)
Crea un nou fitxer de recurs de menú a res/menu/main_drawer_menu.xml. Aquest fitxer defineix els ítems que es mostraran al panell lateral.
```XML
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <group android:checkableBehavior="single">
        <item
            android:id="@+id/nav_home"
            android:icon="@drawable/ic_home"
            android:title="Inici" />
        <item
            android:id="@+id/nav_gallery"
            android:icon="@drawable/ic_gallery"
            android:title="Galeria" />
        <item
            android:id="@+id/nav_settings"
            android:icon="@drawable/ic_settings"
            android:title="Configuració" />
    </group>
    <item android:title="Comunicació">
        <menu>
            <item
                android:id="@+id/nav_share"
                android:icon="@drawable/ic_share"
                android:title="Compartir" />
        </menu>
    </item>
</menu>
```
### Disseny del Layout
El DrawerLayout és el contenidor principal. Ha de contenir dos elements: el contingut principal de l'aplicació i el panell de navegació.

### 2.1 activity_main.xml
```XML
<androidx.drawerlayout.widget.DrawerLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true" 
    tools:openDrawer="start">
    
    <include
        layout="@layout/app_bar_main"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
        
    <com.google.android.material.navigation.NavigationView
        android:id="@+id/nav_view"
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:layout_gravity="start" 
        android:fitsSystemWindows="true"
        app:headerLayout="@layout/nav_header_main" 
        app:menu="@menu/main_drawer_menu" />

</androidx.drawerlayout.widget.DrawerLayout>
```
- DrawerLayout: L'arrel.12android:layout_gravity="start": Crucial. Indica al DrawerLayout que el NavigationView és el panell lateral i ha d'aparèixer des de la vora inicial (esquerra, en la majoria d'idiomes).
- app:menu: Enllaça amb el fitxer de menú XML creat anteriorment.

## 2.2 Implementació de la Lògica
La lògica es realitza a la MainActivity.kt. 

Es centra a sincronitzar el Drawer amb la barra d'eines i gestionar els clics.

Exemple:
```Kotlin
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.view.MenuItem
import android.widget.Toast
import androidx.appcompat.app.ActionBarDrawerToggle
import androidx.core.view.GravityCompat
import com.google.android.material.navigation.NavigationView
import androidx.drawerlayout.widget.DrawerLayout

class MainActivity : AppCompatActivity(), 
                     NavigationView.OnNavigationItemSelectedListener { // 1. Implementa el Listener

    private lateinit var drawerLayout: DrawerLayout

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // 2. Obtenció de les vistes
        drawerLayout = findViewById(R.id.drawer_layout)
        val navView: NavigationView = findViewById(R.id.nav_view)
        
        // La Toolbar (assumint que s'ha inclòs a app_bar_main i se li ha assignat un ID 'toolbar')
        val toolbar: androidx.appcompat.widget.Toolbar = findViewById(R.id.toolbar)
        setSupportActionBar(toolbar) // Opcional: Si vols usar la Toolbar com a ActionBar
        
        // 3. Sincronització amb la Barra d'Eines (Icona d'Hamburguesa)
        val toggle = ActionBarDrawerToggle(
            this, 
            drawerLayout, 
            toolbar, 
            R.string.navigation_drawer_open, // String de recurs: "Open navigation drawer"
            R.string.navigation_drawer_close // String de recurs: "Close navigation drawer"
        )
        drawerLayout.addDrawerListener(toggle)
        toggle.syncState() // Sincronitza l'estat inicial de la icona

        // 4. Establir el Listener per als Clics al Menú
        navView.setNavigationItemSelectedListener(this)
    }

    // 5. Gestió dels Clics al Menú (Implementació del Listener)
    override fun onNavigationItemSelected(item: MenuItem): Boolean {
        when (item.itemId) {
            R.id.nav_home -> {
                Toast.makeText(this, "Inici seleccionat", Toast.LENGTH_SHORT).show()
                // Aquí va la lògica per canviar Fragment / Activity
            }
            R.id.nav_gallery -> {
                Toast.makeText(this, "Galeria seleccionada", Toast.LENGTH_SHORT).show()
            }
            R.id.nav_settings -> {
                Toast.makeText(this, "Configuració seleccionada", Toast.LENGTH_SHORT).show()
            }
            R.id.nav_share -> {
                Toast.makeText(this, "Compartir seleccionat", Toast.LENGTH_SHORT).show()
            }
        }
        
        // Tanca el Drawer un cop s'ha seleccionat l'ítem
        drawerLayout.closeDrawer(GravityCompat.START)
        return true
    }

    // 6. Gestió del Botó Enrere
    override fun onBackPressed() {
        if (drawerLayout.isDrawerOpen(GravityCompat.START)) {
            // Si el Drawer està obert, el tanquem
            drawerLayout.closeDrawer(GravityCompat.START)
        } else {
            // Si el Drawer està tancat, es permet la funció normal del botó Enrere
            super.onBackPressed()
        }
    }
}
```
## 3. Resum de Punts Clau

Component | Funció | Notes Clau
:-------------------------|:-------------|:------------
DrawerLayout | Contenidor arrel que permet l'acció de lliscament. | Ha de tenir dos fills: el contingut principal i el panell lateral.
NavigationView| El panell lateral amb el menú d'elements.|Utilitza android:layout_gravity="start" i app:menu.
ActionBarDrawerToggle | Classe que gestiona la interacció amb la Toolbar. | S'encarrega de mostrar la icona d'hamburguesa i obrir/tancar el Drawer al clicar.
onNavigationItemSelected| Mètode del NavigationView.| 
OnNavigationItemSelectedListener. | On s'implementa la lògica de navegació real (Fragments, Activities). | S'ha de tancar el Drawer després de l'acció.
onBackPressed() | Mètode de l'Activity. | Es sobreescriu per interceptar el botó Enrere i tancar el Drawer abans de sortir de l'Activity.

