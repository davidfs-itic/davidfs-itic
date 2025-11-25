# Viewmodel i livedata

Documentació oficial: https://developer.android.com/topic/libraries/architecture/viewmodel


## Informació Viewmodel i livedata
View model és una arquitectura que permet separar la lògica de la activity, de l'activity mateixa.

La separació és important, perque s'entén que una activity no hauria de saber (no s'hi hauria d'implementar) quines dades 



## Referències

Livedata (amb viewbinding)
https://cursokotlin.com/mvvm-en-android-con-kotlin-livedata-y-view-binding-android-architecture-components/

Video de Devexperto:
https://www.youtube.com/watch?v=LHBbs6QXvic

### Codelab
https://developer.android.com/codelabs/basic-android-kotlin-training-livedata?hl=es-419#0


# FRAGMENTS AMB VIEWMODEL
## 1-Introducció a ViewModel
### Problemes que resol
Pèrdua d'estat: Les dades es mantenen en rotacions de pantalla

Comunicació complexa: Facilita compartir dades entre fragments

Fugues de memòria: Gestió automàtica del cicle de vida

### Conceptes clau
Sobrevivència: El ViewModel no es destrueix en canvis de configuració

Scope: Associat al Lifecycle de l'Activity o Fragment

Netega automàtica: Es destrueix quan el propietari finalitza

### Referència: 
Guia de ViewModel https://developer.android.com/topic/libraries/architecture/viewmodel

## 2-ViewModel amb Fragments
### Escollir l'scope correcte
```kotlin
// ViewModel PER FRAGMENT INDIVIDUAL
private val viewModel: MyViewModel by viewModels()

// ViewModel COMPARTIT entre TOTS els fragments de l'activity  
private val sharedViewModel: SharedViewModel by activityViewModels()
```
### Quan utilitzar cada un

- viewModels(): Dades específiques d'un sol fragment
- activityViewModels(): Dades que múltiples fragments necessiten

## 3-LiveData i StateFlow
### Observar canvis de dades
```kotlin
// LiveData (tradicional)
viewModel.myLiveData.observe(viewLifecycleOwner) { dades ->
    // Actualitzar UI amb les noves dades
}

// StateFlow (modern)
lifecycleScope.launch {
    viewModel.myStateFlow.collect { dades ->
        // Actualitzar UI
    }
}
```

### Pattern recomanat
```kotlin
class MyViewModel : ViewModel() {
    // Privat - només el ViewModel pot modificar
    private val _uiState = MutableStateFlow<UiState>(UiState.Loading)
    
    // Públic - els fragments només poden observar
    val uiState: StateFlow<UiState> = _uiState.asStateFlow()
    
    fun updateData() {
        _uiState.value = UiState.Success(dades)
    }
}
```
### Referència: 
StateFlow i SharedFlow: https://developer.android.com/kotlin/flow/stateflow-and-sharedflow

## 4-Arquitectura amb Fragments + ViewModel
### Separació de responsabilitats
ViewModel: Lògica de negoci, dades, estat

Fragment: UI, interaccions usuari, navegació

Activity: Coordinació, gestió de fragments, permisos

### Comunicació entre fragments
```kotlin
// ✅ CORRECTE - ViewModel compartit
class FragmentA : Fragment() {
    private val sharedViewModel: SharedViewModel by activityViewModels()
    
    fun onButtonClick() {
        sharedViewModel.updateData(novaDada)
    }
}

class FragmentB : Fragment() {
    private val sharedViewModel: SharedViewModel by activityViewModels()
    
    override fun onViewCreated() {
        lifecycleScope.launch {
            sharedViewModel.dades.collect { dades ->
                // Rep les actualitzacions de FragmentA
            }
        }
    }
}
```

## 5 - Casos Pràctics Avançats
### Compartir dades complexes
Usuari autenticat: Estat de sessió compartit per tota l'app

Carro de la compra: Dades persistents durant la sessió

Configuracions: Preferències que afecten múltiples pantalles

### Navegació amb Navigation Component
```kotlin
// ViewModel compartit per tots els fragments del nav graph
private val viewModel: SharedViewModel by navGraphViewModels(R.id.nav_graph)

// Passar arguments segurs
val directions = MyFragmentDirections.actionToNextFragment(argument = valor)
findNavController().navigate(directions)
```

### Gestió d'errors i loading
```kotlin
// Al ViewModel
sealed class UiState {
    object Loading : UiState()
    data class Success(val data: Data) : UiState()
    data class Error(val message: String) : UiState()
}

// Al Fragment
when (val state = viewModel.uiState.value) {
    is UiState.Loading -> showLoading()
    is UiState.Success -> showData(state.data)
    is UiState.Error -> showError(state.message)
}
```
## 6 - QUÈ S'HA DE TENIR EN COMPTE
### Aspectes Clau

- Scope del ViewModel: Pensa si les dades són per un fragment o per compartir

- Cicle de vida: Assegura't que observes les dades amb el lifecycle correcte

- Netega: Els ViewModel es netegen sols, però tanca recursos externs (base de dades, xarxa)

- Testeig: Els ViewModel són fàcils de testejar, aprofita-ho

### Errors Comuns a Evitar

```kotlin
//MAI - Passar context al ViewModel
class MyViewModel(val context: Context) : ViewModel()

//Correcte - Demanar context quan es necessiti
class MyViewModel : ViewModel() {
    fun getData(context: Context) { ... }
}

//MAI- Utilitzar viewModels() quan es necessita compartir
private val sharedViewModel: SharedViewModel by viewModels() 
// Cada fragment té la seva instància

//Correcte - Utilitzar activityViewModels() per compartir
private val sharedViewModel: SharedViewModel by activityViewModels() // Tots comparteixen
```
### Bones Pràctiques
- ViewModel net d'Android: Sense imports de UI al ViewModel

- Estats exhaustius: Gestiona tots els estats possibles (loading, èxit, error)

- Coroutines: Utilitza coroutines per operacions async al ViewModel

- Immutabilitat: Exposa dades immutables als fragments

## Referències addicionals:

- Guia d'arquitectura Android https://developer.android.com/topic/architecture

- Patterns de navegació https://developer.android.com/guide/navigation/navigation-principles

- Guia de coroutines https://developer.android.com/kotlin/coroutines
