# Classes i objectes en kotlin
## Classes

- Son finals per defecte


### Absència de New

No cal posar new

```kotlin
val persona = Persona("Joan")
```

### Constructors més simples

Kotlin incorpora el constructor en la capçalera:

```kotlin
class Persona(val nom: String, var edat: Int,x:Int)
```

Si no s'acompanya var o val, la variable només s'utilitza en el constructor
### A Kotlin no cal escriure getters/setters:

Es generen automàticament segons val (només lectura) o var (mutable).

### Constructor

Init executa el codi quan l'objecte es crea:

```kotlin
class Circle(i: Int) {	
   init {
        ... 
   }
}
```
És tècnicament el mateix que 

```kotlin
class Circle {
    constructor(i: Int) {
        ...
    }
}
```
### Getters i Setters
Es poden sobreescriure els getters i setters per defecte:

```kotlin
class Person(val firstName: String, val lastName:String) {
    val fullName:String
        get() {
            return "$firstName $lastName"
        }
}
```

### Herència

- Kotlin té herència de classe amb un únic pare (single-parent class inheritance)
- Cada classe té exactament una classe pare, anomenada superclasse
- Cada subclasse hereta tots els membres de la seva superclasse, inclosos els que la superclasse mateixa ha heretat

## Classes especials

### Data class

Classe pensada per contenir dades

Generen automàticament:

✔ equals()
✔ hashCode()
✔ toString()
✔ copy()
✔ destructuring

```kotlin
data class Alumne(val nom: String, val edat: Int)
```
#### Pair i Triple son dataclasses predefinides:
Guarden un parell o tres valors

Accedim a les seves propietat amb .first, .second, .third
```kotlin
val bookAuthor = Pair("Harry Potter", "J.K. Rowling")
println(bookAuthor.first)
```

Pair té .to per ometre parèntesis

```kotlin
val bookAuth1 = "Harry Potter".to("J. K. Rowling")
val bookAuth2 = "Harry Potter" to "J. K. Rowling"
val map = mapOf(1 to "x", 2 to "y", 3 to "zz")
```
### Enum class

Tipus de dada predefinit per a un conjunt de valors
```kotlin
enum class Color(val r: Int, val g: Int, val b: Int) {
   RED(255, 0, 0), GREEN(0, 255, 0), BLUE(0, 0, 255)
}
println("" + Color.RED.r + " " + Color.RED.g + " " + Color.RED.b)
```

### Sealed class

Característiques:


- Sembla a una enum, però més avançada, amb dades i comportament.
- Una sealed class no es pot estendre fora del mateix fitxer on està definida.
- Això vol dir que totes les subclasses possibles estan controlades i limitades.


```kotlin
sealed class Operacio {
    class Suma(val a: Int, val b: Int) : Operacio()
    class Resta(val a: Int, val b: Int) : Operacio()
    class Quadrat(val a: Int) : Operacio()
}
```

Les sealed classes serveixen per:

- Restringir el polimorfisme
- Garantir tractament complet a when
- Substituir enums quan es necessita informació
- Fer el codi més segur i expressiu
- Evitar errors inesperats en herències

Són ideals quan sabem tots els casos possibles
i no volem que ningú en defineixi de nous.

El compilador ens avisaria si en el when: falta algun cas, o no els posem tots 
```kotlin
fun calcula(op: Operacio): Int {
    return when(op) {
        is Operacio.Suma -> op.a + op.b
        is Operacio.Resta -> op.a - op.b
        is Operacio.Quadrat -> op.a * op.a
    }
}
```
## Interfícies

- Proporcionen un contracte al qual les classes s'hi han d'adherir (han d'implementar els métodes)
- Poden contenir implementació per defecte (métodes abstractes i implmenentats)
- Poden heredar d'altres interfícies
- Poden tenir propietats
- Permeten múltiples herències
- Poden contenir membres estàtics en forma de companion objects

```kotlin
interface Shape {
    val name:String
    fun computeArea() : Double
}
class Circle(val radius:Double) : Shape {
    override fun computeArea() = Math.PI * radius * radius
}
val c = Circle(3.0)
println(c.computeArea())
```

## Objectes en Kotlin
La paraula **object**: declara singletons reals

```kotlin
object Configuracio {
    val versio = "1.0"
}
```

Equivalent al patró Singleton en Java, però automàtic:

- instància única
- inicialització garantida
- accessible directament:

```kotlin
println(Configuracio.versio)
```

### Object expressions (objectes anònims)
```kotlin
val listener = object : OnClickListener {
    override fun onClick() {
        println("Click!")
    }
}
```
-> substitueix new Interface() de Java

### Companion objects (equivalent a membres estàtics)

A Kotlin no existeixen les variables estàtiques de classe (static).
En lloc seu, s’utilitza companion object:

```kotlin
class Persona {
    companion object {
        val MAX_EDAT = 120
    }
}

println(Persona.MAX_EDAT)
```
Equivalent aproximadament a:

```kotlin
public class Persona {
    public static final int MAX_EDAT = 120;
}
```
