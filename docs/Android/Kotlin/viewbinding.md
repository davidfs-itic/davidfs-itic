# Viewbinding

https://developer.android.com/topic/libraries/view-binding

El viewbinding és una característica (feature) que ens permet accedir als objectes que hi ha en una vista, directament sense haver de buscar-lo amb el findviewbyid


S'activa d'aquesta manera, en el build.grade de nivell de mòdul

```Kotlin
android {
    ...
    buildFeatures {
        viewBinding = true
    }
}
```

El que farà aquesta característica, és generar, per cada layout, una classe amb el nom del layout+binding, per exemple  per al layout activity_main es farà la classe ActivityMainBinding 
Aquesta classe, contindrà tantes properties com elements visuals hi hagi en el layout, per exemple botons, textboxes, etc.

També té un métode getRoot() (o en kotlin, directament root) que accedeix al layout principal que conté la resta de la vista (linear o constraint layout)

Per utilitzar-la, cal
- Crear una variable d'aquest tipus (amb lateinit)
- Crear la instància de la classe amb el métode static inflate() que crearà l'objecte amb les properties necessàries lligades als objectes que va creant del layout.
- Utilitzar el setContentView() per a que l'activity es renderitzi a partir dels objectes creats.

Exemple

```Kotlin
private lateinit var binding: ActivityMainBinding
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    binding = ActivityMainBinding.inflate(layoutInflater)
    setContentView(binding.root)
}
```

A partir d'aquí, es pot accedir als elements de la UI directament sense el findviewbyId

```Kotlin
binding.mainTxt.setText("Login")

binding.btnLogin.setOnClickListener({
    doLogin()
})
```
