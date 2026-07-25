# El composable Scaffold

`Scaffold` és el composable que Material posa a disposició per muntar l'esquelet estàndard d'una pantalla: barra superior, barra inferior, botó flotant, missatges emergents... En lloc d'haver de posicionar manualment cada element amb `Box` o `Column`, `Scaffold` ofereix un seguit de "forats" (*slots*) ja pensats i col·locats segons les normes de Material Design, de manera semblant al que feien conjuntament `CoordinatorLayout` i `AppBarLayout` en el sistema de Views.

Documentació oficial: https://developer.android.com/develop/ui/compose/components/scaffold

## 1. Concepte: una pantalla feta de "slots"

`Scaffold` no dibuixa res per si mateix: és un contenidor que accepta un paràmetre per a cada zona típica d'una pantalla (`topBar`, `bottomBar`, `floatingActionButton`, `snackbarHost`, `content`...), i s'encarrega de col·locar-los a la posició correcta i de calcular l'espai que ocupa cadascun. Cada slot és, com sempre a Compose, una lambda `@Composable`, de manera que s'hi pot posar qualsevol composable, no un component fix.

```kotlin
@Composable
fun PantallaPrincipal() {
    Scaffold(
        topBar = { /* barra superior */ },
        bottomBar = { /* barra inferior */ },
        floatingActionButton = { /* botó flotant */ }
    ) { innerPadding ->
        // contingut principal de la pantalla
    }
}
```

Aquest disseny obliga a pensar la pantalla per zones (què va a dalt, què va a baix, què és contingut principal) en lloc de calcular posicions i marges a mà, i garanteix que tots els elements respectin automàticament les distàncies i comportaments que defineix Material Design.

## 2. `topBar`

