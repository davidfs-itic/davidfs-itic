# Menus

## app bar


## Toolbar (app bar personalitzada)


## botton menu
Anem a la web de material: i busquem el component. Navigation-NavigationBar:
https://m3.material.io/components/navigation-bar/overview

Allà a la part de resources, podem anar a MDC-Android (el git on explica com funciona):
https://github.com/material-components/material-components-android/blob/master/docs/components/BottomNavigation.md


### Preparem menú, icones, 

Fem un menú amb 4 opcions, (creem les icones a drawable→vector)

Si no existeix, creem la carpeta menu en l'apartat res
Afegim un menú i editem els ítems:

```
<item
    android:icon="@drawable/icon_home"
    android:title="Inici"
    android:id="@+id/main_bottom_menu_home"/>
```
