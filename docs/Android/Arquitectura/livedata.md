
# LiveData (i ViewModel): 
## 1. Conceptes fonamentals de LiveData
### Què és LiveData i per què s'utilitza

LiveData és una classe contenidora de dades observable que forma part d'Android Jetpack. 
La seva funció principal és mantenir i gestionar dades que la UI necessita observar.

### Característiques principals:

- Observable: Notifica automàticament als observadors quan les dades canvien
- Lifecycle-aware: Respecta el cicle de vida dels components d'Android
- Prevé memory leaks: S'eliminen automàticament les subscripcions quan el component es destrueix
- Actualitzacions automàtiques: La UI sempre mostra les dades més recents

### Per què utilitzar LiveData:

- Elimina la necessitat de gestionar manualment les subscripcions
- Garanteix que la UI només s'actualitzi quan està activa (STARTED o RESUMED)
- Evita crashes per actualitzar la UI quan l'Activity/Fragment està destruït
- Facilita la comunicació entre ViewModel i la capa de UI

## 2. Diferències entre dades normals i LiveDataDades normals (variables convencionals):
```kotlin
class  ViewModel {
    var contador: Int = 0  // Variable normal
    
    fun incrementar() {
        contador++
        // Necessites notificar manualment la UI del canvi
    }
}
```
Amb LiveData:
```kotlin 
ViewModel : ViewModel() {
    private val _contador = MutableLiveData<Int>(0)
    val contador: LiveData<Int> = _contador
    
    fun incrementar() {
        _contador.value = _contador.value?.plus(1)
        // La UI s'actualitza automàticament (veure patro observer)
    }
}
```


## 3. El patró Observer aplicat a LiveData
LiveData implementa el patró de disseny Observer, on:

Variable (LiveData): Conté les dades i manté una llista d'observadors
Observer (UI component): S'subscriu per rebre notificacions dels canvis

Funcionament:

La UI s'subscriu al LiveData utilitzant observe()
Quan les dades canvien, LiveData notifica automàticament tots els observadors actius
Els observadors reben les noves dades i actualitzen la UI

Exemple:
```kotlin
// Al ViewModel
class ContadorViewModel : ViewModel() {
    private val _contador = MutableLiveData<Int>(0)
    val contador: LiveData<Int> = _contador
    
    fun incrementar() {
        _contador.value = (_contador.value ?: 0) + 1
    }
}

// A l'Activity/Fragment
viewModel.contador.observe(viewLifecycleOwner) { nouValor ->
    // Aquest codi s'executa automàticament cada vegada que canvia el valor
    textViewContador.text = nouValor.toString()
}
```

### Lifecycle awareness: 

Una de les característiques més potents de LiveData és la seva consciència del cicle de vida dels components d'Android.

La funció observe només s'executarà si el fragment és visible.

### Escenaris pràctics:

- App passa a background: LiveData deixa d'enviar actualitzacions
- App torna a foreground: LiveData envia l'última dada disponible
- Rotació de pantalla: El nou Activity/Fragment rep automàticament l'última dada
- Fragment destruït: L'observador s'elimina automàticament, no hi ha memory leaks

Beneficis:

- No cal fer removeObserver() manualment
- No hi ha crashes per actualitzar vistes que ja no existeixen
- No es processen actualitzacions quan la UI no és visible

## 4. Tipus de LiveData
### LiveData immutable (només lectura)
LiveData és la classe base que proporciona només capacitats de lectura. Els observadors poden rebre dades però no modificar-les directament.

Característiques:

- No té mètodes públics per modificar el valor
- S'utilitza per exposar dades a la UI
- Garanteix que la UI no pugui modificar les dades del ViewModel

Declaració:
```kotlin
val nomUsuari: LiveData<String>  // Només lectura
```
### MutableLiveData (lectura i escriptura)

MutableLiveData és una subclasse de LiveData que afegeix els mètodes setValue() i postValue() per modificar el valor.

Característiques:

- Permet modificar el valor de les dades
- S'utilitza internament al ViewModel
- No s'hauria d'exposar directament a la UI

### Declaració:
```kotlin
private val _nomUsuari = MutableLiveData<String>()  // Privat i mutable
val nomUsuari: LiveData<String> = _nomUsuari  // Públic i immutable
```

## 5. Integració LiveData amb ViewModel
### Com declarar LiveData dins del ViewModel
La declaració correcta de LiveData al ViewModel segueix un patró específic que garanteix l'encapsulació i la separació de responsabilitats.

**Patró recomanat:**

