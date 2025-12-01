# Viewmodel

Documentació oficial: https://developer.android.com/topic/libraries/architecture/viewmodel


## Informació Viewmodel i livedata
View model és una arquitectura que permet separar la lògica de la activity, de l'activity mateixa.

La separació és important, perque s'entén que una activity no hauria de saber (no s'hi hauria d'implementar) quines dades 


## 1-Conceptes bàsics
### Què és i per què existeix (separació de lògica i UI)
El ViewModel és un component d'arquitectura dissenyat per emmagatzemar i gestionar dades relacionades amb la UI de manera conscient del cicle de vida.
Per què existeix:

- Separació de responsabilitats: La lògica de negoci i les dades estan separades de la UI (Activity/Fragment)
-Testabilitat: Pots testejar la lògica sense necessitat de components d'Android
- Reutilització: Diversos Fragments poden compartir el mateix ViewModel
- Persistència: Les dades sobreviuen a canvis de configuració

#### Sense ViewModel:
```kotlin
class MainActivity : AppCompatActivity() {
    private var counter = 0 // Es perd en rotar la pantalla
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // Lògica barrejada amb codi de UI
        button.setOnClickListener {
            counter++
            textView.text = counter.toString()
            // Crides a API, càlculs, etc...
        }
    }
}
```

#### Amb ViewModel:
```kotlin
class CounterViewModel : ViewModel() {
    private val _counter = MutableLiveData(0)
    val counter: LiveData<Int> = _counter
    
    fun increment() {
        _counter.value = (_counter.value ?: 0) + 1
    }
}

class MainActivity : AppCompatActivity() {
    private val viewModel: CounterViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        viewModel.counter.observe(this) { count ->
            textView.text = count.toString()
        }
        
        button.setOnClickListener {
            viewModel.increment()
        }
    }
}
```

### Cicle de vida del ViewModel vs Activity/Fragment

El ViewModel té un cicle de vida **diferent i més llarg** que l'Activity o Fragment que l'utilitza.

**Cicle de vida:**
```
ActivityCreatedDestroyed (rotation) → Re-created → Finished
    │                    │                │            │
    ▼                    │                │            ▼
ViewModel Created        │                │        onCleared()
    │                    │                │
    └────────────────────┴────────────────┘
         (ViewModel es manté viu)
```
Exemple pràctic:
```kotlin
class MyViewModel : ViewModel() {
    init {
        Log.d("ViewModel", "ViewModel creat")
    }
    
    override fun onCleared() {
        super.onCleared()
        Log.d("ViewModel", "ViewModel destruït")
        // Neteja recursos: cancel·lar coroutines, tancar connexions, etc.
    }
}

class MainActivity : AppCompatActivity() {
    private val viewModel: MyViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        Log.d("Activity", "onCreate")
    }
    
    override fun onDestroy() {
        super.onDestroy()
        Log.d("Activity", "onDestroy")
    }
}
```

**Logs en rotar la pantalla:**
```
ViewModel creat
Activity onCreate
Activity onDestroy (rotació)
Activity onCreate (nova instància)
// ViewModel segueix viu!

// Només quan tanquem l'Activity:
Activity onDestroy
ViewModel destruït
```
### Supervivència a canvis de configuració (rotacions)
El ViewModel sobreviu automàticament a canvis de configuració com rotacions, canvis d'idioma, etc.

El problema sense ViewModel:
```kotlin
class MainActivity : AppCompatActivity() {
    private var userData: User? = null
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // Cada cop que rotes, has de tornar a carregar les dades
        // i si s'han modificat, es perden les modificacions
        userData=loadUserData() // Crida a l'API, base de dades...
        
    }
}
```

La solució amb ViewModel:
```kotlin
class UserViewModel : ViewModel() {
    private val _userData = MutableLiveData<User>()
    val userData: LiveData<User> = _userData
    
    init {
        loadUserData() // Només es crida UNA vegada
    }

    //loadUserData obté l'usuari de una bbdd, api, etc..
    private fun loadUserData() {
        //obtenir l'usuari de manera asíncronta (veure corrutines)
        viewModelScope.launch {
            _userData.value = "Usuari1"
        }
    }
}

class MainActivity : AppCompatActivity() {
    private val viewModel: UserViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // Les dades ja estan disponibles després de rotar
        viewModel.userData.observe(this) { user ->
            textView.text = user.name
        }
    }
}
```

