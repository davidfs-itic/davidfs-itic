# La classe Application a Android

La classe `Application` és la classe base que manté l'estat global de tota l'aplicació Android. Es crea abans que qualsevol Activity, Service o BroadcastReceiver, i es destrueix només quan el procés de l'aplicació finalitza.

Crear una classe hereva d'`Application` permet executar codi d'inicialització una sola vegada a l'inici de l'aplicació i compartir estat o configuracions entre tots els components.

Documentació oficial: [Application | Android Developers](https://developer.android.com/reference/android/app/Application)

## 1. Cicle de vida de la classe Application

La classe `Application` té un cicle de vida molt senzill:

- **`onCreate()`** — Es crida quan es crea l'aplicació, abans que qualsevol altre component. Es el punt d'entrada principal per a inicialitzacions globals.
- **`onTerminate()`** — Només es crida en entorns d'emulació. No es garanteix que s'executi en dispositius reals.
- **`onLowMemory()`** — Es crida quan el sistema té poca memòria.
- **`onConfigurationChanged()`** — Es crida quan la configuració del dispositiu canvia (rotació, idioma, etc.).

!!! warning
    El mètode `onCreate()` d'`Application` s'executa al fil principal (Main Thread). Evita fer operacions llargues aquí, ja que retardarà l'inici de l'aplicació.

## 2. Crear una classe hereva d'Application

### Pas 1: Crear la classe

```kotlin
class MyApp : Application() {

    override fun onCreate() {
        super.onCreate()
        // Inicialitzacions globals aquí
    }
}
```

### Pas 2: Registrar-la al AndroidManifest.xml

Cal indicar al sistema quina classe `Application` utilitzar mitjançant l'atribut `android:name` a l'etiqueta `<application>`:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.exemple.app">

    <application
        android:name=".MyApp"
        android:label="@string/app_name"
        android:theme="@style/AppTheme">

        <!-- Activities, Services, etc. -->

    </application>
</manifest>
```

!!! info
    Si no registres la classe al manifest, Android utilitzarà la classe `Application` per defecte i el teu codi no s'executarà.

## 3. Casos d'ús principals

### 3.1. Inicialització de llibreries globals

Moltes llibreries necessiten inicialitzar-se una sola vegada a l'inici de l'aplicació. `Application.onCreate()` és el lloc ideal per fer-ho.

```kotlin
class MyApp : Application() {

    override fun onCreate() {
        super.onCreate()

        // Inicialitzar Firebase
        FirebaseApp.initializeApp(this)

        // Inicialitzar una llibreria d'analytics
        Analytics.init(this, "API_KEY")
    }
}
```

### 3.2. Estat global compartit entre components

Es pot utilitzar la classe `Application` per emmagatzemar variables globals accessibles des de qualsevol Activity o Fragment.

```kotlin
class MyApp : Application() {

    // Instància global de Retrofit
    lateinit var retrofit: Retrofit
        private set

    // Estat global
    var isUserLoggedIn: Boolean = false

    override fun onCreate() {
        super.onCreate()

        retrofit = Retrofit.Builder()
            .baseUrl("https://api.exemple.com/")
            .addConverterFactory(GsonConverterFactory.create())
            .build()
    }
}
```

Per accedir-hi des d'una Activity o Fragment:

```kotlin
// Des d'una Activity
val app = application as MyApp
val retrofit = app.retrofit

// Des d'un Fragment
val app = requireActivity().application as MyApp
val isLoggedIn = app.isUserLoggedIn
```

### 3.3. Configuració de temes o idioma

Es pot aplicar una configuració global de tema o idioma abans que es creï cap Activity:

```kotlin
class MyApp : Application() {

    override fun onCreate() {
        super.onCreate()

        // Aplicar mode fosc segons preferències guardades
        val prefs = getSharedPreferences("settings", MODE_PRIVATE)
        val isDarkMode = prefs.getBoolean("dark_mode", false)

        AppCompatDelegate.setDefaultNightMode(
            if (isDarkMode) AppCompatDelegate.MODE_NIGHT_YES
            else AppCompatDelegate.MODE_NIGHT_NO
        )
    }
}
```

### 3.4. Injecció de dependències amb Hilt

Quan s'utilitza Hilt per a la injecció de dependències, cal anotar la classe `Application` amb `@HiltAndroidApp`:

```kotlin
@HiltAndroidApp
class MyApp : Application() {

    override fun onCreate() {
        super.onCreate()
        // Hilt s'inicialitza automàticament
    }
}
```

Aquesta anotació genera el codi necessari perquè Hilt pugui gestionar la injecció de dependències a tota l'aplicació.

### 3.5. Gestió del cicle de vida de processos

Es pot observar quan l'aplicació passa a primer pla o a segon pla utilitzant `ProcessLifecycleOwner`:

```kotlin
class MyApp : Application(), LifecycleObserver {

    override fun onCreate() {
        super.onCreate()
        ProcessLifecycleOwner.get().lifecycle.addObserver(this)
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_START)
    fun onAppForeground() {
        // L'aplicació ha passat a primer pla
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
    fun onAppBackground() {
        // L'aplicació ha passat a segon pla
    }
}
```

!!! info
    Cal afegir la dependència `androidx.lifecycle:lifecycle-process` al fitxer `build.gradle` per utilitzar `ProcessLifecycleOwner`.

### 3.6. Gestió d'errors globals

Es pot definir un gestor d'excepcions no capturades per registrar errors fatals abans que l'aplicació es tanqui:

```kotlin
class MyApp : Application() {

    override fun onCreate() {
        super.onCreate()

        Thread.setDefaultUncaughtExceptionHandler { thread, throwable ->
            // Registrar l'error (per exemple, enviar-lo a un servei de crashlytics)
            Log.e("MyApp", "Error no capturat al fil ${thread.name}", throwable)

            // Finalitzar el procés
            android.os.Process.killProcess(android.os.Process.myPid())
        }
    }
}
```

## 4. Exemple complet

Exemple d'una classe `Application` que inicialitza Retrofit, gestiona l'estat de sessió i configura el tema:

```kotlin
class MyApp : Application() {

    lateinit var retrofit: Retrofit
        private set

    lateinit var apiService: ApiService
        private set

    var isUserLoggedIn: Boolean = false
        private set

    override fun onCreate() {
        super.onCreate()

        // Configurar tema
        configurarTema()

        // Inicialitzar Retrofit
        inicialitzarRetrofit()

        // Comprovar sessió
        comprovarSessio()
    }

    private fun configurarTema() {
        val prefs = getSharedPreferences("settings", MODE_PRIVATE)
        val isDarkMode = prefs.getBoolean("dark_mode", false)
        AppCompatDelegate.setDefaultNightMode(
            if (isDarkMode) AppCompatDelegate.MODE_NIGHT_YES
            else AppCompatDelegate.MODE_NIGHT_NO
        )
    }

    private fun inicialitzarRetrofit() {
        val okHttpClient = OkHttpClient.Builder()
            .connectTimeout(30, TimeUnit.SECONDS)
            .readTimeout(30, TimeUnit.SECONDS)
            .build()

        retrofit = Retrofit.Builder()
            .baseUrl("https://api.exemple.com/")
            .client(okHttpClient)
            .addConverterFactory(GsonConverterFactory.create())
            .build()

        apiService = retrofit.create(ApiService::class.java)
    }

    private fun comprovarSessio() {
        val prefs = getSharedPreferences("auth", MODE_PRIVATE)
        isUserLoggedIn = prefs.contains("token")
    }

    fun tancarSessio() {
        isUserLoggedIn = false
        getSharedPreferences("auth", MODE_PRIVATE).edit().clear().apply()
    }
}
```

## 5. Bones practiques

- **No abusar d'Application com a "bossa global"**: Emmagatzemar massa estat a `Application` dificulta el testing i crea acoblaments forts entre components. Prefereix `ViewModel` per a estat de la UI i injecció de dependencies per a serveis compartits.

- **Preferir injecció de dependencies**: Eines com Hilt o Koin gestionen millor les dependencies globals que variables dins d'`Application`.

- **Mantenir `onCreate()` lleuger**: Inicialitzacions pesades (com accés a base de dades o xarxa) s'han de fer en segon pla amb coroutines o `WorkManager`.

- **No guardar referencies a Activities o Fragments**: L'objecte `Application` viu durant tota l'execució. Si guarda referencies a components amb cicle de vida curt, es produiran fugues de memoria.

- **Utilitzar `AndroidViewModel` per accedir a Application des d'un ViewModel**: En lloc d'accedir directament a la instancia d'`Application`, utilitza `AndroidViewModel` que rep `Application` al constructor de manera segura. Consulta la documentació sobre [Application ViewModel](./Arquitectura/applicationviewmodel.md).

!!! warning
    La classe `Application` no sobreviu a la mort del procés. Si el sistema operatiu mata el procés per alliberar memoria, tot l'estat emmagatzemat a `Application` es perdrà. Utilitza `SharedPreferences`, `Room` o `DataStore` per persistir dades importants.
