# Activities

Una `Activity` és un component de l'aplicació que representa una pantalla amb la qual l'usuari pot interactuar. És el punt d'entrada a la UI: el sistema crea i destrueix les activities segons les accions de l'usuari (obrir l'app, girar el dispositiu, prémer enrere...) i les notifica mitjançant els mètodes del seu cicle de vida.

Documentació oficial:

- [Introduction to activities](https://developer.android.com/guide/components/activities/intro-activities)
- [The activity lifecycle](https://developer.android.com/guide/components/activities/activity-lifecycle)
- [Intents and intent filters](https://developer.android.com/guide/components/intents-filters)

## 1. Crear una Activity

Una Activity necessita tres coses: una classe que hereti d'`AppCompatActivity` (o `ComponentActivity` si es treballa amb Compose), un layout i una entrada al `AndroidManifest.xml`.

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
    }
}
```

En una app amb Jetpack Compose, en lloc de `setContentView()` es crida `setContent { ... }` i el layout es descriu amb composables (vegeu [Jetpack Compose](./Jetpack_compose/index.md)).

### Declarar l'Activity al manifest

Cada Activity ha d'estar declarada al manifest, dins de l'etiqueta `<application>`. Si no hi és, `startActivity()` llançarà una excepció `ActivityNotFoundException`.

```xml
<application ...>

    <!-- Activity principal: la que s'obre en tocar la icona de l'app -->
    <activity
        android:name=".MainActivity"
        android:exported="true">
        <intent-filter>
            <action android:name="android.intent.action.MAIN" />
            <category android:name="android.intent.category.LAUNCHER" />
        </intent-filter>
    </activity>

    <!-- Activity secundària: només es pot obrir des de la nostra app -->
    <activity
        android:name=".DetailActivity"
        android:exported="false"
        android:parentActivityName=".MainActivity" />

</application>
```

- **`intent-filter` amb `MAIN` i `LAUNCHER`:** marca l'Activity que s'obre des del llançador del dispositiu.
- **`android:exported`:** indica si altres aplicacions poden obrir aquesta Activity. Des d'Android 12 (API 31) és obligatori declarar-ho en les activities que tenen `intent-filter`.
- **`android:parentActivityName`:** defineix la pantalla "pare", que s'utilitza per a la navegació cap amunt (botó de fletxa de l'appbar).

!!! info
    Si l'aplicació segueix el patró d'una sola Activity amb diversos [Fragments](./fragments.md) o amb [Navigation Compose](./Jetpack_compose/navigationcompose.md), només caldrà declarar una Activity. Vegeu l'apartat [7. Una Activity o moltes?](#7-una-activity-o-moltes).

## 2. Cicle de vida d'una Activity

Una Activity no viu per sempre: el sistema la crea, la mostra, l'amaga i la destrueix, i avisa de cada canvi cridant un mètode. Sobreescrivint aquests mètodes podem executar codi en el moment adequat (per exemple, aturar una animació quan l'Activity deixa de ser visible).

```
        onCreate()
            |
        onStart()  <----------- onRestart()
            |                        ^
        onResume()                   |
            |                        |
   [Activity en primer pla]          |
            |                        |
        onPause()                    |
            |                        |
        onStop()  -------------------+   (l'usuari torna a l'Activity)
            |
        onDestroy()
```

| Mètode | Quan es crida | Ús habitual |
|:--|:--|:--|
| `onCreate()` | Quan es crea l'Activity | Inflar el layout, crear ViewModels, inicialitzar la UI |
| `onStart()` | Abans que l'Activity sigui visible | Registrar recursos que només calen si és visible |
| `onResume()` | L'Activity és en primer pla i interactiva | Reprendre animacions, càmera, sensors |
| `onPause()` | L'Activity perd el focus (pot seguir parcialment visible) | Pausar animacions, guardar dades lleugeres |
| `onStop()` | L'Activity deixa de ser visible | Alliberar recursos, guardar dades |
| `onRestart()` | L'Activity torna a ser visible després d'un `onStop()` | Poc habitual |
| `onDestroy()` | L'Activity es destrueix | Alliberar els recursos restants |

Es pot veure el cicle de vida real afegint un `Log` a cada mètode:

```kotlin
class MainActivity : AppCompatActivity() {

    private val TAG = "CicleDeVida"

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        Log.d(TAG, "onCreate (savedInstanceState = $savedInstanceState)")
    }

    override fun onStart() {
        super.onStart()
        Log.d(TAG, "onStart")
    }

    override fun onResume() {
        super.onResume()
        Log.d(TAG, "onResume")
    }

    override fun onPause() {
        super.onPause()
        Log.d(TAG, "onPause")
    }

    override fun onStop() {
        super.onStop()
        Log.d(TAG, "onStop")
    }

    override fun onDestroy() {
        super.onDestroy()
        Log.d(TAG, "onDestroy")
    }
}
```

!!! warning
    Cal cridar sempre a `super.onXxx()` en sobreescriure un mètode del cicle de vida. Si s'oblida, Android llançarà una excepció `SuperNotCalledException`.

### Seqüències habituals

Es poden comprovar amb el `Log` anterior:

- **Obrir l'app:** `onCreate` -> `onStart` -> `onResume`.
- **Prémer Home:** `onPause` -> `onStop`. L'Activity segueix viva en memòria.
- **Tornar a l'app:** `onRestart` -> `onStart` -> `onResume`.
- **Prémer enrere:** `onPause` -> `onStop` -> `onDestroy`. L'Activity s'acaba.
- **Girar el dispositiu:** `onPause` -> `onStop` -> `onDestroy` -> `onCreate` -> `onStart` -> `onResume`. L'Activity es destrueix i es crea de nou (vegeu l'apartat següent).
- **Obrir un diàleg del sistema o una altra Activity translúcida:** només `onPause`, ja que l'Activity encara és parcialment visible.

## 3. Canvis de configuració i pèrdua d'estat

Un **canvi de configuració** és qualsevol canvi de l'entorn del dispositiu que afecta la UI: rotació de la pantalla, canvi d'idioma, mode fosc, mida de la lletra, canvi de mida de la finestra (multi-finestra, plegables)...

Quan es produeix, Android **destrueix l'Activity i en crea una de nova** perquè carregui els recursos adequats (per exemple, `layout-land/` en horitzontal). La instància antiga desapareix, i amb ella totes les seves variables.

```kotlin
class ComptadorActivity : AppCompatActivity() {

    private var comptador = 0   // Es perd en girar la pantalla!

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_comptador)
        // ...
    }
}
```

Si l'usuari ha premut el botó 5 vegades i gira el mòbil, el comptador torna a 0. Hi ha diverses maneres d'evitar-ho, segons el tipus de dada.

### 3.1. onSaveInstanceState

Abans de destruir l'Activity, el sistema crida `onSaveInstanceState(outState: Bundle)`. Es pot escriure a aquest `Bundle` l'estat que volem conservar. Després de la recreació, el mateix `Bundle` arriba com a paràmetre `savedInstanceState` d'`onCreate()`.

```kotlin
class ComptadorActivity : AppCompatActivity() {

