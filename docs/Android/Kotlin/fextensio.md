## 1. Què són les Funcions d'Extensió?
Una funció d'extensió és una funció que es declara fora d'una classe, però que es pot cridar com si fos un mètode d'aquesta classe.

Sintaxi: Es defineix prefixant el nom de la funció amb el tipus que s'estén (el tipus receptor), seguit d'un punt (.) i el nom de la funció.

Accés: Dins de la funció d'extensió, pots accedir a les propietats i mètodes públics del tipus estès mitjançant la paraula clau this, que es refereix a la instància del receptor.

L'ús principal és fer el codi més llegible i concís, evitant la necessitat de classes utilitàries plenes de mètodes estàtics. Són molt comunes a la llibreria estàndard de Kotlin (per exemple, funcions com .map(), .filter(), o .joinToString() en col·leccions són funcions d'extensió).

## 2. Exemple Pràctic funció d'extensió
Imagina que sovint necessites una funció per capitalitzar la primera lletra d'una cadena (String). En lloc de crear una classe utilitària, pots estendre directament la classe String.

### Codi de la Funció d'Extensió
Definim una funció d'extensió anomenada capitalizeFirstLetter per al tipus String:

```Kotlin
// Extensions.kt
fun String.capitalizeFirstLetter(): String {
    // Si la cadena està buida o és null (tot i que String no pot ser null), 
    // retornem la cadena tal com està per seguretat.
    if (this.isEmpty()) return this

    // La paraula clau 'this' fa referència a la instància de String 
    // sobre la qual s'està cridant la funció (el receptor).
    return this.substring(0, 1).uppercase() + this.substring(1)
}
```

### Com S'Utiliza
Ara, pots cridar aquesta funció directament sobre qualsevol instància de String, com si fos un mètode propi de la classe:

```Kotlin

fun main() {
    val textNormal = "hola món"
    val textCapitalitzat = textNormal.capitalizeFirstLetter()

    println("Original: $textNormal")
    println("Capitalitzat: $textCapitalitzat")
    // Sortida: Original: hola món
    // Sortida: Capitalitzat: Hola món

    val altreText = "kotlin"
    println(altreText.capitalizeFirstLetter()) // Sortida: Kotlin
}
```

D'aquesta manera qualsevol variable del tipus de la classe extesa, pot utilitzar la funció en qualsevol part del codi.

## 3. Què són les Propietats d'Extensió?

Una propietat d'extensió és essencialment un camp que pots llegir o modificar directament sobre la instància d'una classe, tot i que aquesta propietat no forma part de la definició original de la classe.

- No emmagatzemen estat: És important entendre que les propietats d'extensió no poden inicialitzar-se amb un backing field (és a dir, no poden emmagatzemar dades). 
- Han de ser propietats calculades que només tenen un getter i, opcionalment, un setter.
- Sintaxi: S'assembla molt a la definició d'una propietat normal, però prefixant-la amb el nom de la classe que s'estén.

## 4. Exemple Pràctic: Propietat Calculada

Imagina que tens una classe String i vols una manera ràpida de comprovar si una cadena és una adreça de correu electrònic molt simple sense haver d'escriure isEmail() cada cop. 
Pots crear una propietat d'extensió anomenada isSimpleEmail.

### Codi de la Propietat d'Extensió

Definim la propietat d'extensió isSimpleEmail per al tipus String. És de tipus Boolean i només té un getter (és de només lectura):
```Kotlin
// Extensions.kt
// Definim una propietat de només lectura (val) amb un getter personalitzat
val String.isSimpleEmail: Boolean
    get() {
        // En una implementació real, aquesta lògica seria molt més complexa (p. ex., amb expressions regulars)
        // Però per a l'exemple, simplement mirem si conté un '@' i un '.'
        return this.contains('@') && this.contains('.')
    }
```
### Com S'Utilitza

Ara, pots accedir a aquesta propietat directament sobre qualsevol String com si fos un camp definit a la classe:Kotlinfun main() {
    val emailCorrecte = "usuari@exemple.com"
    val textNormal = "Això no és un correu"
    
    // Accedim-hi com una propietat, sense parèntesis!
    println("'$emailCorrecte' és un correu: ${emailCorrecte.isSimpleEmail}") 
    // Sortida: 'usuari@exemple.com' és un correu: true
    
    println("'$textNormal' és un correu: ${textNormal.isSimpleEmail}")
    // Sortida: 'Això no és un correu' és un correu: false
}
## 5. Diferències entre funció d'extensió i 

Observeu la diferència amb la funció d'extensió:

- Funció d'Extensió: text.capitalizeFirstLetter() $\rightarrow$ Necessita parèntesis.
- Propietat d'Extensió: text.isSimpleEmail $\rightarrow$ No necessita parèntesis, fent que el codi sigui molt més clar i semblant a la lectura d'un atribut.