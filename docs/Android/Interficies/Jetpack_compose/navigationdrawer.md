# NavigationDrawer

`NavigationDrawer` és el panell de navegació lateral que s'obre lliscant des del cantó de la pantalla, habitualment amb la llista de seccions principals de l'aplicació. Equival al `DrawerLayout` del sistema de Views, però a diferència d'un `bottomBar` o una `topBar`, no és un slot de `Scaffold`: és un composable independent que **embolcalla** tota la pantalla (incloent-hi el seu propi `Scaffold`) per poder-se dibuixar per sobre de la resta de contingut.

Documentació oficial: https://developer.android.com/develop/ui/compose/components/drawer

## 1. Concepte: `ModalNavigationDrawer` embolcalla el contingut

El component més habitual és `ModalNavigationDrawer`. Accepta dos paràmetres principals: `drawerContent` (el contingut del panell lateral) i `content` (la resta de la pantalla, típicament el `Scaffold` complet):

```kotlin
@Composable
fun PantallaAmbDrawer() {
    ModalNavigationDrawer(
        drawerContent = {
            ModalDrawerSheet {
                Text("Menú", modifier = Modifier.padding(16.dp))
            }
        }
    ) {
        Scaffold(
            topBar = { TopAppBar(title = { Text("La meva app") }) }
        ) { innerPadding ->
            // contingut principal
        }
    }
}
```

Aquesta estructura (`ModalNavigationDrawer` per fora, `Scaffold` per dins) és la inversa del que es podria esperar: el drawer no és un element "dins" de la pantalla, sinó que la pantalla sencera és el `content` del drawer, perquè el panell ha de poder-se dibuixar per sobre de tot, tapant temporalment la resta de la interfície.

## 2. Estat del drawer: `DrawerState`

Igual que amb el `DropdownMenu` (vegeu [DropdownMenu](./dropdownmenu.md)), el drawer necessita un estat que indiqui si està obert o tancat. En aquest cas, en lloc d'un simple `Boolean`, es fa servir `DrawerState`, creat amb `rememberDrawerState`:

```kotlin
val drawerState = rememberDrawerState(initialValue = DrawerValue.Closed)

ModalNavigationDrawer(
    drawerState = drawerState,
    drawerContent = { ModalDrawerSheet { /* ... */ } }
) {
    // content
}
```

`DrawerValue` només té dos valors possibles, `Closed` i `Open`, però `DrawerState` no és un booleà pla perquè també guarda informació de l'animació (per exemple, la fracció d'obertura mentre l'usuari arrossega el dit). Per obrir o tancar el drawer per codi (per exemple, en prémer una icona de menú a la `topBar`) es criden les seves funcions `open()`/`close()`, que són `suspend` i, per tant, s'han de llançar des d'una coroutine amb `rememberCoroutineScope`:

```kotlin
val drawerState = rememberDrawerState(initialValue = DrawerValue.Closed)
val scope = rememberCoroutineScope()

ModalNavigationDrawer(
    drawerState = drawerState,
    drawerContent = { ModalDrawerSheet { /* ... */ } }
) {
    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("La meva app") },
                navigationIcon = {
                    IconButton(onClick = { scope.launch { drawerState.open() } }) {
                        Icon(Icons.Default.Menu, contentDescription = "Obrir menú")
                    }
                }
            )
        }
    ) { innerPadding ->
        // contingut principal
    }
}
```

Aquest ús de `rememberCoroutineScope` és exactament el mateix patró que el `Snackbar` de [Scaffold](./scaffold.md#5-snackbarhost): una acció que es dispara com a resposta directa a un clic, no automàticament.

## 3. Contingut del drawer: `ModalDrawerSheet` i `NavigationDrawerItem`

`ModalDrawerSheet` proporciona el fons, l'amplada i l'elevació estàndard del panell lateral segons Material Design. A dins, cada opció del menú es representa amb `NavigationDrawerItem`, que segueix el mateix patró controlat vist a altres components de selecció: `selected` i `onClick`.

```kotlin
val opcions = listOf("Inici", "Perfil", "Ajustos")
var opcioSeleccionada by remember { mutableStateOf(opcions[0]) }

ModalDrawerSheet {
    Text("La meva app", modifier = Modifier.padding(16.dp))
    HorizontalDivider()
    opcions.forEach { opcio ->
        NavigationDrawerItem(
            label = { Text(opcio) },
            selected = (opcio == opcioSeleccionada),
            icon = { Icon(Icons.Default.Star, contentDescription = null) },
            onClick = {
                opcioSeleccionada = opcio
                scope.launch { drawerState.close() }
            }
        )
    }
}
```

Igual que a l'exemple del `RadioButton` (vegeu [Switch, Checkbox i RadioButton](./switchcheckboxradiobutton.md)), `opcioSeleccionada` és l'única font de veritat: cada `NavigationDrawerItem` només compara si ell mateix és l'opció actual. En seleccionar una opció, és habitual tancar el drawer immediatament (`drawerState.close()`) perquè l'usuari pugui veure el contingut de la secció triada.

## 4. Variants: `DismissibleNavigationDrawer` i `PermanentNavigationDrawer`

`ModalNavigationDrawer` és pensat per a mòbil: el panell es dibuixa per sobre del contingut i el tapa amb una capa fosca (*scrim*) mentre està obert. En pantalles més amples (tauletes, plegables en mode obert), Compose ofereix dues variants amb la mateixa API de `drawerContent`/`drawerState`:

- **`DismissibleNavigationDrawer`**: el panell empeny el contingut cap al costat en lloc de dibuixar-se per sobre, però es pot seguir obrint i tancant.
- **`PermanentNavigationDrawer`**: el panell és sempre visible, sense estat d'obert/tancat; útil per a disposicions d'escriptori o de pantalla ampla on hi ha espai permanent per a la navegació.

Triar una variant o una altra sol dependre de l'amplada de pantalla disponible (`WindowSizeClass`), més que d'una preferència fixa de disseny.

## 5. Quan fer-lo servir

Un `NavigationDrawer` té sentit quan l'aplicació té diverses seccions de primer nivell (més de les que caben còmodament en una `NavigationBar` inferior) o quan es vol reservar la part inferior de la pantalla per a altres controls. Per a un nombre reduït de seccions (3-5), sol ser preferible la `NavigationBar` vista a [Scaffold](./scaffold.md#3-bottombar-i-floatingactionbutton), ja que és sempre visible i no requereix cap gest per descobrir-la.