    private lateinit var binding: ActivityComptadorBinding
    private var comptador = 0

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityComptadorBinding.inflate(layoutInflater)
        setContentView(binding.root)

        // Si savedInstanceState és null, és la primera vegada que es crea l'Activity
        comptador = savedInstanceState?.getInt(KEY_COMPTADOR) ?: 0
        binding.txtComptador.text = comptador.toString()

        binding.btnSumar.setOnClickListener {
            comptador++
            binding.txtComptador.text = comptador.toString()
        }
    }

    override fun onSaveInstanceState(outState: Bundle) {
        super.onSaveInstanceState(outState)
        outState.putInt(KEY_COMPTADOR, comptador)
    }

    // Kotlin no té membres static: el companion object els substitueix.
    // Les seves propietats i funcions pertanyen a la classe, no a cada instància,
    // i s'hi accedeix directament (KEY_COMPTADOR) des de dins de la classe.
    companion object {
        // const val: constant fixada en temps de compilació (només tipus primitius i String).
        // La clau és la mateixa a onSaveInstanceState i a onCreate, així evitem errors tipogràfics.
        private const val KEY_COMPTADOR = "comptador"
    }
}
```

Punts importants:

- `savedInstanceState` és **`null`** quan l'Activity es crea per primera vegada, i conté el `Bundle` quan es tracta d'una recreació.
- Existeix també `onRestoreInstanceState()`, que es crida després d'`onStart()` només si hi ha un estat guardat. En la pràctica és més senzill restaurar directament a `onCreate()`.
- Les vistes amb `android:id` (per exemple un `EditText`) guarden i restauren el seu contingut automàticament. Això no passa amb les vistes sense `id`, ni amb les variables de l'Activity.
- El `Bundle` és petit i està pensat només per a **estat de la UI** (posició d'un scroll, text d'un formulari, pestanya seleccionada). No s'hi han de guardar llistes grans ni imatges (vegeu el límit de mida a l'[apartat 5.3](#53-limit-de-mida-del-bundle)).

### 3.2. ViewModel

Per a l'estat que ha de sobreviure a una rotació i que no és trivial (dades carregades d'una API o d'una base de dades, resultat d'un càlcul), l'opció recomanada és el **ViewModel**: sobreviu als canvis de configuració i només es destrueix quan l'Activity s'acaba de veritat.

```kotlin
class ComptadorViewModel : ViewModel() {
    var comptador = 0
        private set

