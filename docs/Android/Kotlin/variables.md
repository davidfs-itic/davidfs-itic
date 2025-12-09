 
 ## Variebles en Kotlin

Existeixen dues formes principals de declarar variables

Kotlin diferencia clarament entre variables que poden canviar el seu contingut i variables que no.

### Declaració

#### val – Variables inmutables

- Representen valors constants o no modificables.
- Un cop assignades, no es poden canviar.
- Equivaldria a una constant o a una variable final en altres llenguatges.
- Són recomanades per defecte perquè fan el codi més segur i previsible.

#### var – Variables mutables

- Es poden modificar després de declarar-les.
- S’utilitzen quan el valor realment ha de canviar amb el temps.
- L’ús de var és útil, però no abusiu; Kotlin fomenta l’immutabilitat sempre que sigui possible.

### Tipus explícits o implícits

**Kotlin és un llenguatge fortament tipat.**

Kotlin permet indicar el tipus de la variable, però no sempre és necessari.

Gràcies a la **inferència de tipus**, el compilador pot deduir quin tipus és segons el valor assignat.

Tot i això, els tipus existeixen i són importants: 



### Condicionals (if com expressió)

les sentencies if, when poden retornar valors
```kotlin
val edat = 20
val missatge = if (edat >= 18) "Major d'edat" else "Menor d'edat"

println(missatge)
```

La variable missatge obté directament un resultat. 

No cal declarar primer la variable i després assignar un valor.

