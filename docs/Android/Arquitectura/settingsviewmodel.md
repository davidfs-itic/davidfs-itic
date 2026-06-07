# ViewModel de Settings amb injecció manual de dependències

Aquest document mostra com fer un ViewModel que gestiona la **configuració de l'usuari** (settings) separant l'accés a dades en un **Repository** i **injectant-lo manualment** al ViewModel mitjançant una Factory, sense fer servir cap framework d'injecció com Hilt.

La idea és repartir bé les responsabilitats: l'accés al sistema (fitxers, preferències, base de dades) viu a la **capa de dades** (el Repository), i el ViewModel només s'encarrega de la lògica de presentació, demanant les dades al Repository que rep pel constructor.

Documentació oficial: [Guide to app architecture](https://developer.android.com/topic/architecture)

## 1. Arquitectura de l'exemple

L'exemple desa una preferència senzilla (el **mode fosc**) i el flux de dependències és aquest:

```
Activity/Fragment  ──crea──>  SettingsViewModelFactory  ──crea──>  SettingsViewModel
       │                                                                 │
       │                                                          (rep el Repository)
       │                                                                 │
       └──────────────── construeix el ──────────────>  SettingsRepository  ──>  DataStore
```

- **SettingsRepository**: capa de dades. És l'únic que coneix el `DataStore`.
- **SettingsViewModel**: lògica de presentació. Només coneix el Repository (per constructor).
- **SettingsViewModelFactory**: construeix el ViewModel passant-li el Repository (injecció manual).
- **Activity/Fragment**: connecta tot, construint el Repository i la Factory.

## 2. Capa de dades: el Repository

El Repository encapsula DataStore. Per a la configuració de DataStore, vegeu [DataStore Preferences](../Llibreries/datastore.md).

```kotlin
import android.content.Context
import androidx.datastore.preferences.core.booleanPreferencesKey
import androidx.datastore.preferences.core.edit
import androidx.datastore.preferences.preferencesDataStore
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.map

// DataStore a nivell de fitxer: garanteix una sola instància
val Context.dataStore by preferencesDataStore(name = "settings")

class SettingsRepository(private val context: Context) {

    private val MODE_FOSC = booleanPreferencesKey("mode_fosc")

    // Lectura reactiva: un Flow que emet cada cop que canvia el valor
    val modeFosc: Flow<Boolean> = context.dataStore.data
        .map { preferences -> preferences[MODE_FOSC] ?: false }

    // Escriptura: funció suspend (DataStore és asíncron)
    suspend fun setModeFosc(actiu: Boolean) {
        context.dataStore.edit { preferences ->
            preferences[MODE_FOSC] = actiu
        }
    }
}
```

## 3. El ViewModel

El ViewModel estén la classe base `ViewModel` i rep el Repository pel **constructor**. Delega tot l'accés a dades al Repository.

```kotlin
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import kotlinx.coroutines.flow.SharingStarted
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.stateIn
import kotlinx.coroutines.launch

class SettingsViewModel(
    private val repository: SettingsRepository
) : ViewModel() {

    // Exposem l'estat a la UI com a StateFlow (vegeu apunts de StateFlow)
    val modeFosc: StateFlow<Boolean> = repository.modeFosc
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5000),
            initialValue = false
        )

    // La UI crida aquesta funció; el ViewModel delega l'escriptura al Repository
    fun canviaModeFosc(actiu: Boolean) {
        viewModelScope.launch {
            repository.setModeFosc(actiu)
        }
    }
}
```

- `repository.modeFosc` és un `Flow`; el convertim en `StateFlow` amb `stateIn` perquè la UI sempre tingui un valor actual. Vegeu [StateFlow](../Kotlin/stateflow.md).
- L'escriptura es fa dins `viewModelScope.launch` perquè `setModeFosc` és `suspend`. Com que DataStore ja gestiona el seu propi dispatcher (és *main-safe*), no cal `Dispatchers.IO` aquí. Vegeu [Corrutines i Dispatchers](../Kotlin/coroutines.md).

## 4. La Factory: injecció manual

Com que el ViewModel té un paràmetre al constructor (`repository`), el sistema no el pot crear sol amb el constructor buit. Cal una **Factory** que li passi la dependència. Aquest és el patró Factory aplicat a ViewModels (vegeu [Patró Factory](./factory.md)).

```kotlin
import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider

class SettingsViewModelFactory(
    private val repository: SettingsRepository
) : ViewModelProvider.Factory {

    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        if (modelClass.isAssignableFrom(SettingsViewModel::class.java)) {
            @Suppress("UNCHECKED_CAST")
            return SettingsViewModel(repository) as T
        }
        throw IllegalArgumentException("ViewModel desconegut: ${modelClass.name}")
    }
}
```

## 5. Connectar-ho des de la UI

Finalment, a l'Activity (o Fragment) construïm el Repository i l'injectem a través de la Factory. Fixa't que passem `applicationContext`, **no** `this`, per evitar fugues de memòria.

```kotlin
class SettingsActivity : AppCompatActivity() {

    // Injecció manual: construïm el Repository i el passem via Factory
    private val viewModel: SettingsViewModel by viewModels {
        SettingsViewModelFactory(SettingsRepository(applicationContext))
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_settings)

        // Recollim el StateFlow de manera segura segons el cicle de vida
        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.modeFosc.collect { actiu ->
                    switchModeFosc.isChecked = actiu
                }
            }
        }

        // Quan l'usuari canvia el switch, ho comuniquem al ViewModel
        switchModeFosc.setOnCheckedChangeListener { _, isChecked ->
            viewModel.canviaModeFosc(isChecked)
        }
    }
}
```

!!! warning "Sempre applicationContext al Repository"
    Si passes l'Activity (`this`) com a `Context` al Repository i aquest sobreviu a l'Activity, tindràs una fuga de memòria. Usa `applicationContext`, que viu tant com el procés de l'aplicació.

## 6. Avantatge: testabilitat

Com que el ViewModel només depèn del Repository, el podem provar amb un test unitari pur substituint el Repository real per un de fals amb valors controlats:

```kotlin
val viewModel = SettingsViewModel(FakeSettingsRepository())
```

Per fer els tests encara més nets, el més habitual és extreure una **interfície** del Repository i fer que el ViewModel en depengui (principi d'[Inversió de Dependències](./inversiodependencies.md)), de manera que el fals només hagi d'implementar la interfície.

## 7. Resum

- El ViewModel rep les dependències pel **constructor** (aquí, el Repository).
- L'accés a dades viu a la **capa de dades** (Repository), que usa `applicationContext`.
- Les dependències s'**injecten manualment** amb una `ViewModelProvider.Factory`, sense Hilt.
- Resultat: codi desacoblat i fàcil de testejar.
