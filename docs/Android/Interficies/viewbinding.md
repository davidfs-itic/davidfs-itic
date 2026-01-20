# Viewbinding

[https://developer.android.com/topic/libraries/view-binding](https://developer.android.com/topic/libraries/view-binding)

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

Per utilitzar-la, cal:


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

# Viewbinding en Fragments

En aquest cas cal crear la variable amb possibilitat de nuls, i que des de fora el fragment sigui readonly. Aixó s'aconsegueix fent la variable nula, i creant un métode get:

```kotlin
// 1. Una variable privada i nul·lable per guardar la referència real
    private var _binding: ResultProfileBinding? = null
    
    // 2. Una propietat de només lectura per no haver d'usar "!!" a tot arreu
    private val binding get() = _binding!!
```

No es declara lateinit com en el cas de les activities, a causa del cicle de vida diferent dels fragments.

La vista d'un Fragment pot ser destruïda mentre el Fragment encara és viu.

Això passa, per exemple, quan un usuari navega d'un Fragment A a un Fragment B. El Fragment A es queda a la backstack (està "viu"), però la seva interfície gràfica es destrueix per estalviar recursos.

Si guardem una referència al binding i no la destruïm, realment la vista no es podrà destruir perque encara hi haurà una referència a la vista.

Aixó fà que haguem de destruïr explicitament la variable en el onDestroy.

Exemple de codi complert de viewbinding amb fragments:

```kotlin
class ProfileFragment : Fragment() {

    private var _binding: ResultProfileBinding? = null
    
    private val binding get() = _binding!!

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View {
        _binding = ResultProfileBinding.inflate(inflater, container, false)
        return binding.root
    }

    override fun onDestroyView() {
        super.onDestroyView()
        // 3. IMPORTANT: Netegem la referència per evitar memory leaks
        _binding = null
    }
}
```