## 2-Àmbits (Scopes)
### ViewModel no compartit (viewModels())
Cada Activity o Fragment té la seva pròpia instància del ViewModel.
Àmbit d'Activity:
```kotlin
class MainActivity : AppCompatActivity() {
    // ViewModel amb àmbit d'aquesta Activity
    private val viewModel: MyViewModel by viewModels()
}

class OtherActivity : AppCompatActivity() {
    // Instància DIFERENT del mateix ViewModel
    private val viewModel: MyViewModel by viewModels()
}
```

Àmbit de Fragment:
```kotlin
class FirstFragment : Fragment() {
    // ViewModel amb àmbit d'aquest Fragment
    private val viewModel: MyViewModel by viewModels()
}

class SecondFragment : Fragment() {
    // Instància DIFERENT, inclús si estan a la mateixa Activity
    private val viewModel: MyViewModel by viewModels()
}
```

Quan utilitzar-ho:

 - Dades específiques d'una única pantalla
 - No necessites compartir informació entre components
 - Lògica aïllada d'un sol Fragment

### ViewModel compartit (activityViewModels())
Diversos Fragments dins de la mateixa Activity poden accedir a la mateixa instància del ViewModel.

Exemple de comunicació entre Fragments:
```kotlin
class SharedViewModel : ViewModel() {
    private val _selectedItem = MutableLiveData<Item>()
    val selectedItem: LiveData<Item> = _selectedItem
    
    fun selectItem(item: Item) {
        _selectedItem.value = item
    }
}

class ListFragment : Fragment() {
    // Obté el ViewModel de l'Activity pare
    private val sharedViewModel: SharedViewModel by activityViewModels()
    
    private fun onItemClicked(item: Item) {
        sharedViewModel.selectItem(item)
    }
}

class DetailFragment : Fragment() {
    // La MATEIXA instància del ViewModel
    private val sharedViewModel: SharedViewModel by activityViewModels()
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        sharedViewModel.selectedItem.observe(viewLifecycleOwner) { item ->
            // Mostra els detalls de l'item seleccionat
            textView.text = item.name
        }
    }
}
```

**Diagrama:**
```
MainActivity
    │
    ├─ SharedViewModel (única instància)
    │
    ├─ ListFragment ──→ activityViewModels() ──→ SharedViewModel
    │
    └─ DetailFragment ──→ activityViewModels() ──→ SharedViewModel
                            (mateixa instància!)
```
Quan utilitzar-ho:

- Comunicació entre Fragments
- Compartir estat dins d'una Activity
- Navegació amb dades (passar informació entre pantalles)
- Patró Master-detail

## 3-ViewModelProvider i Factories
Quan el ViewModel necessita paràmetres al constructor, és necessari implementar un patró factory, per poder passar les dades. Es fa així:

ViewModel amb paràmetres:
```kotlin
class UserViewModel(
    private val userId: String,
    private val repository: UserRepository
) : ViewModel() {
    
    private val _user = MutableLiveData<User>()
    val user: LiveData<User> = _user
    
    init {
        loadUser()
    }
    
    private fun loadUser() {
        viewModelScope.launch {
            _user.value = repository.getUser(userId)
        }
    }
}
```

Crear un Factory:
```kotlin
class UserViewModelFactory(
    private val userId: String,
    private val repository: UserRepository
) : ViewModelProvider.Factory {
    
    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        if (modelClass.isAssignableFrom(UserViewModel::class.java)) {
            @Suppress("UNCHECKED_CAST")
            return UserViewModel(userId, repository) as T
        }
        throw IllegalArgumentException("Unknown ViewModel class")
    }
}
```
Utilitzar el Factory:
```kotlin
class UserActivity : AppCompatActivity() {
    
    private val viewModel: UserViewModel by viewModels {
        UserViewModelFactory(
            userId = intent.getStringExtra("USER_ID") ?: "",
            repository = UserRepository()
        )
    }
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        viewModel.user.observe(this) { user ->
            textView.text = user.name
        }
    }
}
```