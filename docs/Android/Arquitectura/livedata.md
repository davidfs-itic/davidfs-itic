
# LiveData i ViewModel: 
## Conceptes fonamentals de LiveData
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

## Diferències entre dades normals i LiveDataDades normals (variables convencionals):
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
## El patró Observer aplicat a LiveData
LiveData implementa el patró de disseny Observer, on:

Subject (LiveData): Conté les dades i manté una llista d'observadors
Observer (UI component): S'subscriu per rebre notificacions dels canvis

Funcionament:

La UI s'subscriu al LiveData utilitzant observe()
Quan les dades canvien, LiveData notifica automàticament tots els observadors actius
Els observadors reben les noves dades i actualitzen la UI








Patró recomanat (ViewModel exposa LiveData, UI observa)
Comunicació entre components
Gestió d'estat de la UI (loading, success, error)
Testing de ViewModels amb LiveData




## Referències

Livedata (amb viewbinding)
https://cursokotlin.com/mvvm-en-android-con-kotlin-livedata-y-view-binding-android-architecture-components/

Video de Devexperto:
https://www.youtube.com/watch?v=LHBbs6QXvic

### Codelab
https://developer.android.com/codelabs/basic-android-kotlin-training-livedata?hl=es-419#0

