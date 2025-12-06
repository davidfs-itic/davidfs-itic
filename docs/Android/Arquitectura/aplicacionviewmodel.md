## 1. Què és l'ApplicationViewModel


ApplicationViewModel es refereix a una instància de ViewModel que està vinculada al cicle de vida de l'aplicació sencera mitjançant la classe Application d'Android.

Àmbit (Scope): Si un ViewModel s'associa amb l'objecte Application (que actua com a ViewModelStoreOwner), aquest ViewModel existirà durant tota la vida de l'aplicació. Es destrueix només quan el procés de l'aplicació es finalitza pel sistema operatiu.

AndroidViewModel: La classe AndroidViewModel és una subclasse de ViewModel que rep l'objecte Application al seu constructor, cosa que li permet accedir a recursos a nivell d'aplicació (com Context) de manera segura (sense provocar fugues de memòria).

L'Application ViewModel s'utilitza principalment per a la gestió de dades a nivell global que han de sobreviure a tota la vida de l'aplicació i ser accessibles des de qualsevol punt de la UI.

És important notar que, en lloc d'utilitzar directament la classe base ViewModel, utilitzem la seva subclasse AndroidViewModel.

Referència: https://www.youtube.com/watch?v=THt9QISnIMQ&t=51s

## 2. Creació de l'AndroidViewModel:

Per crear un "Application ViewModel" de forma correcta, cal estendre la classe AndroidViewModel

```kotlin
// Exemple: Aplicació per gestionar les preferències d'usuari globals
class AppViewModel(application: Application) : AndroidViewModel(application) {

    // 1. Variable privada modificable (MutableLiveData)
    private val _counter = MutableLiveData<Int>() 
    
    // 2. Variable pública no modificable (LiveData)
    // Aquesta és la variable que les Vistes (Fragments/Activities) observaran.
    val counter: LiveData<Int> = _counter

    // 3. Inicialització
    init {
        _counter.value = 0 
    }

    // 4. Mètode per modificar l'estat (la lògica de negoci)
    fun incrementCounter() {
        // Utilitzem el mètode 'value' per assignació síncrona
        val currentValue = _counter.value ?: 0
        _counter.value = currentValue + 1
    }

    fun decrementCounter() {
        val currentValue = _counter.value ?: 0
        _counter.value = currentValue - 1
    }
}
```

## 3. Instanciació 

Des d'una Activity o un Fragment, s'utilitza la funció de delegat viewModels()

En realitat, el framework de ViewModel ja gestiona automàticament la dependència de l'Application quan detecta que s'està utilitzant la subclasse AndroidViewModel.

```kotlin
private val appViewModel: AppPreferencesViewModel by viewModels()
```

No cal que passis explícitament l'objecte Application a viewModels() o a ViewModelProvider perquè el sistema ja s'encarrega de fer-ho automàticament per tu quan utilitzes la subclasse correcta.

Quan crides a viewModels(), Android utilitza un objecte anomenat ViewModelProvider.Factory. Per defecte, si no en proporciones un de propi, s'utilitza el DefaultViewModelProviderFactory.

Aquesta Factory per defecte conté lògica específica per gestionar els dos casos més comuns:

### Cas 1: ViewModel Base (class MyViewModel : ViewModel)

La Factory intenta instanciar-lo directament utilitzant el seu constructor sense arguments (constructor buit).

### Cas 2: AndroidViewModel (class MyAppViewModel : AndroidViewModel)

La Factory detecta que la classe del ViewModel sol·licitat és una subclasse d'AndroidViewModel.

Quan detecta AndroidViewModel, automàticament mira el ViewModelStoreOwner (que és el teu Activity o Fragment) i n'extreu l'objecte Application associat (a través del Context ).

Finalment, utilitza aquest objecte Application per cridar al constructor de la teva classe: AppPreferencesViewModel(application).

!!! info
    Tota Activity i Fragment té accés al Context, i mitjançant el context, es pot accedir a l'objecte Application de l'aplicació.
    $$\text{Activity/Fragment} \rightarrow \text{Context} \rightarrow \text{Application}$$

## 4. Utilització

```kotlin
class CounterFragment : Fragment() {

    // Instanciació: Obtenim la mateixa instància del ViewModel Global
    // El sistema sap que és un AndroidViewModel i li passa l'Application
    private val appViewModel: AppViewModel by viewModels()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        appViewModel.counter.observe(viewLifecycleOwner) { count ->
            // Actualització automàtica de la UI quan canvia el valor
            counterTextView.text = "Comptador Global: $count"
        }
    }
}
```