El slot `topBar` és on normalment es col·loca una `TopAppBar` (o la seva variant `CenterAlignedTopAppBar`), amb un títol, una icona de navegació (per exemple, la fletxa d'enrere) i accions a la dreta:

```kotlin
Scaffold(
    topBar = {
        TopAppBar(
            title = { Text("La meva app") },
            navigationIcon = {
                IconButton(onClick = { /* tornar enrere */ }) {
                    Icon(Icons.Default.ArrowBack, contentDescription = "Enrere")
                }
            },
            actions = {
                IconButton(onClick = { /* obrir cerca */ }) {
                    Icon(Icons.Default.Search, contentDescription = "Cercar")
                }
            }
        )
    }
) { innerPadding ->
    // contingut
}
```

- **`title`**: contingut del títol (normalment un `Text`).
- **`navigationIcon`**: icona a l'esquerra, típicament per tornar a la pantalla anterior.
- **`actions`**: icones a la dreta, per a accions ràpides relacionades amb la pantalla actual.

## 3. `bottomBar` i `floatingActionButton`

El slot `bottomBar` acostuma a contenir una `NavigationBar` (l'equivalent en Compose al `BottomNavigationView` de Views) amb diverses `NavigationBarItem`, cadascuna amb una icona, una etiqueta i el seu propi estat de selecció:

```kotlin
Scaffold(
    bottomBar = {
        NavigationBar {
            NavigationBarItem(
                selected = true,
                onClick = { },
                icon = { Icon(Icons.Default.Home, contentDescription = null) },
                label = { Text("Inici") }
            )
            NavigationBarItem(
                selected = false,
                onClick = { },
                icon = { Icon(Icons.Default.Settings, contentDescription = null) },
                label = { Text("Ajustos") }
            )
        }
    }
) { innerPadding ->
    // contingut
}
```

`bottomBar` és igualment un slot genèric: `NavigationBar` és simplement el composable que sol utilitzar-s'hi, però s'hi podria posar qualsevol altre contingut.

El botó d'acció flotant es col·loca amb `floatingActionButton`, i la seva posició es controla amb `floatingActionButtonPosition`:

```kotlin
Scaffold(
    floatingActionButton = {
        FloatingActionButton(onClick = { /* acció principal */ }) {
            Icon(Icons.Default.Add, contentDescription = "Afegir")
        }
    },
    floatingActionButtonPosition = FabPosition.End
) { innerPadding ->
    // contingut
}
```

`FabPosition.End` (per defecte) el situa a la cantonada inferior dreta; `FabPosition.Center` el centra horitzontalment, habitual quan es combina amb una `bottomBar` amb una osca (*cutout*) per encaixar-lo.

## 4. `content` i el padding intern

El paràmetre de contingut principal de `Scaffold` no és una lambda `@Composable () -> Unit` normal: és una lambda que **rep un `PaddingValues`** com a argument (anomenat sovint `innerPadding`), calculat automàticament segons quins slots s'hagin fet servir (l'alçada de la `topBar`, la `bottomBar`, etc.):

```kotlin
Scaffold(
    topBar = { TopAppBar(title = { Text("Llista") }) }
) { innerPadding ->
    LazyColumn(
        modifier = Modifier
            .fillMaxSize()
            .padding(innerPadding)
    ) {
        // elements de la llista
    }
}
```

Aquest `innerPadding` **s'ha d'aplicar sempre** al contingut arrel, normalment amb `Modifier.padding(innerPadding)`. Si s'ignora, el contingut es dibuixa ocupant tota la pantalla per sota de les barres, i queda parcialment tapat per la `topBar` o la `bottomBar`:

```kotlin
// Incorrecte: el innerPadding no s'aplica enlloc
Scaffold(
    topBar = { TopAppBar(title = { Text("Llista") }) }
) { innerPadding ->
    LazyColumn(modifier = Modifier.fillMaxSize()) {
        // els primers elements queden amagats sota la topBar
    }
}
```

Aquest és, amb diferència, l'error més habitual en començar a fer servir `Scaffold`: el `Scaffold` calcula l'espai correcte, però és responsabilitat del contingut aplicar-lo.

## 5. `snackbarHost`

Un `Snackbar` és un missatge breu que apareix a la part inferior de la pantalla i desapareix automàticament al cap d'uns segons. `Scaffold` en gestiona la posició i l'animació a través del slot `snackbarHost`, però mostrar-lo requereix un estat (`SnackbarHostState`) i, com que `showSnackbar` és una funció `suspend`, cal llançar-la des d'una coroutine:

```kotlin
@Composable
fun PantallaAmbSnackbar() {
    val snackbarHostState = remember { SnackbarHostState() }
    val scope = rememberCoroutineScope()

    Scaffold(
        snackbarHost = { SnackbarHost(snackbarHostState) },
        floatingActionButton = {
            FloatingActionButton(onClick = {
                scope.launch {
                    snackbarHostState.showSnackbar("Element afegit")
                }
            }) {
                Icon(Icons.Default.Add, contentDescription = "Afegir")
            }
        }
    ) { innerPadding ->
        // contingut
    }
}
```

Aquí `rememberCoroutineScope` és l'eina adequada perquè el `Snackbar` s'ha de mostrar com a resposta directa a un clic, no automàticament en aparèixer el composable. Per aquest mateix motiu, quan calgui mostrar un `Snackbar` a l'inici (per exemple, per avisar d'un error carregat des d'un `ViewModel`), s'utilitza en canvi un `LaunchedEffect`; la diferència entre tots dos casos es detalla a [LaunchedEffect](./launchedeffect.md#6-launchedeffect-vs-remembercoroutinescope).

## 6. Exemple complet

```kotlin
@Composable
fun PantallaCompleta() {
    val snackbarHostState = remember { SnackbarHostState() }
    val scope = rememberCoroutineScope()

    Scaffold(
        topBar = {
            TopAppBar(title = { Text("Tasques") })
        },
        floatingActionButton = {
            FloatingActionButton(onClick = {
                scope.launch { snackbarHostState.showSnackbar("Tasca afegida") }
            }) {
                Icon(Icons.Default.Add, contentDescription = "Afegir tasca")
            }
        },
        snackbarHost = { SnackbarHost(snackbarHostState) }
    ) { innerPadding ->
        LazyColumn(
            modifier = Modifier
                .fillMaxSize()
                .padding(innerPadding)
        ) {
            item { Text("Contingut de la pantalla") }
        }
    }
}
```

Aquest exemple combina els quatre slots vistos (`topBar`, `floatingActionButton`, `snackbarHost` i `content`) i mostra el patró que es repetirà a la majoria de pantalles construïdes amb Material en Compose.