    fun sumar() { comptador++ }
}

class ComptadorActivity : AppCompatActivity() {

    private val viewModel: ComptadorViewModel by viewModels()

    // ...
}
```

Tot el detall està a [ViewModel](../Arquitectura/viewmodel.md#supervivencia-a-canvis-de-configuracio-rotacions).

### 3.3. Mort del procés i SavedStateHandle

Hi ha una altra situació on es perd l'estat: quan l'aplicació està en segon pla i el sistema **mata el procés** per alliberar memòria. Quan l'usuari torna a l'app, Android recrea l'Activity i li dona el `savedInstanceState` guardat, però **el ViewModel no sobreviu** a la mort del procés.

Per cobrir aquest cas el ViewModel pot rebre un `SavedStateHandle`, que guarda els valors al mateix mecanisme que `onSaveInstanceState`:

```kotlin
class ComptadorViewModel(private val state: SavedStateHandle) : ViewModel() {

    var comptador: Int
        get() = state["comptador"] ?: 0
        set(value) { state["comptador"] = value }
}
```

El delegat `by viewModels()` proporciona el `SavedStateHandle` automàticament, sense necessitat de crear una Factory.

!!! tip
    Per provar la mort del procés, activa l'opció de desenvolupador **"No conservis les activitats"** (Don't keep activities) al dispositiu. Cada vegada que surtis de l'Activity, el sistema la destruirà, com si li faltés memòria.

### 3.4. Resum: on guardar cada dada

| On es guarda | Gir de pantalla | Mort del procés | L'usuari tanca l'app |
|:--|:--:|:--:|:--:|
| Variable de l'Activity | No sobreviu | No sobreviu | No sobreviu |
| `onSaveInstanceState` / `SavedStateHandle` | Sobreviu | Sobreviu | No sobreviu |
| `ViewModel` | Sobreviu | No sobreviu | No sobreviu |
| `SharedPreferences`, `DataStore`, `Room` | Sobreviu | Sobreviu | Sobreviu |

En Jetpack Compose l'equivalent d'`onSaveInstanceState` és `rememberSaveable`. Vegeu [Estats en Jetpack Compose](./Jetpack_compose/estats.md#2-remember-i-el-cicle-de-vida-de-lactivity).

!!! warning
    Es pot evitar la recreació de l'Activity amb `android:configChanges="orientation|screenSize"` al manifest, però **no és recomanable**: l'Activity ha de gestionar manualment cada canvi i no rep els recursos alternatius de forma automàtica. Només cal recórrer-hi en casos molt concrets (per exemple, un joc o un reproductor de vídeo).

## 4. Canviar d'Activity: Intents

Per anar d'una Activity a una altra s'utilitza un **Intent**, un objecte que descriu una operació que es vol realitzar. Quan l'Activity destí és una classe de la nostra pròpia aplicació, s'anomena **Intent explícit**.

```kotlin
// Des de MainActivity
binding.btnObrir.setOnClickListener {
    val intent = Intent(this, DetailActivity::class.java)
    startActivity(intent)
}
```

El sistema crea `DetailActivity`, la posa a la part superior de la **pila d'activities** (back stack) i `MainActivity` passa per `onPause` i `onStop`.

### Tancar una Activity

Per tornar a l'Activity anterior s'utilitza `finish()`, que destrueix l'Activity actual i la treu de la pila. És el que fa també el botó enrere del sistema.

```kotlin
binding.btnTancar.setOnClickListener {
    finish()
}
```

Per evitar que l'usuari pugui tornar a una Activity amb el botó enrere (per exemple, una pantalla de login un cop identificat), es pot cridar `finish()` just després de `startActivity()`:

```kotlin
startActivity(Intent(this, MainActivity::class.java))
finish()
```

### Intents implícits

Un **Intent implícit** no indica quina classe s'ha d'obrir, sinó l'acció que es vol fer, i el sistema busca una aplicació que la pugui gestionar.

Un exemple habitual és enviar un correu electrònic. L'app no envia el missatge: obre el client de correu instal·lat (Gmail, Outlook...) amb el destinatari, l'assumpte i el cos ja emplenats, i és l'usuari qui prem Enviar.

```kotlin
private fun enviarCorreu() {
    val intent = Intent(Intent.ACTION_SENDTO).apply {
        // "mailto:" fa que només responguin les aplicacions de correu
        data = Uri.parse("mailto:")
        putExtra(Intent.EXTRA_EMAIL, arrayOf("professor@exemple.cat"))
        putExtra(Intent.EXTRA_SUBJECT, "Consulta sobre l'activitat")
        putExtra(Intent.EXTRA_TEXT, "Hola, tinc un dubte sobre...")
    }

    try {
        startActivity(intent)
    } catch (e: ActivityNotFoundException) {
        // El dispositiu no té cap aplicació de correu instal·lada
        Toast.makeText(this, "No hi ha cap aplicació de correu", Toast.LENGTH_SHORT).show()
    }
}
```

- **`ACTION_SENDTO` amb `mailto:`:** és la manera correcta de limitar el resultat a aplicacions de correu. Amb `ACTION_SEND` i `type = "text/plain"` també apareixerien aplicacions de missatgeria i xarxes socials.
- **`EXTRA_EMAIL`:** ha de ser un `Array<String>`, encara que només hi hagi un destinatari.
- **`try/catch`:** si cap aplicació pot gestionar l'Intent, `startActivity()` llança `ActivityNotFoundException`. Cal capturar-la perquè l'app no es tanqui.

!!! info
    Aquest Intent no necessita cap permís al manifest, perquè és l'aplicació de correu qui envia el missatge, no la nostra.

## 5. Passar dades entre Activities

Un `Intent` pot transportar dades addicionals, anomenades **extras**, que internament es guarden en un `Bundle`: un mapa de parells clau-valor.

### 5.1. Dades simples

Els tipus primitius i els `String` es poden afegir directament amb `putExtra()`.

Al costat que envia (`MainActivity`):

```kotlin
val intent = Intent(this, DetailActivity::class.java).apply {
    putExtra(DetailActivity.EXTRA_NOM, "Anna")
    putExtra(DetailActivity.EXTRA_EDAT, 17)
    putExtra(DetailActivity.EXTRA_MAJOR_EDAT, false)
}
startActivity(intent)
```

Al costat que rep (`DetailActivity`):

```kotlin
class DetailActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_detail)

        val nom = intent.getStringExtra(EXTRA_NOM) ?: "Desconegut"
        val edat = intent.getIntExtra(EXTRA_EDAT, 0)          // 0 és el valor per defecte
        val majorEdat = intent.getBooleanExtra(EXTRA_MAJOR_EDAT, false)
    }

    companion object {
        const val EXTRA_NOM = "extra_nom"
        const val EXTRA_EDAT = "extra_edat"
        const val EXTRA_MAJOR_EDAT = "extra_major_edat"
    }
}
```

!!! tip
    Declara les claus com a constants al `companion object` de l'Activity que **rep** les dades. Així evites errors tipogràfics en cadenes repetides i queda documentat què espera l'Activity.

Els mètodes `getXxxExtra()` de tipus primitiu (`getIntExtra`, `getBooleanExtra`...) demanen un valor per defecte, mentre que els d'objectes (`getStringExtra`) retornen `null` si la clau no existeix.

### Utilitzar un Bundle explícit

Quan hi ha moltes dades és més net construir un `Bundle` amb `bundleOf()` (dependència `androidx.core:core-ktx`) i afegir-lo d'un sol cop amb `putExtras()`:

```kotlin
val extras = bundleOf(
    DetailActivity.EXTRA_NOM to "Anna",
    DetailActivity.EXTRA_EDAT to 17
)
val intent = Intent(this, DetailActivity::class.java).apply {
    putExtras(extras)
}
startActivity(intent)

