# Navigation Compose

Navigation Compose és la llibreria que gestiona la navegació entre pantalles en una app construïda amb Jetpack Compose. En l'arquitectura *Single Activity* (vegeu [Introducció a Jetpack Compose](./index.md#2-arquitectura-una-sola-activity)), ja no hi ha una Activity o un Fragment per pantalla: totes les "pantalles" són funcions `@Composable`, i és Navigation Compose qui decideix quina d'elles es mostra en cada moment, mantenint una pila (*back stack*) de destinacions.

Documentació oficial: https://developer.android.com/develop/ui/compose/navigation

## 1. Concepte: navegar és canviar de composable

Sense una llibreria de navegació, mostrar una pantalla o una altra es podria simular amb un simple `if` o un `when` sobre una variable d'estat. Navigation Compose fa exactament això per sota, però hi afegeix el que aquesta solució manual no té: una pila de destinacions amb historial (per poder tornar enrere), pas d'arguments tipats entre pantalles, i integració amb el botó de retrocés del sistema.

Els tres elements centrals són:

- **`NavController`**: l'objecte que coneix l'estat de la navegació i que es fa servir per demanar un canvi de pantalla.
- **`NavHost`**: el composable contenidor que mostra la destinació activa.
- **Destinacions**: cada pantalla, associada a una ruta.

## 2. Dependències necessàries

Afegir les dependències mínimes en el **build.gradle** (build.gradle Module), tal com indica la documentació oficial:

```kotlin
dependencies {
    // Jetpack Compose integration
    implementation("androidx.navigation:navigation-compose:2.9.8")
    // JSON serialization library, works with the Kotlin serialization plugin
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.7.3")
}
```

La llibreria `kotlinx-serialization-json` és necessària per fer servir rutes tipades amb `@Serializable` (vegeu punt 4), i requereix afegir també el plugin de serialització de Kotlin:

```kotlin
// build.gradle (Module), bloc plugins
plugins {
    kotlin("plugin.serialization") version "2.0.21"
}
```

## 3. Destinacions, `NavController` i `NavHost`

Des de Navigation 2.8, l'enfocament recomanat és definir cada destinació com una classe Kotlin marcada amb `@Serializable`, en lloc d'una ruta de text:

```kotlin

@Serializable
object ToLoginScreen

@Serializable
object ToMyListScreen

@Serializable
data class Item(
    val id: Int, val nom: String, val descripcio: String
)
```

Un objecte (`object`) es fa servir per a destinacions sense arguments, i una `data class` per a destinacions que en reben. 

