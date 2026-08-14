# Qüestionari previ: de Java a Kotlin

Abans d'endinsar-te en la resta d'apunts d'aquesta secció, respon aquestes preguntes obertes. No busquen que recitis la sintaxi de Kotlin, sinó que et facin pensar en com resols cada situació en Java i què canvia (o per què cal que canviï) en Kotlin. Si no saps respondre alguna pregunta amb seguretat, és senyal que cal repassar aquell apunt concret abans de continuar.

## 1. Variables i tipus

### 1.1 Inferència de tipus

En Java, sempre has d'escriure el tipus d'una variable (`int edat = 20;`, `String nom = "Anna";`). En Kotlin sovint no cal escriure'l (`val edat = 20`).

- Si Kotlin no obliga a escriure el tipus, vol dir que les variables no tenen tipus? Com decideix el compilador quin tipus té `edat`?
- Quin avantatge i quin risc hi veus, respecte de Java, a no escriure sempre el tipus?

Consulta [Variables i Constants](./variables.md) si tens dubtes.

### 1.2 Substitució de variables en strings

En Java, per construir un text amb el valor d'una variable normalment fas servir la concatenació (`"Hola " + nom + ", tens " + edat + " anys"`) o `String.format(...)`.

- Com creus que es podria simplificar aquesta construcció de text si el llenguatge permetés "incrustar" directament una variable dins del text, sense sortir de les cometes?
- Quines diferències esperaries entre incrustar una variable simple i incrustar una expressió sencera (per exemple, `edat + 1`)?

### 1.3 Mutabilitat: val vs var

Java no distingeix entre "variable que canviarà" i "variable que no canviarà" a nivell de paraula clau bàsica: només tens `final` com a modificador opcional.

- Per què creus que Kotlin ofereix dues paraules clau diferents (`val` i `var`) en lloc d'una de sola amb un modificador opcional com `final`?
- Si en Java gairebé mai fas servir `final` en variables locals, quin canvi d'hàbit et suposarà treballar amb `val` i `var`?

Consulta [Variables i Constants](./variables.md) si tens dubtes.

## 2. Null safety

En Java, qualsevol variable d'un tipus objecte (`String`, `List`, una classe pròpia...) pot valer `null` en qualsevol moment, i el compilador no t'avisa. Això provoca el clàssic `NullPointerException` en temps d'execució.

- Com resols normalment, en Java, el fet de no saber si una variable pot ser `null` abans d'utilitzar-la?
- Si Kotlin distingís entre "aquesta variable mai serà null" i "aquesta variable pot ser null" ja al tipus (per exemple, `String` davant de `String?`), com creus que això afectaria els errors que detectes en temps de compilació versus en temps d'execució?
- Quina diferència esperaries entre dir-li al compilador "estic segur que no és null" (assumint el risc tu) i dir-li "si és null, fes servir aquest altre valor"?

Consulta [Null Safety](./nullsafety.md) si tens dubtes.

## 3. Expressions vs sentències

En Java, `if`, `switch` i `try` són **sentències**: executen codi però no "retornen" cap valor que puguis assignar directament a una variable. Per assignar un valor calculat condicionalment, normalment declares la variable abans i la reassignes dins de cada branca, o fas servir l'operador ternari `?:` només per al cas `if`/`else` més senzill.

- Com escriuries en Java una assignació que depengui d'una condició `if`/`else if`/`else` amb tres o més casos, sense operador ternari?
- Si en Kotlin `if`, `when` i `try` poguessin "retornar" directament un valor (com fa el ternari de Java), què hauria de passar amb totes les branques perquè això funcioni de manera consistent?
- Java té el tipus `void` per a mètodes que no retornen res. Si a Kotlin *tot* és una expressió (fins i tot cridar una funció que no calcula res útil), quin tipus creus que hauria de tenir aquesta "no resposta", en lloc de no tenir-ne cap?

## 4. Funcions i lambdes

Java permet passar comportament com a paràmetre des de fa temps (interfícies funcionals, `Runnable`, `Comparator`, i des de Java 8 les lambdes amb `->`).