// A l'Activity destí
val nom = intent.extras?.getString(DetailActivity.EXTRA_NOM)
```

El mateix mecanisme de `Bundle` és el que utilitzen els [Fragments](./fragments.md) amb la propietat `arguments`.

### 5.2. Dades complexes: Parcelable

Un `Bundle` només accepta tipus bàsics (números, `String`, `Boolean`, arrays d'aquests tipus...). Per passar un **objecte propi** (una `Task`, un `Usuari`...) cal que la classe sigui **`Parcelable`**, la manera nativa d'Android de convertir un objecte en dades que es puguin enviar a través d'un `Bundle`.

Implementar `Parcelable` a mà requereix bastant codi repetitiu. Amb el plugin `kotlin-parcelize` n'hi ha prou amb una anotació.

#### Pas 1: Afegir el plugin

```kotlin
// build.gradle.kts (Module: app)
plugins {
    id("kotlin-parcelize")
    // ... la resta de plugins
}
```

#### Pas 2: Anotar la classe

```kotlin
import android.os.Parcelable
import kotlinx.parcelize.Parcelize

@Parcelize
data class Usuari(
    val id: Int,
    val nom: String,
    val correu: String,
    val actiu: Boolean = true
) : Parcelable
```

Totes les propietats del constructor han de ser tipus que es puguin empaquetar: primitius, `String`, altres classes `Parcelable`, llistes d'aquests tipus, etc. Si una classe conté una altra classe, aquesta també ha de ser `@Parcelize`.

```kotlin
@Parcelize
data class Adreca(val carrer: String, val ciutat: String) : Parcelable