Al `NavHost`, cada destinació es registra amb `composable<Ruta>` (vegeu l'exemple del punt anterior).

Aquest enfocament substitueix el que trobareu en molta documentació més antiga, on cada ruta és un `String` (per exemple `"detall/{id}"`) i els arguments es declaren a part amb `navArgument`. Amb rutes de text, tant construir la ruta com llegir l'argument es fa sense cap comprovació del compilador. Amb rutes tipades, el compilador verifica que els tipus i els noms dels arguments són correctes.


El `NavController` es crea amb `rememberNavController()` a l'arrel de la jerarquia de composables (típicament dins del composable principal cridat des de `MainActivity`), perquè ha de sobreviure a les recomposicions i ser accessible des de qualsevol pantalla que necessiti navegar.

El `NavHost` és el composable que ocupa l'espai on es dibuixa la destinació activa. Rep el `NavController` i la destinació inicial (`startDestination`):

```kotlin
@Composable
fun AppNavigation() {
    val navController = rememberNavController()

    NavHost(
        navController = navController,
        startDestination = ToLoginScreen
    ) {
        composable<ToLoginScreen> {
            MyLoginScreen()
        }
        composable<ToMyListScreen> {
            Mylist()
        }
        composable<Item> { backStackEntry ->
            val item: Item = backStackEntry.toRoute()
            DetallScreen(item)
        }
    }
}
```
Aquest codi ja funciona. Si es col·loca AppNavigation al MainActivity, carregarà MyLoginScreen() a causa del paràmetre startDestination. Però com anar de MyLoginScreen a MyList ?

## 4. Navegar: `navController.navigate(...)`

Per anar a una destinació, es crida `navigate` sobre el `NavController` passant-li la instància de la ruta:

```kotlin
navController.navigate(Detall(item))
```

Per tornar enrere (equivalent a prémer el botó físic/gestual d'enrere), es fa servir `popBackStack()`:

```kotlin
navController.popBackStack()
```

Algunes opcions habituals de `navigate`, indicades amb una lambda de configuració:

```kotlin
navController.navigate(Llista) {
    popUpTo(Llista) { inclusive = true }
    launchSingleTop = true
}
```

- **`popUpTo`**: elimina de la pila totes les destinacions fins a la indicada, útil per evitar acumular pantalles quan es torna a una secció ja visitada (per exemple, en pitjar un item de la `NavigationBar`).
- **`inclusive`**: si és `true`, també elimina la destinació indicada a `popUpTo`. D'aquesta manera només tindríem a la pila una pantalla. Si ho posem a false, eliminara totes les pantalles anteriors fins a la destinació, però no la destinació per la qual cosa tindríem 2 pantalles seguides iguals a la pila.
- **`launchSingleTop`**: evita crear una nova còpia de la destinació si ja és la que està al capdamunt de la pila.


## 5. Navegació sense arguments entre pantalles.

Hem vist les funcions que permeten navegar, però per a que des de la pantalla Login puguem anar a MyList, tenim 2 aproximacions:

La primera, passar el navController com a paràmetre al LoginScreen. Dins el codi de LoginScreen utilitzar-lo per a navegar. Aquesta opció no és molt neta, doncs implica que gairebé totes les pantalles haurien de tenir com argument el navController, i potser la pantalla tindria massa responsabilitat.

La segona, passar una funció lambda a LoginScreen "anarAMylist". Dins la pantalla, es cridarà a la funció on toqui, però tot el codi de navegació estarà junt amb el navigationHost i el navigationController a la funció AppNavigation:

```kotlin
@Composable
fun MyLoginScreen(toMyList:()->Unit){
    
    Column(modifier = Modifier.fillMaxSize(), horizontalAlignment = Alignment.CenterHorizontally, verticalArrangement = Arrangement.SpaceEvenly) {
        Spacer(modifier = Modifier.weight(1f))
        Text("LoginScreen", fontSize = 38.sp)
        Spacer(modifier = Modifier.weight(1f))
        Button(onClick = toMyList ){ Text("Login")}
        Spacer(modifier = Modifier.weight(1f))
    }
}
```

D'aquesta manera la navegació queda delegada a la funció AppNavigation:

```kotlin
fun AppNavigation() {
    val navController = rememberNavController()

    NavHost(
        navController = navController,
        startDestination = ToLoginScreen
    ) {
        composable<ToLoginScreen> {
            MyLoginScreen({navController.navigate(ToMyListScreen)})
        }
        composable<ToMyListScreen> {
            Mylist()
        }
        composable<Item> { backStackEntry ->
            val item: Item = backStackEntry.toRoute()
            DetallScreen(item)
        }
    }
    
}
```


## 6. Pas d'arguments entre pantalles

### Què és el `backStackEntry`

Cada vegada que es navega a una destinació, Navigation Compose crea un objecte `NavBackStackEntry` per representar **aquella instància concreta** de la destinació dins la pila. Aquest objecte és el que la lambda de `composable<Ruta>` rep com a paràmetre (l'hem anomenat `backStackEntry` als exemples anteriors), i és qui guarda, entre altres coses, els arguments amb què s'ha navegat fins allà. No és la ruta en si, sinó el "carnet" d'aquesta pantalla concreta mentre és a la pila: per això té mètodes com `toRoute<Ruta>()`, que llegeix aquests arguments i reconstrueix l'objecte de ruta original amb els seus camps ja tipats.

### Exemple: 

La forma més senzilla de passar dades a una pantalla de detall és fer servir directament una `data class` que ja representa l'element, en lloc de crear una classe de ruta a part que només porti un `id`. Només cal marcar-la amb `@Serializable` perquè es pugui fer servir com a destinació:

```kotlin
@Serializable
data class Item(
    val id: Int,
    val nom: String,
    val descripcio: String
)
```

Seguint el mateix patró del punt 5, passem una lambda `toDetall: (Item) -> Unit`, que es crida passant-li l'element sobre el qual s'ha fet clic:

```kotlin
@Composable
fun Mylist(items: List<Item>, toDetall: (Item) -> Unit) {
    LazyColumn {
        items(items) { item ->
            ItemCard(item = item, modifier = Modifier.clickable { toDetall(item) })
        }
    }
}
```

Igual que amb `MyLoginScreen` i `toMyList`, la navegació queda delegada a `AppNavigation`, que ara connecta la lambda `toDetall` amb `navController.navigate(item)`:

```kotlin
NavHost(navController = navController, startDestination = ToMyListScreen) {
    composable<ToMyListScreen> {
        Mylist(items = llistaItems, toDetall = { item -> navController.navigate(item) })
    }
    composable<Item> { backStackEntry ->
        val item: Item = backStackEntry.toRoute()
        DetallScreen(item)
    }
}
```

`Mylist` no necessita conèixer el `navController`, ni tan sols saber que existeix navegació: només indica *quin* element s'ha triat. És `AppNavigation` qui rep aquest `Item` a `toDetall` i decideix navegar-hi, passant-lo directament com a destinació (sense calcular ni passar el seu `id` per separat). Així, `DetallScreen` rep l'`Item` complet (`id`, `nom` i `descripcio`) i no necessita tornar a cercar-lo enlloc: `toRoute<Item>()` ja n'ha reconstruït una còpia completa a partir dels arguments guardats al `backStackEntry`.

## 7. Integració amb `Scaffold` i `NavigationBar`/`NavigationDrawer`


Per saber quina és la destinació actual (i així marcar `selected = true` a l'item corresponent de la `NavigationBar`), es fa servir `currentBackStackEntryAsState`:

```kotlin
val backStackEntry by navController.currentBackStackEntryAsState()
val destinacioActual = backStackEntry?.destination

NavigationBarItem(
    selected = destinacioActual?.hasRoute<Llista>() == true,
    onClick = { navController.navigate(Llista) },
    icon = { Icon(Icons.Default.Home, contentDescription = null) },
    label = { Text("Inici") }
)
```

### Exemple

Aquest exemple munta un `Scaffold` amb una `NavigationBar` de dos elements ("Llista" i "Perfil") i un `NavHost` al seu interior, reaprofitant les destinacions `ToMyListScreen`, `Item` i el composable `Mylist` del punt anterior, i afegint-ne una de nova, `ToPerfilScreen`. La pantalla de login (`ToLoginScreen`, punt 5) queda fora d'aquest `NavHost`: la barra de navegació inferior només té sentit un cop l'usuari ja ha entrat a l'aplicació.

```kotlin

@Serializable
object ToMyListScreen

@Serializable
object ToLoginScreen

@Composable
fun AppNavigation() {
    val navController = rememberNavController()
    val backStackEntry by navController.currentBackStackEntryAsState()
    val destinacioActual = backStackEntry?.destination

    Scaffold(
        bottomBar = {
            NavigationBar {
                NavigationBarItem(
                    selected = destinacioActual?.hasRoute<ToMyListScreen>() == true,
                    onClick = {
                        navController.navigate(ToMyListScreen) {
                            popUpTo(ToMyListScreen) { inclusive = true }
                            launchSingleTop = true
                        }
                    },
                    icon = { Icon(Icons.Default.Home, contentDescription = null) },
                    label = { Text("Llista") }
                )
                NavigationBarItem(
                    selected = destinacioActual?.hasRoute<ToLoginScreen>() == true,
                    onClick = {
                        navController.navigate(ToLoginScreen) {
                            popUpTo(ToMyListScreen)
                            launchSingleTop = true
                        }
                    },
                    icon = { Icon(Icons.Default.Person, contentDescription = null) },
                    label = { Text("Login") }
                )
            }
        }
    ) { innerPadding ->
        NavHost(
            navController = navController,
            startDestination = ToLoginScreen,
            modifier = Modifier.padding(innerPadding)
        ) {
            composable<ToLoginScreen> {
                MyLoginScreen({navController.navigate(ToMyListScreen)})
            }
            composable<ToMyListScreen> {
                Mylist( toDetail = { item -> navController.navigate(item) })
            }
            composable<Item> { backStackEntry ->
                val item: Item = backStackEntry.toRoute()
                DetallScreen(item)
            }
        }
    }
}

```

Dos detalls importants d'aquest exemple:

- El `innerPadding` que rep el contingut del `Scaffold` s'ha d'aplicar al `NavHost` (`Modifier.padding(innerPadding)`); si no es fa, la `NavigationBar` tapa la part inferior de cada pantalla.
- Els dos `NavigationBarItem` fan `popUpTo(ToMyListScreen)` en navegar. Així, en anar de "Login" cap a "Llista" (o a l'inrevés) no s'acumulen destinacions a la pila cada vegada que es canvia de pestanya.

