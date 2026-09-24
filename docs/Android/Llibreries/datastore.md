# DataStore Preferences

## 1. Introducció

**DataStore** és la solució moderna de Google per emmagatzemar dades clau-valor de forma local en aplicacions Android. Substitueix les antigues `SharedPreferences` amb una API asíncrona basada en **Kotlin Coroutines** i **Flow**.

DataStore Preferences és ideal per guardar configuracions d'usuari, preferències de l'aplicació o petites dades que no requereixen una base de dades completa.

Documentació oficial: [DataStore - Android Developers](https://developer.android.com/topic/libraries/architecture/datastore)

## 2. DataStore vs SharedPreferences

| Característica | SharedPreferences | DataStore Preferences |
|---|---|---|
| API | Síncrona (bloqueja el fil principal) | Asíncrona (Coroutines + Flow) |
| Seguretat de fils | No garantida | Garantida |
| Gestió d'errors | Excepcions no controlades | Gestió amb `try/catch` i `Flow` |
| Transaccional | No | Sí |

!!! warning "SharedPreferences obsoletes"
    Google recomana migrar de `SharedPreferences` a `DataStore` en tots els projectes nous. SharedPreferences pot causar bloquejos al fil principal i no gestiona correctament els accessos concurrents.

## 3. Dependències

Afegir la dependència al fitxer `build.gradle.kts` **(Module: app)**:

```kotlin
dependencies {
    implementation("androidx.datastore:datastore-preferences:1.1.4")
}
```

O amb el catàleg de versions `libs.versions.toml`:

```toml
[versions]
datastore = "1.1.4"

[libraries]
datastore-preferences = { group = "androidx.datastore", name = "datastore-preferences", version.ref = "datastore" }
```

```kotlin
dependencies {
    implementation(libs.datastore.preferences)
}
```

## 4. Crear el DataStore

Es crea una instància de DataStore com a propietat d'extensió a nivell de fitxer. Normalment es defineix en un fitxer a part o al fitxer de la classe que l'utilitza.

```kotlin
import android.content.Context
import androidx.datastore.preferences.preferencesDataStore

// Creem el DataStore amb el nom "settings"
val Context.dataStore by preferencesDataStore(name = "settings")
```

!!! info "Per què és una extensió de Context?"
    DataStore necessita accedir al sistema de fitxers de l'aplicació per guardar les dades. En Android, qualsevol accés a fitxers locals passa pel `Context` (és qui coneix el directori intern de l'app). Fer-ho com a [funció d'extensió](../Kotlin/fextensio.md) de `Context` permet que des de qualsevol lloc on tinguis un `Context` (Activity, Service, Application...) puguis escriure `context.dataStore` directament, sense haver de crear cap classe extra ni passar paràmetres.

!!! info "Una sola instància (propietat delegada)"
    El delegat `preferencesDataStore` (la paraula clau `by`) garanteix que només es crea **una única instància** del DataStore (patró Singleton). Si cada cop que s'accedís es creés una instància nova, es corromprien les dades perquè hi hauria múltiples escriptors al mateix fitxer. Internament, el delegat utilitza un mecanisme lazy i thread-safe.

## 5. Definir les claus

Les claus es defineixen amb funcions específiques segons el tipus de dada:

```kotlin
import androidx.datastore.preferences.core.booleanPreferencesKey
import androidx.datastore.preferences.core.intPreferencesKey
import androidx.datastore.preferences.core.stringPreferencesKey

object PreferencesKeys {
    val NOM_USUARI = stringPreferencesKey("nom_usuari")
    val EDAT = intPreferencesKey("edat")
    val MODE_FOSC = booleanPreferencesKey("mode_fosc")
}
```

Tipus de claus disponibles:

| Funció | Tipus |
|---|---|
| `stringPreferencesKey` | `String` |
| `intPreferencesKey` | `Int` |
| `booleanPreferencesKey` | `Boolean` |
| `floatPreferencesKey` | `Float` |
| `longPreferencesKey` | `Long` |
| `doublePreferencesKey` | `Double` |
| `stringSetPreferencesKey` | `Set<String>` |

## 6. Escriure dades

Per escriure dades s'utilitza la funció `edit`, que és una funció **suspend** (necessita una coroutine):

```kotlin
import androidx.datastore.preferences.core.edit

suspend fun guardarNomUsuari(context: Context, nom: String) {
    context.dataStore.edit { preferences ->
        preferences[PreferencesKeys.NOM_USUARI] = nom
    }
}
```

Es poden escriure diverses claus alhora dins del mateix bloc `edit`:

```kotlin
suspend fun guardarPerfil(context: Context, nom: String, edat: Int) {
    context.dataStore.edit { preferences ->
        preferences[PreferencesKeys.NOM_USUARI] = nom
        preferences[PreferencesKeys.EDAT] = edat
    }
}
```

## 7. Llegir dades

La lectura es fa mitjançant un `Flow`, que emet un nou valor cada cop que les dades canvien:

```kotlin
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.map

fun obtenirNomUsuari(context: Context): Flow<String> {
    return context.dataStore.data.map { preferences ->
        preferences[PreferencesKeys.NOM_USUARI] ?: "Sense nom"
    }
}
```

L'operador `?:` permet definir un **valor per defecte** quan la clau encara no existeix.

## 8. Exemple complet amb ViewModel

### 8.1. Fitxer de DataStore

```kotlin
// PreferencesManager.kt
import android.content.Context
import androidx.datastore.preferences.core.booleanPreferencesKey
import androidx.datastore.preferences.core.edit
import androidx.datastore.preferences.core.stringPreferencesKey
import androidx.datastore.preferences.preferencesDataStore
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.map

val Context.dataStore by preferencesDataStore(name = "settings")

object PreferencesKeys {
    val NOM_USUARI = stringPreferencesKey("nom_usuari")
    val MODE_FOSC = booleanPreferencesKey("mode_fosc")
}

class PreferencesManager(private val context: Context) {

    val nomUsuari: Flow<String> = context.dataStore.data.map { preferences ->
        preferences[PreferencesKeys.NOM_USUARI] ?: ""
    }

    val modeFosc: Flow<Boolean> = context.dataStore.data.map { preferences ->
        preferences[PreferencesKeys.MODE_FOSC] ?: false
    }

    suspend fun guardarNomUsuari(nom: String) {
        context.dataStore.edit { preferences ->
            preferences[PreferencesKeys.NOM_USUARI] = nom
        }
    }

    suspend fun guardarModeFosc(activat: Boolean) {
        context.dataStore.edit { preferences ->
            preferences[PreferencesKeys.MODE_FOSC] = activat
        }
    }
}
```

### 8.2. ViewModel

```kotlin
// SettingsViewModel.kt
import android.app.Application
import androidx.lifecycle.AndroidViewModel
import androidx.lifecycle.viewModelScope
import kotlinx.coroutines.flow.SharingStarted
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.stateIn
import kotlinx.coroutines.launch

class SettingsViewModel(application: Application) : AndroidViewModel(application) {

    private val preferencesManager: PreferencesManager
    val nomUsuari: StateFlow<String>
    val modeFosc: StateFlow<Boolean>

    init {
        preferencesManager = PreferencesManager(application)

        nomUsuari = preferencesManager.nomUsuari.stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5000),
            initialValue = ""
        )

        modeFosc = preferencesManager.modeFosc.stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5000),
            initialValue = false
        )
    }

    fun guardarNom(nom: String) {
        viewModelScope.launch {
            preferencesManager.guardarNomUsuari(nom)
        }
    }

    fun canviarModeFosc(activat: Boolean) {
        viewModelScope.launch {
            preferencesManager.guardarModeFosc(activat)
        }
    }
}
```

### 8.3. Activity

```kotlin
// SettingsActivity.kt
import android.os.Bundle
import androidx.activity.viewModels
import androidx.appcompat.app.AppCompatActivity
import androidx.lifecycle.lifecycleScope
import kotlinx.coroutines.launch

class SettingsActivity : AppCompatActivity() {

    private val viewModel: SettingsViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_settings)

        // Observar els canvis del nom d'usuari
        lifecycleScope.launch {
            viewModel.nomUsuari.collect { nom ->
                // Actualitzar la UI amb el nom
                binding.etNom.setText(nom)
            }
        }

        // Observar el mode fosc
        lifecycleScope.launch {
            viewModel.modeFosc.collect { activat ->
                binding.switchModeFosc.isChecked = activat
            }
        }

        // Guardar el nom quan es prem el botó
        binding.btnGuardar.setOnClickListener {
            val nom = binding.etNom.text.toString()
            viewModel.guardarNom(nom)
        }

        // Canviar el mode fosc
        binding.switchModeFosc.setOnCheckedChangeListener { _, activat ->
            viewModel.canviarModeFosc(activat)
        }
    }
}
```

## 9. Esborrar dades

Per esborrar una clau específica:

```kotlin
suspend fun esborrarNomUsuari(context: Context) {
    context.dataStore.edit { preferences ->
        preferences.remove(PreferencesKeys.NOM_USUARI)
    }
}
```

Per esborrar totes les dades:

```kotlin
suspend fun esborrarTot(context: Context) {
    context.dataStore.edit { preferences ->
        preferences.clear()
    }
}
```
