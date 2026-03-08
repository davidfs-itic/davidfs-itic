# Tests d'Integració (UI) amb Espresso

Els **tests d'integració** o **tests instrumentats** proven la interfície d'usuari real de l'aplicació. S'executen en un dispositiu o emulador Android, interactuant amb els elements de la pantalla com ho faria un usuari.

## 1. Què són els tests instrumentats?

A diferència dels tests unitaris (que s'executen a la JVM local), els tests instrumentats:

- S'executen en un **dispositiu real o emulador**.
- Poden interactuar amb la **interfície d'usuari** (clicar botons, escriure text, etc.).
- Comproven que la **pantalla mostra** el que esperem.
- Són més **lents** que els unitaris, però proven el comportament real de l'app.

- **On van**: carpeta `app/src/androidTest/java/`
- **S'executen a**: dispositiu o emulador Android
- **Velocitat**: lents (segons)
- **Què proven**: interacció amb la UI, navegació entre pantalles

## 2. Espresso: conceptes bàsics

**Espresso** és el framework de Google per escriure tests d'UI en Android. Funciona amb tres conceptes:

| Concepte | Què fa | Exemple |
|---|---|---|
| **ViewMatchers** | Trobar un element a la pantalla | `withId(R.id.btnCalcular)` |
| **ViewActions** | Fer una acció sobre l'element | `click()`, `typeText("hola")` |
| **ViewAssertions** | Comprovar l'estat de l'element | `matches(isDisplayed())` |

El patró bàsic d'Espresso és:

```kotlin
onView(ViewMatcher)      // Trobar l'element
    .perform(ViewAction)  // Fer una acció (opcional)
    .check(ViewAssertion) // Comprovar el resultat
```

## 3. Trobar elements (ViewMatchers)

```kotlin
import androidx.test.espresso.Espresso.onView
import androidx.test.espresso.matcher.ViewMatchers.*

// Per ID del recurs
onView(withId(R.id.etNom))

// Per text visible
onView(withText("Calcular"))

// Per hint (placeholder)
onView(withHint("Escriu el teu nom"))
```

## 4. Fer accions (ViewActions)

```kotlin
import androidx.test.espresso.action.ViewActions.*

// Escriure text en un EditText
onView(withId(R.id.etNombre1)).perform(typeText("42"))

// Fer clic a un botó
onView(withId(R.id.btnCalcular)).perform(click())

// Esborrar el text d'un camp
onView(withId(R.id.etNombre1)).perform(clearText())

// Tancar el teclat (important després de typeText!)
onView(withId(R.id.etNombre1)).perform(typeText("42"), closeSoftKeyboard())
```

!!! warning "Tanca el teclat!"
    Després d'escriure text amb `typeText()`, el teclat virtual queda obert i pot tapar altres elements. Afegeix sempre `closeSoftKeyboard()` per evitar problemes:
    ```kotlin
    .perform(typeText("text"), closeSoftKeyboard())
    ```

## 5. Fer comprovacions (ViewAssertions)

```kotlin
import androidx.test.espresso.assertion.ViewAssertions.matches
import androidx.test.espresso.matcher.ViewMatchers.*

// Comprovar que l'element és visible
onView(withId(R.id.tvResultat)).check(matches(isDisplayed()))

// Comprovar que mostra un text concret
onView(withId(R.id.tvResultat)).check(matches(withText("Resultat: 15")))

// Comprovar que NO és visible
onView(withId(R.id.tvError)).check(matches(not(isDisplayed())))
```

!!! note "Import de `not()`"
    Per usar `not()` en les comprovacions, cal importar:
    ```kotlin
    import org.hamcrest.Matchers.not
    ```

## 6. Exemple pràctic: testejar una calculadora

Suposem que tenim una Activity amb una calculadora senzilla que suma dos nombres:

### El layout (`activity_calculadora.xml`)

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <EditText
        android:id="@+id/etNombre1"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Nombre 1"
        android:inputType="numberDecimal" />

    <EditText
        android:id="@+id/etNombre2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Nombre 2"
        android:inputType="numberDecimal" />

    <Button
        android:id="@+id/btnSumar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Sumar" />

    <TextView
        android:id="@+id/tvResultat"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textSize="24sp" />

    <TextView
        android:id="@+id/tvError"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textColor="#FF0000"
        android:visibility="gone" />

</LinearLayout>
```

### El ViewModel (`CalculadoraViewModel.kt`)

La lògica de negoci viu al ViewModel (el mateix que hem testat amb tests unitaris):

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

### L'Activity (`CalculadoraActivity.kt`)

L'Activity només s'encarrega de la UI: recull les dades, crida el ViewModel i observa els resultats:

```kotlin
import androidx.activity.viewModels

class CalculadoraActivity : AppCompatActivity() {

    private val viewModel: CalculadoraViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_calculadora)

        val etNombre1 = findViewById<EditText>(R.id.etNombre1)
        val etNombre2 = findViewById<EditText>(R.id.etNombre2)
        val btnSumar = findViewById<Button>(R.id.btnSumar)
        val tvResultat = findViewById<TextView>(R.id.tvResultat)
        val tvError = findViewById<TextView>(R.id.tvError)

        btnSumar.setOnClickListener {
            val text1 = etNombre1.text.toString()
            val text2 = etNombre2.text.toString()
            viewModel.sumar(text1, text2)
        }

        viewModel.resultat.observe(this) { text ->
            tvResultat.text = text
        }

        viewModel.error.observe(this) { missatge ->
            if (missatge != null) {
                tvError.visibility = View.VISIBLE
                tvError.text = missatge
            } else {
                tvError.visibility = View.GONE
            }
        }
    }
}
```

!!! note "Separació de responsabilitats"
    - **ViewModel**: conté tota la lògica (validacions, càlculs). Es pot testejar amb **tests unitaris** ràpids.
    - **Activity**: només connecta la UI amb el ViewModel (observers + click listeners). Es testeja amb **tests d'UI** (Espresso).

### Els tests d'UI

Fitxer `app/src/androidTest/java/com/example/app/CalculadoraActivityTest.kt`:

```kotlin
import androidx.test.espresso.Espresso.onView
import androidx.test.espresso.action.ViewActions.*
import androidx.test.espresso.assertion.ViewAssertions.matches
import androidx.test.espresso.matcher.ViewMatchers.*
import androidx.test.ext.junit.rules.ActivityScenarioRule
import androidx.test.ext.junit.runners.AndroidJUnit4
import org.hamcrest.Matchers.not
import org.junit.Rule
import org.junit.Test
import org.junit.runner.RunWith

