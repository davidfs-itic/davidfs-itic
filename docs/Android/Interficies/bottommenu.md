## Menú de Navegació Inferior (BottomNavigationView)

El BottomNavigationView gestiona la navegació de nivell superior entre un nombre reduït de vistes principals.

La seva funció principal és la navegació entre fragments o activities.

Vegeu la referència en Material 3 aquí:

https://m3.material.io/components/navigation-bar/overview

I aneu a l'apartat Implementation: [MDC-Android (pàgina de Github)](https://github.com/material-components/material-components-android/blob/master/docs/components/BottomNavigation.md)


### XML de Menú de la Navegació Inferior

Per definir un menú, cal afegir-lo a la carpeta menú, en l'apartat recursos.
(si no existeix, cal crear-lo fent un nou recurs de tipus menú)

![Creació del recurs menú](./Imatges/crearmenu.png)


Exemple: Recurs de Menú (res/menu/bottom_nav_menu.xml)
```XML
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:id="@+id/home_fragment"
        android:icon="@drawable/ic_home"
        android:title="Inici" />
    <item
        android:id="@+id/dashboard_fragment"
        android:icon="@drawable/ic_dashboard"
        android:title="Panell" />
    <item
        android:id="@+id/notifications_fragment"
        android:icon="@drawable/ic_notifications"
        android:title="Avisos" />
</menu>
```
Exemple: Layout de l'activity (fixeu-vos en la linia ressaltada)
Es vincula el component amb el menú. (també es pot fer per programació)
```XML hl_lines="26"
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <androidx.fragment.app.FragmentContainerView
        android:id="@+id/fragment_container"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toTopOf="@id/bottom_navigation"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

    <com.google.android.material.bottomnavigation.BottomNavigationView
        android:id="@+id/bottom_navigation"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:background="?android:attr/windowBackground"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:menu="@menu/bottom_nav_menu" /> 
    </androidx.constraintlayout.widget.ConstraintLayout>
```


Exemple: Gestió Manual de la Navegació (Kotlin).
Cal afegir un listener al component, tal i com fariem amb un botó.

En aquest cas estem utiitzant fragments, però és el mateix amb activities, simplement faríem la gestió de la navegació amb intents i startactivity

```Kotlin 

// ...
val bottomNav: BottomNavigationView = findViewById(R.id.bottom_navigation)

bottomNav.setOnItemSelectedListener { item ->
    val selectedFragment: Fragment? = when (item.itemId) {
        R.id.home_fragment -> HomeFragment()
        R.id.dashboard_fragment -> DashboardFragment()
        R.id.notifications_fragment -> NotificationsFragment()
        else -> null
    }
    
    // Realitza la transacció de Fragment manualment
    if (selectedFragment != null) {
        supportFragmentManager.beginTransaction()
            .replace(R.id.fragment_container, selectedFragment)
            .commit()
    }
    true // Indica que la selecció ha estat gestionada
}
// ...
```

