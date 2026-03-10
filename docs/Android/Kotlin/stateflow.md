# StateFlow

Un **StateFlow** es un tipus especial de [Flow](./flows.md) dissenyat per representar un **estat** que canvia al llarg del temps. A diferencia d'un Flow normal, un StateFlow sempre te un valor actual i nomes emet quan el valor canvia.

Documentacio oficial: [StateFlow and SharedFlow - Android Developers](https://developer.android.com/kotlin/flow/stateflow-and-sharedflow)

## 1. Caracteristiques principals

- **Sempre te un valor actual**: requereix un valor inicial al crear-lo.
- **No emet duplicats consecutius**: si s'assigna el mateix valor, no notifica als col·lectors.
- **Es calent (hot)**: mante el valor encara que no hi hagi col·lectors actius, a diferencia dels Flows normals que son freds (cold).
- **Ideal per a la UI**: representa l'estat actual de la pantalla al ViewModel.

## 2. MutableStateFlow vs StateFlow

Segueix el mateix patro que `MutableLiveData` / `LiveData`:

- `MutableStateFlow`: permet modificar el valor. Es mante **privat** dins del ViewModel.
- `StateFlow`: nomes lectura. S'exposa **public** perque la UI nomes pugui observar.

```kotlin
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow

class ComptadorViewModel : ViewModel() {

    private val _comptador = MutableStateFlow(0)
    val comptador: StateFlow<Int> = _comptador

    fun incrementar() {
        _comptador.value++
    }
}
```

## 3. Observar un StateFlow des de la UI

Per recollir els valors d'un StateFlow cal fer-ho dins d'una coroutine, normalment amb `lifecycleScope`:

```kotlin
import androidx.lifecycle.lifecycleScope
import kotlinx.coroutines.launch

class ComptadorActivity : AppCompatActivity() {

    private val viewModel: ComptadorViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_comptador)

        lifecycleScope.launch {
            viewModel.comptador.collect { valor ->
                binding.tvComptador.text = "Comptador: $valor"
            }
        }
    }
}
```

!!! warning "Multiples col·lectors"
    Si necessites observar diversos StateFlows, cal llançar un `launch` separat per a cada `collect`, ja que `collect` suspen la coroutine fins que el Flow finalitza.

```kotlin
lifecycleScope.launch {
    viewModel.comptador.collect { valor ->
        binding.tvComptador.text = "Comptador: $valor"
    }
}

lifecycleScope.launch {
    viewModel.nomUsuari.collect { nom ->
        binding.tvNom.text = nom
    }
}
```

## 4. Convertir un Flow a StateFlow amb stateIn

Quan treballem amb un Flow normal (per exemple, el que retorna [DataStore](../Llibreries/datastore.md) o Room) i el volem exposar com a StateFlow des del ViewModel, utilitzem l'operador `stateIn`:

```kotlin
import kotlinx.coroutines.flow.SharingStarted
import kotlinx.coroutines.flow.stateIn

class SettingsViewModel(application: Application) : AndroidViewModel(application) {

    private val preferencesManager = PreferencesManager(application)

    val nomUsuari: StateFlow<String> = preferencesManager.nomUsuari.stateIn(
        scope = viewModelScope,
        started = SharingStarted.WhileSubscribed(5000),
        initialValue = ""
    )
}
```

### Parametres de stateIn

| Parametre | Descripcio |
|---|---|
| `scope` | El CoroutineScope on s'executa el Flow. Normalment `viewModelScope` |
| `started` | Quan comença a col·lectar. `WhileSubscribed(5000)` mante la subscripcio 5 segons despres que l'ultim observador desaparegui |
| `initialValue` | Valor inicial mentre el Flow encara no ha emes cap valor |

!!! info "Per que WhileSubscribed(5000)?"
    El parametre de 5000 ms dona un marge perque, en una rotacio de pantalla, l'Activity es destrueix i es recrea rapidament. Sense aquest marge, el Flow es cancelaria i es reiniciaria innecessariament.

## 5. StateFlow vs LiveData

| Caracteristica | StateFlow | LiveData |
|---|---|---|
| Valor inicial | Obligatori | Opcional |
| Lifecycle-aware | No (cal `lifecycleScope`) | Si, automatic |
| Duplicats consecutius | No emet | Si emet |
| Funciona fora d'Android | Si (Kotlin pur) | No |
| Operadors de transformacio | Tots els de Flow (`map`, `filter`, `combine`...) | Limitats |
| Fil d'emissio | Qualsevol | Nomes fil principal |

## 6. Resum

| Concepte | Us |
|---|---|
| `MutableStateFlow` | Estat mutable dins del ViewModel |
| `StateFlow` | Estat immutable exposat a la UI |
| `stateIn` | Convertir un Flow fred a StateFlow |
| `collect` | Observar els canvis des de la UI |
