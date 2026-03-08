# Tests Unitaris

Els **tests unitaris** proven funcions o classes de manera aïllada, sense necessitat del sistema Android. S'executen directament a la JVM del teu ordinador, cosa que els fa molt ràpids.

## 1. Què són els tests unitaris?

Un test unitari comprova que una **unitat de codi** (una funció, un mètode o una classe) funciona correctament de manera independent. No necessiten un dispositiu ni un emulador.

- **On van**: carpeta `app/src/test/java/`
- **S'executen a**: la JVM local (sense Android)
- **Velocitat**: molt ràpids (mil·lisegons)
- **Què proven**: lògica de negoci, validacions, càlculs, transformacions de dades

## 2. Estructura d'un test: patró AAA

Cada test segueix el patró **AAA** (Arrange-Act-Assert):

```kotlin
@Test
fun sumar_dosNombresPositius_retornaLaSuma() {
    // Arrange — Preparar les dades
    val viewModel = CalculadoraViewModel()

    // Act — Executar l'acció
    viewModel.sumar("10", "5")

    // Assert — Comprovar el resultat
    assertEquals("Resultat: 15.0", viewModel.resultat.value)
}
```

| Fase | Què fa | Exemple |
|---|---|---|
| **Arrange** | Prepara els objectes i dades necessaris | Crear instàncies, definir variables |
| **Act** | Executa la funció que vols provar | Cridar el mètode |
| **Assert** | Comprova que el resultat és l'esperat | Usar assertions de JUnit |

## 3. Assertions bàsiques de JUnit

JUnit proporciona diverses funcions per comprovar resultats:

```kotlin
import org.junit.Assert.*

// Comprovar igualtat
assertEquals(esperat, real)

// Comprovar igualtat amb decimals (delta = marge d'error)
assertEquals(3.14, resultat, 0.01)

// Comprovar que és cert
assertTrue(condicio)

// Comprovar que és fals
assertFalse(condicio)

// Comprovar que no és null
assertNotNull(objecte)

// Comprovar que és null
assertNull(objecte)

// Comprovar que llança una excepció
@Test(expected = IllegalArgumentException::class)
fun test_llancaExcepcio() {
    // codi que ha de llançar l'excepció
}
```

