## Funcions en kotlin

A part de les característiques que poden tenir les funcions, com el altres llenguatges com per exemple Java, (noms, paràmetres, valors per defecte, etc), Kotlin té les seves particularitats:


### Funcions compactes (single-expression)

Serveixen per simplificar:

Forma completa:
```kotlin
fun double(x: Int): Int {
    return x * 2
}
```

Forma compacta:

```kotlin
fun double(x: Int): Int = x * 2
```
### Lambdes i funcions d’ordre superior

Kotlin tracta les funcions com si fossin de "primera classe", o "first class":
Aixó vol dir que:

- es poden guardar en variables
- passar com a paràmetres
- retornar-les des d'altres funcions

#### Lambda

Una funció sense nom és una funció lambda, en aquest cas guardada com a variable:

```kotlin
val waterFilter : (Int) -> Int = { level: Int -> level / 2 }
println(waterFilter(20))  // 10
```
En aquest cas la definició de la variables segueix les normes:

```kotlin
val variable:tipus=valor
```
Però en comptes d'un tipus simple o una classe, el tipus i el valor és una funció.
En el tipus, una firma d'una funció, i en el valor, la implementació de la funció.

Si no especifiquem el paràmetre en una funció lambda i només té un paràmetre, aquest per defecte serà "it", i podem ometre el paràmetre i -> :

```kotlin
val waterFilter : (Int) -> Int = { it / 2 }
```

#### Funció d’ordre superior

Les funcions d'ordre superior són aquelles que reben una funció com a argument:

```kotlin
fun encodeMsg(msg: String, encode: (String) -> String): String {
    return encode(msg)
}

val upper = { s: String -> s.uppercase() }
println(encodeMsg("abc", upper))
```

#### Passar referència de funció

Amb :: podem passar la referència a una funció.

```kotlin
fun enc2(input:String): String = input.reversed()
encodeMessage("abc", ::enc2)
```

#### Últim paràmetre d'una funció

En Kotlin és preferible que si hi ha algun paràmetre que sigui una funció, aquest sigui l'ultim paràmetre:

```kotlin
encodeMessage("acronym", { input -> input.toUpperCase() })
```

Si la funció només té un paràmetre, podem ometre:

**Els parèntesi:**

```kotlin
val ints = listOf(1, 2, 3)
ints.filter { n: Int -> n > 0 }
```

**El nom del paràmetre i la -> si només n'hi ha un (serà it):**
```kotlin
ints.filter { it > 0 }
```
