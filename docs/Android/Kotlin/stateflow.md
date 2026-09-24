# StateFlow

Un **StateFlow** és un tipus especial de [Flow](./flows.md) dissenyat per representar un **estat** que canvia al llarg del temps. A diferència d'un Flow normal, un StateFlow sempre té un valor actual i només emet quan el valor canvia.

Documentació oficial: [StateFlow and SharedFlow - Android Developers](https://developer.android.com/kotlin/flow/stateflow-and-sharedflow)

## 1. Característiques principals

- **Sempre té un valor actual**: requereix un valor inicial al crear-lo.
- **No emet duplicats consecutius**: si s'assigna el mateix valor, no notifica als col·lectors.
- **És calent (hot)**: manté el valor encara que no hi hagi col·lectors actius, a diferència dels Flows normals que són freds (cold).
- **Ideal per a la UI**: representa l'estat actual de la pantalla al ViewModel.

## 2. MutableStateFlow vs StateFlow

Segueix el mateix patró que `MutableLiveData` / `LiveData`:

- `MutableStateFlow`: permet modificar el valor. Es manté **privat** dins del ViewModel.
- `StateFlow`: només lectura. S'exposa **públic** perquè la UI només pugui observar.

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

!!! warning "Múltiples col·lectors"
    Si necessites observar diversos StateFlows, cal llançar un `launch` separat per a cada `collect`, ja que `collect` suspèn la coroutine fins que el Flow finalitza.

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

### Paràmetres de stateIn

| Paràmetre | Descripció |
|---|---|
| `scope` | El CoroutineScope on s'executa el Flow. Normalment `viewModelScope` |
| `started` | Quan comença a col·lectar. `WhileSubscribed(5000)` manté la subscripció 5 segons després que l'últim observador desaparegui |
| `initialValue` | Valor inicial mentre el Flow encara no ha emès cap valor |

!!! info "Per què WhileSubscribed(5000)?"
    El paràmetre de 5000 ms dona un marge perquè, en una rotació de pantalla, l'Activity es destrueix i es recrea ràpidament. Sense aquest marge, el Flow es cancel·laria i es reiniciaria innecessàriament.

## 5. StateFlow vs LiveData

| Característica | StateFlow | LiveData |
|---|---|---|
| Valor inicial | Obligatori | Opcional |
| Lifecycle-aware | No (cal `lifecycleScope`) | Sí, automàtic |
| Duplicats consecutius | No emet | Sí emet |
| Funciona fora d'Android | Sí (Kotlin pur) | No |
| Operadors de transformació | Tots els de Flow (`map`, `filter`, `combine`...) | Limitats |
| Fil d'emissió | Qualsevol | Només fil principal |

## 6. Resum

| Concepte | Ús |
|---|---|
| `MutableStateFlow` | Estat mutable dins del ViewModel |
| `StateFlow` | Estat immutable exposat a la UI |
| `stateIn` | Convertir un Flow fred a StateFlow |
| `collect` | Observar els canvis des de la UI |
