# Disseny de Proves (TDD)

Abans de programar, cal **planificar les proves**. En aquest document veurem com aplicar el procés TDD complet: des de l'anàlisi de requisits fins a la implementació.

## 1. Analitzar requisits

El primer pas és entendre **què ha de fer** la funcionalitat. A partir de la descripció, extraiem les **regles de negoci**.

!!! note "Exemple: Calculadora"
    Volem crear una classe `Calculadora` que faci les quatre operacions bàsiques: suma, resta, multiplicació i divisió.

    **Regles de negoci:**

    - La suma de dos nombres retorna el resultat correcte.
    - La resta de dos nombres retorna el resultat correcte.
    - La multiplicació de dos nombres retorna el resultat correcte.
    - La divisió de dos nombres retorna el resultat correcte.
    - La divisió per zero ha de llançar una excepció.
    - Les operacions han de funcionar amb nombres decimals.
    - Les operacions han de funcionar amb nombres negatius.

## 2. Definir casos de prova

Un cop tenim les regles, les convertim en una **taula de casos de prova**. Per cada cas definim: l'entrada, el resultat esperat i el tipus de test.

| # | Cas de prova | Entrada | Resultat esperat | Tipus |
|---|---|---|---|---|
| 1 | Suma de dos positius | `suma(2, 3)` | `5.0` | Unitari |
| 2 | Suma amb zero | `suma(5, 0)` | `5.0` | Unitari |
| 3 | Suma de negatius | `suma(-2, -3)` | `-5.0` | Unitari |
| 4 | Resta bàsica | `resta(10, 3)` | `7.0` | Unitari |
| 5 | Resta amb resultat negatiu | `resta(3, 10)` | `-7.0` | Unitari |
| 6 | Multiplicació bàsica | `multiplica(4, 3)` | `12.0` | Unitari |
| 7 | Multiplicació per zero | `multiplica(5, 0)` | `0.0` | Unitari |
| 8 | Divisió bàsica | `divideix(10, 2)` | `5.0` | Unitari |
| 9 | Divisió amb decimals | `divideix(7, 2)` | `3.5` | Unitari |
| 10 | Divisió per zero | `divideix(5, 0)` | Excepció | Unitari |

!!! tip "Consells per triar casos de prova"
    - **Camí normal**: el cas normal, que hauria de funcionar sempre.
    - **Valors límit**: zero, nombres negatius, valors molt grans.
    - **Casos d'error**: entrades invàlides que han de produir errors controlats.

## 3. Escriure els tests (Red)

Ara escrivim els tests **abans** d'implementar la classe `Calculadora`. En aquest moment, el codi **no compilarà** perquè la classe encara no existeix. Això és normal al TDD — és la fase **Red**.

Creem el fitxer `CalculadoraTest.kt` a la carpeta `app/src/test/java/com/example/app/`:

```kotlin
import org.junit.Assert.*
import org.junit.Before
import org.junit.Test

class CalculadoraTest {

    private lateinit var calc: Calculadora

    @Before
    fun setUp() {
        calc = Calculadora()
    }

    // --- SUMA ---

    @Test
    fun suma_deDosPositius_retornaResultatCorrecte() {
        val resultat = calc.suma(2.0, 3.0)
        assertEquals(5.0, resultat, 0.001)
    }

    @Test
    fun suma_ambZero_retornaElMateixNombre() {
        val resultat = calc.suma(5.0, 0.0)
        assertEquals(5.0, resultat, 0.001)
    }

    @Test
    fun suma_deDosNegatius_retornaNegatiu() {
        val resultat = calc.suma(-2.0, -3.0)
        assertEquals(-5.0, resultat, 0.001)
    }

    // --- RESTA ---

    @Test
    fun resta_basica_retornaResultatCorrecte() {
        val resultat = calc.resta(10.0, 3.0)
        assertEquals(7.0, resultat, 0.001)
    }

    @Test
    fun resta_ambResultatNegatiu_retornaNegatiu() {
        val resultat = calc.resta(3.0, 10.0)
        assertEquals(-7.0, resultat, 0.001)
    }

    // --- MULTIPLICACIÓ ---

    @Test
    fun multiplica_basica_retornaResultatCorrecte() {
        val resultat = calc.multiplica(4.0, 3.0)
        assertEquals(12.0, resultat, 0.001)
    }

    @Test
    fun multiplica_perZero_retornaZero() {
        val resultat = calc.multiplica(5.0, 0.0)
        assertEquals(0.0, resultat, 0.001)
    }

    // --- DIVISIÓ ---

    @Test
    fun divideix_basica_retornaResultatCorrecte() {
        val resultat = calc.divideix(10.0, 2.0)
        assertEquals(5.0, resultat, 0.001)
    }

    @Test
    fun divideix_ambDecimals_retornaResultatCorrecte() {
        val resultat = calc.divideix(7.0, 2.0)
        assertEquals(3.5, resultat, 0.001)
    }

    @Test(expected = IllegalArgumentException::class)
    fun divideix_perZero_llancaExcepcio() {
        calc.divideix(5.0, 0.0)
    }
}
```