@Parcelize
data class Client(
    val nom: String,
    val adreca: Adreca,                 // Adreca també és Parcelable
    val telefons: List<String>
) : Parcelable
```

#### Pas 3: Enviar l'objecte

```kotlin
val usuari = Usuari(id = 1, nom = "Anna", correu = "anna@exemple.cat")

val intent = Intent(this, DetailActivity::class.java).apply {
    putExtra(DetailActivity.EXTRA_USUARI, usuari)
}
startActivity(intent)
```

#### Pas 4: Rebre l'objecte

Des d'Android 13 (API 33) el mètode `getParcelableExtra(String)` està deprecated perquè no és segur respecte al tipus. La versió recomanada és passar-hi la classe esperada, i per mantenir la compatibilitat amb versions anteriors es fa servir `IntentCompat` (de `androidx.core`):

```kotlin
val usuari = IntentCompat.getParcelableExtra(
    intent,
    EXTRA_USUARI,
    Usuari::class.java
)

usuari?.let {
    binding.txtNom.text = it.nom
    binding.txtCorreu.text = it.correu
}
```

Per llegir un `Parcelable` d'un `Bundle` qualsevol (per exemple els `arguments` d'un Fragment) hi ha l'equivalent `BundleCompat.getParcelable(bundle, clau, Classe::class.java)`.

Per passar una llista d'objectes, s'utilitza `putParcelableArrayListExtra()` / `IntentCompat.getParcelableArrayListExtra()`, o bé s'encapsula la llista dins d'una altra classe `@Parcelize`.

!!! info
    `Serializable` (de Java) també permet passar objectes, però és molt més lent perquè utilitza reflexió. En Android es prefereix sempre `Parcelable`. Un altre exemple d'ús de `@Parcelize` es pot veure a [DialogFragment amb ViewModel](../Arquitectura/dialogfragmentviewmodel.md).

### 5.3. Límit de mida del Bundle

Tots els `Bundle` (extras d'un Intent, `onSaveInstanceState`, arguments d'un Fragment) viatgen a través d'un mecanisme del sistema (Binder) amb un buffer d'aproximadament **1 MB compartit** per a totes les transaccions de l'aplicació. Si es supera, l'aplicació peta amb una `TransactionTooLargeException`.

Per tant, no s'ha de passar mai com a extra:

- Imatges (`Bitmap`).
- Llistes molt llargues.
- Respostes senceres d'una API.

En aquests casos, es passa només un **identificador** (o una URI) i la segona pantalla carrega les dades des del repositori, la base de dades o el ViewModel.

```kotlin
// Malament: enviar tot l'objecte amb una llista enorme
putExtra("comandes", llistaDeMilComandes)