@RunWith(AndroidJUnit4::class)
class CalculadoraActivityTest {

    @get:Rule
    val activityRule = ActivityScenarioRule(CalculadoraActivity::class.java)

    @Test
    fun sumar_dosNombresValids_mostraResultat() {
        // Escriure els nombres
        onView(withId(R.id.etNombre1))
            .perform(typeText("10"), closeSoftKeyboard())
        onView(withId(R.id.etNombre2))
            .perform(typeText("5"), closeSoftKeyboard())

        // Clicar el botó
        onView(withId(R.id.btnSumar)).perform(click())

        // Verificar el resultat
        onView(withId(R.id.tvResultat))
            .check(matches(withText("Resultat: 15.0")))
    }

    @Test
    fun sumar_ambCampsBuits_mostraError() {
        // No escrivim res, directament cliquem
        onView(withId(R.id.btnSumar)).perform(click())

        // Verificar que es mostra l'error
        onView(withId(R.id.tvError))
            .check(matches(isDisplayed()))
        onView(withId(R.id.tvError))
            .check(matches(withText("Has d'omplir els dos camps")))
    }

    @Test
    fun sumar_ambTextInvalid_mostraError() {
        onView(withId(R.id.etNombre1))
            .perform(typeText("abc"), closeSoftKeyboard())
        onView(withId(R.id.etNombre2))
            .perform(typeText("5"), closeSoftKeyboard())

        onView(withId(R.id.btnSumar)).perform(click())

        onView(withId(R.id.tvError))
            .check(matches(isDisplayed()))
        onView(withId(R.id.tvError))
            .check(matches(withText("Introdueix nombres vàlids")))
    }

    @Test
    fun sumar_despresError_amagaError() {
        // Primer provoquem un error
        onView(withId(R.id.btnSumar)).perform(click())
        onView(withId(R.id.tvError)).check(matches(isDisplayed()))

        // Després fem una suma vàlida
        onView(withId(R.id.etNombre1))
            .perform(typeText("3"), closeSoftKeyboard())
        onView(withId(R.id.etNombre2))
            .perform(typeText("7"), closeSoftKeyboard())
        onView(withId(R.id.btnSumar)).perform(click())

        // L'error ha de desaparèixer
        onView(withId(R.id.tvError))
            .check(matches(not(isDisplayed())))
        onView(withId(R.id.tvResultat))
            .check(matches(withText("Resultat: 10.0")))
    }

    @Test
    fun elementsInicials_sónVisibles() {
        onView(withId(R.id.etNombre1)).check(matches(isDisplayed()))
        onView(withId(R.id.etNombre2)).check(matches(isDisplayed()))
        onView(withId(R.id.btnSumar)).check(matches(isDisplayed()))
    }
}
```

## 7. ActivityScenarioRule

L'anotació `@get:Rule` amb `ActivityScenarioRule` s'encarrega de:

- **Llançar l'Activity** abans de cada test.
- **Tancar l'Activity** després de cada test.
- Gestionar el cicle de vida correctament.

```kotlin
@get:Rule
val activityRule = ActivityScenarioRule(CalculadoraActivity::class.java)
```

!!! note "Diferència entre `@Rule` i `@get:Rule`"
    En Kotlin, cal usar `@get:Rule` (no `@Rule`) perquè l'anotació s'apliqui al getter de la propietat, que és el que JUnit espera.

## 8. Executar tests instrumentats des d'Android Studio

### Requisits previs

- Tenir un **emulador** configurat o un **dispositiu** connectat.
- L'emulador/dispositiu ha d'estar encès i funcionant.

### Des de l'editor

- Fes clic a la **fletxa verda** al costat del nom del test o de la classe.
- Selecciona **Run 'NomDelTest'**.

### Des del terminal

```bash
./gradlew connectedAndroidTest
```

!!! warning "Els tests instrumentats són lents"
    Com que s'executen en un dispositiu real o emulador, tarden molt més que els unitaris. Per això, reserva els tests d'UI per provar la interfície i usa tests unitaris per a la lògica de negoci.

## 9. Resum d'imports

Aquí tens tots els imports que necessitaràs normalment en un test d'Espresso:

```kotlin
// Espresso
import androidx.test.espresso.Espresso.onView
import androidx.test.espresso.action.ViewActions.*
import androidx.test.espresso.assertion.ViewAssertions.matches
import androidx.test.espresso.matcher.ViewMatchers.*

// JUnit i AndroidJUnit4
import androidx.test.ext.junit.rules.ActivityScenarioRule
import androidx.test.ext.junit.runners.AndroidJUnit4
import org.junit.Rule
import org.junit.Test
import org.junit.runner.RunWith

// Matchers addicionals
import org.hamcrest.Matchers.not
```
