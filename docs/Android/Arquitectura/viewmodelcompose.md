# Us de viewmodel amb compose

Documentació oficial: https://developer.android.com/topic/libraries/architecture/viewmodel

El [ViewModel](./viewmodel.md) compleix el mateix paper amb Jetpack Compose que amb el sistema de Views: sobreviu als canvis de configuració i separa la lògica de l'estat de la interfície. La diferència és que amb Compose l'estat no s'assigna directament a widgets (`textView.text = ...`), sinó que es llegeix dins de funcions `@Composable`, que es tornen a executar (recomponen) cada vegada que l'estat que llegeixen canvia.

Aquest document només tracta les particularitats d'utilitzar un ViewModel dins de Compose. Per als conceptes bàsics (per què existeix, cicle de vida, `by viewModels()` vs `by activityViewModels()`, Factory per a paràmetres al constructor) consulteu [ViewModel](./viewmodel.md).

## 1. Obtenir el ViewModel dins d'un Composable

En comptes de `by viewModels()` (propi d'una `Activity` o un `Fragment`), dins d'un Composable s'obté el ViewModel amb la funció `viewModel()`:

```kotlin
import androidx.lifecycle.viewmodel.compose.viewModel

@Composable
fun ContadorScreen(
    viewModel: ContadorViewModel = viewModel()
) {
    // ...
}
```

`viewModel()` retorna sempre la mateixa instància mentre el Composable es manté a la mateixa entrada de `NavBackStackEntry` (o a la mateixa Activity si no s'utilitza Navigation Compose), encara que hi hagi recomposicions o canvis de configuració. L'àmbit (scope) segueix les mateixes regles que ja es coneixen de `viewmodel.md`: si el Composable forma part d'un graf de navegació, el ViewModel es pot compartir entre pantalles fent-lo dependre de l'entrada del graf en lloc de la pantalla individual.

## 2. Exposar l'estat: StateFlow enlloc de LiveData

A `viewmodel.md` l'estat s'exposa amb `LiveData`, perquè és el que s'observa còmodament amb `.observe(this) { }` des d'una Activity o Fragment. Dins de Compose, `LiveData` també funciona (amb `observeAsState()`), però `StateFlow` és la opció recomanada perquè és la primitiva d'estat pròpia de les corrutines i s'integra de manera nativa amb Compose.

#### Sense StateFlow (LiveData + observeAsState):

```kotlin
class ContadorViewModel : ViewModel() {
    private val _contador = MutableLiveData(0)
    val contador: LiveData<Int> = _contador
}

@Composable
fun ContadorScreen(viewModel: ContadorViewModel = viewModel()) {
    val contador by viewModel.contador.observeAsState(0)
    Text("Contador: $contador")
}
```

#### Amb StateFlow:

```kotlin
class ContadorViewModel : ViewModel() {
    private val _contador = MutableStateFlow(0)
    val contador: StateFlow<Int> = _contador.asStateFlow()
}

@Composable
fun ContadorScreen(viewModel: ContadorViewModel = viewModel()) {
    val contador by viewModel.contador.collectAsStateWithLifecycle()
    Text("Contador: $contador")
}
```

!!! info "collectAsStateWithLifecycle() vs collectAsState()"
    `collectAsState()` recull el `StateFlow` mentre el Composable estigui a la composició, sense tenir en compte el cicle de vida de l'Activity. Això fa que continuï recollint (i, per tant, gastant recursos) encara que la pantalla estigui en segon pla.

    `collectAsStateWithLifecycle()` només recull mentre el cicle de vida estigui com a mínim en `STARTED`, igual que fa `repeatOnLifecycle` amb corrutines. És l'opció recomanada per Google i requereix afegir la dependència `androidx.lifecycle:lifecycle-runtime-compose`.

## 3. Patró estat avall / events amunt

El mateix principi de *state hoisting* explicat a [Estats en Jetpack Compose](../Interficies/Jetpack_compose/estats.md#3-state-hoisting) s'aplica quan l'estat viu al ViewModel: el Composable no modifica l'estat directament, només el llegeix i notifica events cap amunt.

- **Estat cap avall**: el ViewModel exposa un `StateFlow` amb l'estat actual de la pantalla; el Composable el llegeix amb `collectAsStateWithLifecycle()` i el mostra.
- **Events cap amunt**: el Composable no canvia l'estat; crida funcions del ViewModel (o lambdes que hi apunten) quan l'usuari interactua, i és el ViewModel qui decideix com actualitzar l'estat.

Aquest patró es coneix com a *Unidirectional Data Flow* (UDF): la informació sempre circula en un únic sentit, cosa que fa que l'estat de la pantalla sigui predictible i fàcil de provar.

## 4. Exemple complet: pantalla d'un comptador

Adaptem a Compose l'exemple de comptador amb un requisit afegit: el botó "Tornar" només s'ha d'activar quan el comptador arriba a 10. Tot l'estat de la pantalla (el valor del comptador i si el botó està actiu) es modela en una única data class, `ContadorUiState`:

```kotlin
data class ContadorUiState(
    val contador: Int = 0,
    val botoTornarActivat: Boolean = false
)

class ContadorViewModel : ViewModel() {

    private val _uiState = MutableStateFlow(ContadorUiState())
    val uiState: StateFlow<ContadorUiState> = _uiState.asStateFlow()

    fun incrementar() {
        _uiState.update { actual ->
            val nouContador = actual.contador + 1
            actual.copy(
                contador = nouContador,
                botoTornarActivat = nouContador >= 10
            )
        }
    }
}

@Composable
fun ContadorScreen(
    viewModel: ContadorViewModel = viewModel(),
    onTornar: () -> Unit
) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()

    Column {
        Text("Contador: ${uiState.contador}")

        Button(onClick = { viewModel.incrementar() }) {
            Text("Incrementa")
        }

        Button(
            onClick = onTornar,
            enabled = uiState.botoTornarActivat
        ) {
            Text("Tornar")
        }
    }
}
```

Cal fixar-se que el Composable no calcula en cap moment si el botó ha d'estar actiu: només llegeix `uiState.botoTornarActivat`. Tota la lògica (quan incrementar, quan activar el botó) viu al ViewModel, dins de `incrementar()`. Això és el que permet, per exemple, provar `ContadorViewModel` amb un test unitari sense necessitat d'executar cap Composable.

## 5. ViewModel amb paràmetres al constructor (dependències)

Quan el ViewModel necessita dependències al constructor (repository, use cases...), cal un [Factory](./viewmodel.md#3-viewmodelprovider-i-factories), igual que amb el sistema de Views. La manera recomanada és definir-lo com a `companion object` dins del mateix ViewModel, amb `viewModelFactory { }` i `initializer { }`:

```kotlin
class LlistatScreenViewModel(
    private val getItemsUseCase: GetItemsUseCase,
    private val addItemUseCase: AddItemUseCase
) : ViewModel() {

    // ...

    companion object {
        val Factory: ViewModelProvider.Factory = viewModelFactory {
            initializer {
                val local = ItemsLocalDataSource()
                val remote = ItemsRemoteDataSource()
                val repository: ItemsRepository = ItemsRepositoryImpl(local, remote)
                LlistatScreenViewModel(
                    getItemsUseCase = GetItemsUseCase(repository),
                    addItemUseCase = AddItemUseCase(repository)
                )
            }
        }
    }
}
```

Dins d'un Composable, en comptes de `by viewModels { }`, el Factory es passa al paràmetre `factory` de la funció `viewModel()`:

```kotlin
@Composable
fun LlistatScreen(
    viewModel: LlistatScreenViewModel = viewModel(factory = LlistatScreenViewModel.Factory)
) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()
    // ...
}
```

D'aquesta manera el Composable no necessita saber com es construeixen el repository ni els use cases: només indica quin Factory ha d'utilitzar `viewModel()` per crear la instància.

## 6. Recursos

- [ViewModel](./viewmodel.md)
- [Estats en Jetpack Compose](../Interficies/Jetpack_compose/estats.md)
- https://developer.android.com/topic/libraries/architecture/viewmodel
- https://developer.android.com/jetpack/compose/state