// Bé: enviar l'identificador i carregar les dades a l'Activity destí
putExtra("id_client", client.id)
```

## 6. Retornar un resultat a l'Activity anterior

De vegades una Activity en llança una altra per obtenir-ne una dada (escollir un contacte, editar un element...). Per fer-ho s'utilitza l'**Activity Result API**, que substitueix l'antic `startActivityForResult()` (deprecated).

### Activity que espera el resultat

El `launcher` es registra com a **propietat** de l'Activity, abans que arribi a l'estat `STARTED`:

```kotlin
class MainActivity : AppCompatActivity() {

    private val editarLauncher = registerForActivityResult(
        ActivityResultContracts.StartActivityForResult()
    ) { result ->
        if (result.resultCode == RESULT_OK) {
            val nouNom = result.data?.getStringExtra(EditActivity.EXTRA_NOM_EDITAT)
            binding.txtNom.text = nouNom
        }
    }

    // ...

    private fun obrirEditor() {
        val intent = Intent(this, EditActivity::class.java).apply {
            putExtra(EditActivity.EXTRA_NOM, binding.txtNom.text.toString())
        }
        editarLauncher.launch(intent)
    }
}
```

### Activity que retorna el resultat

```kotlin
class EditActivity : AppCompatActivity() {

    // ...

    private fun desar() {
        val resultat = Intent().apply {
            putExtra(EXTRA_NOM_EDITAT, binding.edtNom.text.toString())
        }
        setResult(RESULT_OK, resultat)
        finish()
    }

    companion object {
        const val EXTRA_NOM = "extra_nom"
        const val EXTRA_NOM_EDITAT = "extra_nom_editat"
    }
}
```

Si l'usuari surt amb el botó enrere sense cridar `setResult()`, el `resultCode` serà `RESULT_CANCELED`.

!!! info
    El mateix `registerForActivityResult` s'utilitza amb altres contractes ja preparats, com `RequestPermission` (per demanar permisos, vegeu [Voice Recognition](./voicerecognition.md)), `TakePicture` o `GetContent`.

## 7. Una Activity o moltes?

Tradicionalment, cada pantalla de l'aplicació era una Activity. Avui la recomanació d'Android és l'arquitectura **d'una sola Activity** (single-activity), que actua de contenidor, i on la navegació entre pantalles es fa amb:

- [Fragments](./fragments.md) i el Navigation Component, en aplicacions amb vistes XML.
- [Navigation Compose](./Jetpack_compose/navigationcompose.md), en aplicacions amb Jetpack Compose.

Els avantatges són que les transicions són més senzilles, es comparteix fàcilment l'estat entre pantalles amb un ViewModel compartit i no cal passar dades a través d'Intents ni Parcelables.

Segueix sent habitual tenir més d'una Activity per a casos concrets, com una pantalla de login o d'onboarding separada, o per obrir la nostra app des d'una altra amb un Intent.

Altres temes relacionats amb l'Activity que es tracten en documents separats:

- [Splash Screen](./splashscreen.md): pantalla de presentació de l'Activity inicial.
- [Edge-to-edge i System Bars](./layoutactivities.md): com es dibuixa l'Activity sota les barres del sistema.
- [ViewBinding](./viewbinding.md): accés a les vistes del layout de l'Activity.
- [La classe Application](./application.md): estat global anterior a qualsevol Activity.
