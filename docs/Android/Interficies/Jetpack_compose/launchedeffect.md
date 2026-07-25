# LaunchedEffect

`LaunchedEffect` és el mecanisme de Compose per executar codi que **no és pur càlcul d'interfície**: crides a coroutines, peticions de xarxa, temporitzadors, animacions úniques, etc. Aquest tipus de codi s'anomena *side effect* (efecte secundari), perquè passa "al marge" del procés normal de composició/recomposició, i per això Compose exigeix fer-lo servir dins d'una API específica en lloc d'escriure'l directament dins d'una funció `@Composable`.

Documentació oficial: https://developer.android.com/develop/ui/compose/side-effects

## 1. Per què no es pot cridar una coroutine directament

Una funció `@Composable` es pot executar múltiples vegades (a cada recomposició), en qualsevol ordre, i fins i tot pot no arribar a executar-se sencera. Si dins d'un composable es cridés directament una funció `suspend` o es llancés una coroutine amb `CoroutineScope(...).launch { }`, aquesta crida es repetiria a cada recomposició sense control, provocant peticions de xarxa duplicades, temporitzadors múltiples o fuites de memòria.

`LaunchedEffect` resol això: llança una coroutine lligada al cicle de vida de la composició, que Compose gestiona automàticament (la crea, la cancel·la i la torna a llançar) seguint unes regles clares.

## 2. Ús bàsic

`LaunchedEffect` necessita una **clau** (*key*) i un bloc de codi `suspend`:

```kotlin
@Composable
fun PantallaUsuari(usuariId: String) {
    var usuari by remember { mutableStateOf<Usuari?>(null) }

    LaunchedEffect(usuariId) {
        usuari = obtenirUsuari(usuariId) // funció suspend
    }

    if (usuari != null) {
        Text("Nom: ${usuari!!.nom}")
    }
}
```

Quan aquest composable entra a la composició per primera vegada, `LaunchedEffect` llança una coroutine que executa el bloc. Si el composable es recompon però `usuariId` no ha canviat, el bloc **no** es torna a executar; la coroutine ja llançada es manté en marxa (o ja ha acabat). Aquest comportament és el que evita repetir la crida a `obtenirUsuari` a cada recomposició.

## 3. La clau (`key`): quan es torna a executar l'efecte

La clau que es passa a `LaunchedEffect` determina quan cal cancel·lar la coroutine anterior i llançar-ne una de nova. Hi ha tres casos habituals:

```kotlin
// S'executa una sola vegada, quan el composable entra a la composició
LaunchedEffect(Unit) {
    // inicialització única
}

// Es torna a executar cada vegada que canvia "usuariId"
LaunchedEffect(usuariId) {
    usuari = obtenirUsuari(usuariId)
}

// Es torna a executar quan canvia "usuariId" O "filtre"
LaunchedEffect(usuariId, filtre) {
    usuari = obtenirUsuari(usuariId, filtre)
}
```

- **`Unit` (o qualsevol valor constant)**: com que la clau mai canvia, l'efecte només s'executa un cop, la primera vegada que el composable es mostra. És l'equivalent aproximat a un `onCreate()` o `initState()` d'altres frameworks.
- **Una variable d'estat**: cada vegada que aquesta variable canvia de valor, Compose cancel·la la coroutine en curs (si encara no havia acabat) i en llança una de nova amb el valor actualitzat.
- **Diverses claus**: l'efecte es torna a llançar si **qualsevol** de les claus canvia.

Triar bé la clau és la part més important de `LaunchedEffect`: una clau que canvia massa sovint provoca reexecucions innecessàries; una clau que no inclou totes les dades rellevants (per exemple, deixar-se `filtre` de l'exemple anterior) fa que l'efecte quedi desactualitzat sense que Compose ho detecti.

## 4. Cancel·lació automàtica

Quan la clau canvia, o quan el composable que conté el `LaunchedEffect` desapareix de la composició (per exemple, es navega a una altra pantalla), Compose **cancel·la automàticament** la coroutine en curs abans de llançar-ne una de nova (si n'hi ha). Això es tradueix directament en `CoroutineScope` i excepcions de cancel·lació (`CancellationException`) igual que amb qualsevol altra coroutine:

```kotlin
LaunchedEffect(consultaText) {
    delay(300) // debounce: espera abans de cercar
    resultats = cercar(consultaText)
}
```

En aquest exemple típic de *debounce*, si l'usuari escriu una nova lletra abans que passin els 300 ms, la coroutine anterior es cancel·la (i el `delay` es talla) abans d'acabar, de manera que `cercar(...)` només s'arriba a executar amb el darrer text introduït. Aquest patró seria molt difícil d'implementar correctament sense la gestió automàtica de cancel·lació que ofereix `LaunchedEffect`.

## 5. Casos d'ús habituals

- **Carregar dades en entrar a una pantalla**: `LaunchedEffect(Unit) { dades = repositori.obtenirDades() }`.
- **Reaccionar a un canvi de paràmetre**: tornar a carregar dades quan canvia un identificador o un filtre, com a l'exemple del punt 2.
- **Mostrar un `Snackbar`**: `SnackbarHostState.showSnackbar(...)` és una funció `suspend`, per tant només es pot cridar des d'un `LaunchedEffect` o una coroutine equivalent.
- **Animacions o temporitzadors únics**: per exemple, esperar uns segons i navegar automàticament (una pantalla de splash).
- **Debounce de cerca**: com a l'exemple del punt 4.

## 6. `LaunchedEffect` vs `rememberCoroutineScope`

És habitual confondre `LaunchedEffect` amb `rememberCoroutineScope`. La diferència és **qui inicia la coroutine**:

- **`LaunchedEffect`**: Compose l'inicia automàticament quan el composable entra a la composició (o quan canvia la clau). No es pot llançar manualment des d'un `onClick`, perquè el seu bloc s'executa fora de qualsevol gestor d'esdeveniments.
- **`rememberCoroutineScope`**: proporciona un `CoroutineScope` que **es llança manualment** en resposta a un esdeveniment, típicament dins d'un `onClick`, ja que aquest no és una funció `suspend`:

```kotlin
val scope = rememberCoroutineScope()

Button(onClick = {
    scope.launch {
        snackbarHostState.showSnackbar("Element eliminat")
    }
}) {
    Text("Eliminar")
}
```

Com a regla pràctica: si l'efecte s'ha de disparar automàticament (en aparèixer el composable o quan canvia una dada), s'utilitza `LaunchedEffect`; si s'ha de disparar com a resposta directa a una acció de l'usuari (un clic, per exemple), s'utilitza `rememberCoroutineScope`.
