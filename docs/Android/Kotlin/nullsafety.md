## Variables i seguretat amb valors nuls

Un dels trets més distintius és com gestiona els valors nuls:

- Una variable normal **no pot tenir null**.
```kotlin
val nom: String = "David"
```

Si volem permetre valors nuls, aquell mateix tipus es converteix en un tipus diferent.

- Int? és diferent de Int

- String? és diferent de String

```kotlin
val nom: String? = null
```


En Kotlin, si una variable pot ser null, el compilador obliga el programador a gestionar el cas null abans d’utilitzar-la.
Aquest sistema evita molts errors freqüents en llenguatges més antics.

### Operadors per gestionar nulabilitat de variables

#### Operador d’accés segur ?.

Serveix per accedir a una propietat o funció només quan el valor no és null.

```kotlin
val nom: String? = "David"
println(nom?.length)
```

També el compilador sap si hem comprovat la nullabilitat:
```kotlin
val nom: String? = "David"
if (nom != null){
    println(nom.length)
}
```
En aquest cas, no cal utilitzar l'operador ?.

#### Operador Elvis ?:

Permet donar un valor alternatiu quan la variable és null.
```kotlin
val nom: String? = null
val resultat = nom ?: "Desconegut"
println(resultat)
```
#### Operador de no-null forçat !!

És com dir: "estic segur que no és null", (o capturar l'excepció si ho és).

```kotlin
val nom: String? = null
println(nom!!.length) // Error en execució!
```

- Si la variable és null → el programa falla.
- Només s’ha d’utilitzar si es té la certesa que no serà null.

#### Funció let combinada amb ?.

S’executa el bloc només si el valor no és null.
```kotlin
val nom: String? = "David"
nom?.let {
    println("El nom té ${it.length} lletres")
}
```