!!! tip "assertEquals amb decimals"
    Quan compares nombres `Double`, afegeix sempre un tercer paràmetre `delta` (marge d'error), perquè els decimals en punt flotant poden tenir petites imprecisions:
    ```kotlin
    assertEquals(3.14, resultat, 0.001)
    ```

## 4. Testejar un ViewModel amb LiveData

Quan el ViewModel utilitza **LiveData**, cal una configuració especial perquè LiveData necessita el thread principal d'Android, que no existeix als tests unitaris. La solució és afegir una `Rule` que fa que LiveData funcioni de manera síncrona.

### Dependència necessària

Al fitxer `build.gradle.kts` del mòdul `app`, afegeix:

```kotlin
dependencies {
    // Tests unitaris
    testImplementation("junit:junit:4.13.2")
    testImplementation("androidx.arch.core:core-testing:2.2.0")
}
```

### InstantTaskExecutorRule

Aquesta `Rule` fa que les operacions de LiveData s'executin immediatament en el mateix thread, en lloc de necessitar el thread principal d'Android:

```kotlin
@get:Rule
val instantTaskExecutorRule = InstantTaskExecutorRule()
```

!!! warning "Sense aquesta Rule, els tests fallen!"
    Si intentes observar un `LiveData` en un test unitari sense `InstantTaskExecutorRule`, obtindràs un error com:
    ```
    java.lang.RuntimeException: Method getMainLooper in android.os.Looper not mocked.
    ```

## 5. Exemple pràctic: CalculadoraViewModel

Crearem un `CalculadoraViewModel` que suma dos nombres i exposa el resultat i els errors mitjançant LiveData.

### Primer: els tests (TDD)

Fitxer `app/src/test/java/com/example/app/CalculadoraViewModelTest.kt`:

```kotlin
import androidx.arch.core.executor.testing.InstantTaskExecutorRule
import org.junit.Assert.*
import org.junit.Before
import org.junit.Rule
import org.junit.Test

class CalculadoraViewModelTest {

    @get:Rule
    val instantTaskExecutorRule = InstantTaskExecutorRule()

    private lateinit var viewModel: CalculadoraViewModel

    @Before
    fun setUp() {
        viewModel = CalculadoraViewModel()
    }

    // --- sumar amb valors vàlids ---

    @Test
    fun sumar_dosNombresPositius_mostraResultat() {
        viewModel.sumar("10", "5")

        assertEquals("Resultat: 15.0", viewModel.resultat.value)
    }

    @Test
    fun sumar_ambDecimals_mostraResultat() {
        viewModel.sumar("3.5", "2.5")

        assertEquals("Resultat: 6.0", viewModel.resultat.value)
    }

    @Test
    fun sumar_ambNegatius_mostraResultat() {
        viewModel.sumar("-3", "8")

        assertEquals("Resultat: 5.0", viewModel.resultat.value)
    }

    @Test
    fun sumar_ambZeros_mostraResultat() {
        viewModel.sumar("0", "0")

        assertEquals("Resultat: 0.0", viewModel.resultat.value)
    }

    // --- sumar amb camps buits ---

    @Test
    fun sumar_ambPrimerCampBuit_mostraError() {
        viewModel.sumar("", "5")

        assertEquals("Has d'omplir els dos camps", viewModel.error.value)
    }

    @Test
    fun sumar_ambSegonCampBuit_mostraError() {
        viewModel.sumar("10", "")

        assertEquals("Has d'omplir els dos camps", viewModel.error.value)
    }

    @Test
    fun sumar_ambDosCampsBuits_mostraError() {
        viewModel.sumar("", "")

        assertEquals("Has d'omplir els dos camps", viewModel.error.value)
    }

    // --- sumar amb text invàlid ---

    @Test
    fun sumar_ambTextAlPrimerCamp_mostraError() {
        viewModel.sumar("abc", "5")

        assertEquals("Introdueix nombres vàlids", viewModel.error.value)
    }

    @Test
    fun sumar_ambTextAlSegonCamp_mostraError() {
        viewModel.sumar("5", "xyz")

        assertEquals("Introdueix nombres vàlids", viewModel.error.value)
    }

    // --- comportament de neteja ---

    @Test
    fun sumar_despresError_netejaError() {
        // Primer provoquem un error
        viewModel.sumar("", "")
        assertEquals("Has d'omplir els dos camps", viewModel.error.value)

        // Després fem una suma vàlida
        viewModel.sumar("3", "7")

        assertEquals("Resultat: 10.0", viewModel.resultat.value)
        assertNull(viewModel.error.value)
    }

    @Test
    fun sumar_despresResultat_netejaResultat() {
        // Primer fem una suma vàlida
        viewModel.sumar("3", "7")
        assertEquals("Resultat: 10.0", viewModel.resultat.value)

        // Després provoquem un error
        viewModel.sumar("abc", "5")

        assertEquals("Introdueix nombres vàlids", viewModel.error.value)
        assertEquals("", viewModel.resultat.value)
    }
}
```

### Després: la implementació

Fitxer `app/src/main/java/com/example/app/CalculadoraViewModel.kt`:

```kotlin
import androidx.lifecycle.LiveData
import androidx.lifecycle.MutableLiveData
import androidx.lifecycle.ViewModel

class CalculadoraViewModel : ViewModel() {

    private val _resultat = MutableLiveData<String>()
    val resultat: LiveData<String> = _resultat

    private val _error = MutableLiveData<String?>()
    val error: LiveData<String?> = _error

    fun sumar(text1: String, text2: String) {
        if (text1.isBlank() || text2.isBlank()) {
            _error.value = "Has d'omplir els dos camps"
            _resultat.value = ""
            return
        }

        val n1: Double? = text1.toDoubleOrNull()
        val n2: Double? = text2.toDoubleOrNull()

        if (n1 == null || n2 == null) {
            _error.value = "Introdueix nombres vàlids"
            _resultat.value = ""
            return
        }

        _error.value = null
        _resultat.value = "Resultat: ${n1 + n2}"
    }
}
```

## 6. Organització amb @Before i @After

JUnit proporciona anotacions per executar codi **abans** i **després** de cada test:

```kotlin
class CalculadoraViewModelTest {

    @get:Rule
    val instantTaskExecutorRule = InstantTaskExecutorRule()

    private lateinit var viewModel: CalculadoraViewModel

    @Before
    fun setUp() {
        // S'executa ABANS de cada test
        // Cada test obté un ViewModel nou i net
        viewModel = CalculadoraViewModel()
    }

    @After
    fun tearDown() {
        // S'executa DESPRÉS de cada test
        // Ideal per netejar recursos (fitxers, connexions, etc.)
    }

    @Test
    fun elMeuTest() {
        // Aquí viewModel ja està inicialitzat
    }
}
```

!!! note "Per què @Before?"
    Usant `@Before`, cada test obté una **instància nova** del ViewModel. Això garanteix que els tests són **independents**: un test no afecta els altres.

## 7. Executar tests des d'Android Studio

Hi ha diverses maneres d'executar els tests unitaris:

### Des de l'editor

- Fes clic a la **fletxa verda** que apareix al costat del nom del test o de la classe.
- Selecciona **Run 'NomDelTest'**.

### Des del menú

- **Run → Run...** i selecciona la classe de test.

### Des del terminal

```bash
./gradlew test
```

Aquesta comanda executa **tots** els tests unitaris del projecte.

!!! tip "Resultats dels tests"
    Android Studio mostra els resultats en una finestra amb:

    - Tests en **verd**: han passat correctament.
    - Tests en **vermell**: han fallat — fes clic per veure el detall de l'error.