!!! warning "En aquest punt els tests NO compilen"
    Això és correcte! La classe `Calculadora` encara no existeix. Si intentem executar els tests, obtindrem errors de compilació. Estem a la fase **Red** del TDD.

## 4. Implementar el codi mínim (Green)

Ara creem la classe `Calculadora` amb el codi **mínim** perquè tots els tests passin. No afegim res que no sigui necessari.

Creem el fitxer `Calculadora.kt` a la carpeta `app/src/main/java/com/example/app/`:

```kotlin
class Calculadora {

    fun suma(a: Double, b: Double): Double {
        return a + b
    }

    fun resta(a: Double, b: Double): Double {
        return a - b
    }

    fun multiplica(a: Double, b: Double): Double {
        return a * b
    }

    fun divideix(a: Double, b: Double): Double {
        if (b == 0.0) {
            throw IllegalArgumentException("No es pot dividir per zero")
        }
        return a / b
    }
}
```

Ara executem els tests i haurien de **passar tots** (fase **Green**).

## 5. Refactoritzar (Refactor)

En aquest cas la implementació ja és neta, però en casos més complexos podríem:

- Eliminar codi duplicat.
- Millorar noms de variables o funcions.
- Simplificar lògica complexa.

!!! warning "Regla d'or del Refactor"
    Després de cada canvi, **executa els tests**. Si algun test falla, desfés el canvi i torna a intentar-ho. Els tests han de seguir verds.

## 6. Convencions per nomenar tests

Una bona convenció per nomenar els tests és:

```
nomFuncio_condicio_resultatEsperat
```

Exemples:

- `suma_deDosPositius_retornaResultatCorrecte`
- `divideix_perZero_llancaExcepcio`
- `multiplica_perZero_retornaZero`

Això fa que quan un test falla, el nom ja t'explica **què** s'estava provant i **què** hauria de passar.

## 7. Consells per escriure bons casos de prova

### Valors límit

Prova sempre amb valors als extrems del rang esperat:

- Zero
- Nombres negatius
- Cadenes buides
- Llistes buides
- Valors molt grans o molt petits

### Casos d'error

Comprova que el codi gestiona correctament les situacions d'error:

- Entrades nul·les (`null`)
- Divisions per zero
- Índexs fora de rang
- Formats incorrectes

### Camí normal

No oblidis provar el cas **normal** — el que hauria de funcionar sempre. De vegades ens centrem tant en els casos límit que oblidem provar el cas bàsic.

### Un test, una cosa

Cada test hauria de comprovar **una sola cosa**. Si un test falla, has de saber immediatament què ha anat malament. Evita tests que comproven múltiples comportaments alhora.

!!! tip "Resum del procés TDD"
    1. **Analitza** els requisits i extreu regles de negoci.
    2. **Defineix** els casos de prova en una taula.
    3. **Escriu** els tests (Red — no compilen).
    4. **Implementa** el codi mínim (Green — tots passen).
    5. **Refactoritza** mantenint els tests verds (Refactor).
    6. **Repeteix** per a la següent funcionalitat.