- Quan has fet servir una lambda o una referència a mètode (`::metode`) en Java, per exemple amb `Comparator` o `Stream`? Quina sintaxi feies servir?
- Si Kotlin tracta les funcions com a "ciutadans de primera classe" (es poden guardar en variables, passar com a paràmetre i retornar), quina diferència esperaries respecte de fer servir una interfície funcional explícita com a Java?
- Si una lambda només té un paràmetre, què guanyaries (i què perdries en claredat) si el llenguatge et deixés ometre'n el nom?

## 5. Col·leccions i programació funcional

Des de Java 8, `Stream` permet fer `.filter(...).map(...).collect(...)` sobre una col·lecció.

- Quina diferència hi ha, en Java, entre una `List` normal i una `List` embolicada amb `Collections.unmodifiableList(...)`? Amb quina freqüència feies servir aquesta segona opció?
- Si un llenguatge distingís, ja des del tipus declarat, entre una col·lecció que es pot modificar i una que no, quin efecte creus que tindria sobre els errors de "algú m'ha modificat la llista per sorpresa"?
- `Stream.filter().map()` de Java necessita convertir la col·lecció a stream i després tornar-la a col·lecció (`.collect(Collectors.toList())`). Si `filter` i `map` es poguessin cridar directament sobre la llista i retornessin directament una altra llista, què et simplificaria i què hauries de vigilar?

Consulta [Col·leccions](./colleccions.md) si tens dubtes.

## 6. Classes i objectes

### 6.1 Herència: final per defecte

En Java, qualsevol classe es pot heretar tret que l'hagis marcat explícitament com a `final`.

- Quantes vegades, en un projecte Java, has marcat una classe com a `final` de manera deliberada perquè ningú l'heretés per error?
- Si un llenguatge invertís aquesta norma per defecte (les classes no es poden heretar tret que ho autoritzis explícitament), quin tipus de bugs de "herència accidental" creus que evitaria?

### 6.2 Constructors i creació d'objectes

En Java, crear un objecte sempre passa per `new NomClasse(...)`, i si vols un getter/setter, l'has d'escriure tu (o generar-lo amb l'IDE).

- Quantes línies de codi "repetitiu" (getters, setters, constructor) sol tenir una classe Java senzilla amb 3 o 4 camps?
- Si el llenguatge generés automàticament getters i setters a partir de com declares la propietat (només lectura o lectura/escriptura), què hauries de decidir tu i què deixaries de escriure?

### 6.3 Dataclasses

En Java (abans dels `record`, o en projectes que encara no els fan servir), una classe pensada només per transportar dades sol necessitar `equals()`, `hashCode()`, `toString()` i, si vols una còpia amb un camp canviat, un mètode fet a mà.

- Si mai has fet servir `record` de Java (Java 16+), quina diferència hi trobes respecte d'una classe POJO normal amb Lombok o generada per l'IDE?
- Quins d'aquests mètodes generats automàticament (`equals`, `hashCode`, `toString`, còpia) et sembla que és més fàcil d'oblidar actualitzar quan afegeixes un camp nou a la classe, si els haguessis d'escriure a mà?

### 6.4 when amb enum i sealed classes

Java té `enum` des de fa temps, i `switch` es pot fer exhaustiu sobre un enum, encara que el compilador no sempre t'obliga a cobrir tots els casos (depèn de si hi ha un valor de retorn i de la versió de Java).

- Si afegeixes un nou valor a un `enum` de Java i tens diversos `switch` escampats pel projecte que en depenen, com t'assegures que no n'has oblidat cap?
- Un enum de Java pot tenir camps i mètodes, però totes les instàncies comparteixen la mateixa "forma" (els mateixos camps). Si necessitessis que cada cas d'un tipus tancat pogués tenir camps *diferents* (per exemple, una operació de suma amb dos operands i una operació d'arrel quadrada amb un de sol), com ho resoldries amb les eines de Java que coneixes (herència, interfícies, enum...)?

Consulta [Classes i Objectes](./classesobjectes.md) si tens dubtes.