```kotlin
class ProducteViewModel : ViewModel() {
    // 1. Declarar MutableLiveData PRIVAT
    private val _producte = MutableLiveData<Producte>()
    
    // 2. Exposar LiveData immutable PÚBLIC
    val producte: LiveData<Producte> = _producte
    
    // 3. Funcions per modificar les dades
    fun carregarProducte(id: Int) {
        _producte.value = repositori.obtenirProducte(id)
    }
}
```
Amb valors inicials:
```kotlin
class ContadorViewModel : ViewModel() {
    private val _contador = MutableLiveData<Int>(0)  // Valor inicial: 0
    val contador: LiveData<Int> = _contador
    
    private val _nomUsuari = MutableLiveData<String>().apply {
        value = "Usuari desconegut"  // Alternativa per valor inicial
    }
    val nomUsuari: LiveData<String> = _nomUsuari
}
```
Amb tipus complexos:
```kotlin
data class EstatUI(
    val carregant: Boolean = false,
    val dades: List<String> = emptyList(),
    val error: String? = null
)

class DadesViewModel : ViewModel() {
    private val _estat = MutableLiveData<EstatUI>(EstatUI())
    val estat: LiveData<EstatUI> = _estat
    
    fun carregarDades() {
        _estat.value = EstatUI(carregant = true)
        // ... lògica de càrrega
    }
}
```

__Exposar LiveData immutable però mantenir MutableLiveData privat__

Aquest patró és fonamental per mantenir l'encapsulació i seguir els principis de disseny d'Android.

Per què és important:

- Encapsulació: La UI no pot modificar directament les dades
- Únic punt de control: Només el ViewModel pot canviar les dades
- Mantenibilitat: Més fàcil de depurar i mantenir
- Testabilitat: Pots provar el ViewModel sense preocupar-te per modificacions externes


## 6. Exemple complet amb múltiples LiveData:
```kotlin
class CarretCompraViewModel : ViewModel() {
    // Articles del carret
    private val _articles = MutableLiveData<List<Article>>(emptyList())
    val articles: LiveData<List<Article>> = _articles
    
    // Total del carret
    private val _total = MutableLiveData<Double>(0.0)
    val total: LiveData<Double> = _total
    
    // Estat de càrrega
    private val _carregant = MutableLiveData<Boolean>(false)
    val carregant: LiveData<Boolean> = _carregant
    
    fun afegirArticle(article: Article) {
        val llistaActual = _articles.value?.toMutableList() ?: mutableListOf()
        llistaActual.add(article)
        _articles.value = llistaActual
        calcularTotal()
    }
    
    fun eliminarArticle(article: Article) {
        val llistaActual = _articles.value?.toMutableList() ?: return
        llistaActual.remove(article)
        _articles.value = llistaActual
        calcularTotal()
    }
    
    private fun calcularTotal() {
        val articles = _articles.value ?: emptyList()
        _total.value = articles.sumOf { it.preu }
    }
}
```
Ús a l'Activity/Fragment:
```kotlin
class CarretActivity : AppCompatActivity() {
    private val viewModel: CarretCompraViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // La UI només pot observar, no modificar
        viewModel.articles.observe(this) { articles ->
            actualitzarLlistaUI(articles)
        }
        
        viewModel.total.observe(this) { total ->
            textViewTotal.text = "Total: $total€"
        }
        
        // Les modificacions es fan a través del ViewModel
        buttonAfegir.setOnClickListener {
            viewModel.afegirArticle(articleSeleccionat)
        }
    }
}
```

### Separació entre lògica de negoci i UI
La separació adequada entre lògica de negoci (ViewModel) i UI (Activity/Fragment) és essencial per mantenir un codi net i mantenible.

**Responsabilitats del ViewModel:**

- Mantenir i gestionar les dades de la UI
- Implementar la lògica de negoci
- Comunicar-se amb repositoris o fonts de dades
- Processar i transformar dades
- Gestionar l'estat de l'aplicació

**Responsabilitats de la UI:**

- Observar les dades del ViewModel
- Mostrar les dades a l'usuari
- Capturar events de l'usuari
- Cridar funcions del ViewModel en resposta a events
- Gestionar elements visuals (animacions, diàlegs, etc.)





## Referències

Livedata (amb viewbinding)
https://cursokotlin.com/mvvm-en-android-con-kotlin-livedata-y-view-binding-android-architecture-components/

Video de Devexperto:
https://www.youtube.com/watch?v=LHBbs6QXvic

### Codelab
https://developer.android.com/codelabs/basic-android-kotlin-training-livedata?hl=es-419#0

