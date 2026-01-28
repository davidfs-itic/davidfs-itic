# Observació de LiveData des de la UI
## Mètode observe() i el LifecycleOwner
El mètode observe() és la forma principal d'observar LiveData des dels components de UI. Requereix dos paràmetres: un LifecycleOwner i un Observer.

Signatura del mètode:
```kotlin
fun observe(owner: LifecycleOwner, observer: Observer<T>)
```

Components:

- LifecycleOwner: Component amb cicle de vida (Activity, Fragment)
- Observer: Lambda o classe que rep les actualitzacions

A Activity:
```kotlin
class MainActivity : AppCompatActivity() {
    private val viewModel: DadesViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // 'this' és el LifecycleOwner (Activity)
        viewModel.dades.observe(this) { dades ->
            // Actualitzar UI amb les noves dades
            textView.text = dades
        }
    }
}
```
A Fragment (IMPORTANT):
```kotlin
class MeuFragment : Fragment() {
    private val viewModel: DadesViewModel by viewModels()
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        //CORRECTE: usar viewLifecycleOwner
        viewModel.dades.observe(viewLifecycleOwner) { dades ->
            textView.text = dades
        }
        
        //INCORRECTE: NO usar 'this' en Fragments
        // viewModel.dades.observe(this) { ... }  // Pot causar memory leaks!
    }
}
```
Per què viewLifecycleOwner en Fragments:

Els Fragments tenen DOS cicles de vida: el del Fragment i el de la View
La View pot destruir-se i recrear-se mentre el Fragment segueix viu.
Usar this pot causar que l'observer segueixi actiu quan la View ja no existeix
viewLifecycleOwner s'atura quan la View es destrueix, evitant fugues de memòria (memory leaks).
