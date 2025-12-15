# Realitzar una splash screen

Android proporciona una API de Splash Screen que facilita la implementació i assegura que la pantalla de presentació es mostri correctament a partir d'Android 12, tot i que funciona amb una biblioteca de retrocompatibilitat per a versions anteriors.

## 1. Dependències:
Afegir la Dependència: Afegir la biblioteca androidx.core:core-splashscreen al teu fitxer build.gradle (mòdul: app).


```kotlin
dependencies {
    ...
    implementation 'androidx.core:core-splashscreen:1.2.0' 
    ...
```

Android studio suggerirà la versió i si utilitzem el format libs.versions.toml suggerirà fer el canvi.

## 2. Tema

Definir el Tema del Splash Screen (en themes.xml): Crear un estil que hereti de Theme.SplashScreen i especificar l'icona animada i el tema de la següent activitat.

```XML
<style name="Theme.App.SplashScreen" parent="Theme.SplashScreen">
<item name="windowSplashScreenBackground">@color/dark_grey</item>
    <item name="windowSplashScreenAnimatedIcon">@drawable/logo_animat</item>
    <item name="postSplashScreenTheme">@style/Base.Theme.LaMeuaApp</item>
    <item name="windowSplashScreenAnimationDuration">1000</item>
</style>
```

Per tenir una icona animada (windowSplashScreenAnimatedIcon), pots utilitzar un Animated Vector Drawable (AVD), que és un XML que defineix com un Vector Drawable canvia al llarg del temps.

## 3. Manifest

Configurar el Manifest: Establir l'estil creat com el tema de la teva MainActivity (o l'activitat inicial) a AndroidManifest.xml.

```XML
<activity
    android:name=".MainActivity"
    android:theme="@style/Theme.App.SplashScreen"
    ...>
    </activity>
```

## 4. Controlar la Durada (en MainActivity.kt): 

A la teva activitat principal, has de cridar installSplashScreen() i pots utilitzar setKeepOnScreenCondition per mantenir la pantalla de presentació visible fins que l'animació hagi acabat o les dades s'hagin carregat.

```Kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        // Cridar abans de super.onCreate() i setContentView()
        val splashScreen = installSplashScreen()

        // Opcional: Per mantenir-lo a la pantalla durant més temps (p. ex. carregar dades)
        splashScreen.setKeepOnScreenCondition { 
            // Retorna 'true' per mantenir el splash screen, 'false' per amagar-lo
            // Aquí hi aniria la teva lògica de càrrega
            false 
        }

        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        // ...
    }
}
```