# Toolbar amb Fragments i botó Enrere

Quan una aplicació té una sola Activity i canvia de pantalla substituint Fragments, la fletxa enrere de la Toolbar no pot dependre del manifest, com passa entre Activities (vegeu [El botó "Up"](./toolbar.md#3-gestio-de-la-navegacio-a-lappbar)). Ha d'aparèixer quan l'usuari és en una pantalla secundària i desaparèixer quan torna a la principal.

Aquest document mostra com fer-ho de manera centralitzada a l'Activity, fent servir el back stack del `supportFragmentManager`. Els Fragments no han de saber res de la Toolbar: tota la lògica és a l'Activity.

Documentació oficial:

- [Fragment transactions i back stack](https://developer.android.com/guide/fragments/transactions)
- [FragmentManager](https://developer.android.com/guide/fragments/fragmentmanager)

Partim d'una Activity que ja té una Toolbar configurada com a AppBar. Vegeu [La Toolbar com a AppBar](./toolbar.md#1-la-toolbar-com-a-appbar-fonaments).

## 1. La idea

El `supportFragmentManager` guarda una pila (el back stack) amb les transaccions de Fragments que hem afegit amb `addToBackStack()`. El nombre d'elements de la pila ens diu on som:

- **0 elements**: som a la pantalla principal. La fletxa s'amaga.
- **1 o més elements**: som en una pantalla secundària. La fletxa es mostra.

Per saber quan canvia la pila, el `supportFragmentManager` ens ofereix un listener: `addOnBackStackChangedListener`. S'executa cada vegada que s'afegeix o es treu un element.

## 2. Codi de la MainActivity

```kotlin
import android.os.Bundle
import androidx.appcompat.app.AppCompatActivity
import androidx.appcompat.widget.Toolbar

class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // 1. Configurem la Toolbar com a AppBar de l'Activity
        val toolbar = findViewById<Toolbar>(R.id.toolbar)
        setSupportActionBar(toolbar)

        // 2. Carreguem el Fragment principal només la primera vegada
        if (savedInstanceState == null) {
            supportFragmentManager.beginTransaction()
                .replace(R.id.fragment_container, HomeFragment())
                .commit()
        }

        // 3. Cada vegada que canvia el back stack, actualitzem la fletxa
        supportFragmentManager.addOnBackStackChangedListener {
            actualitzarFletxa()
        }

        // 4. També en obrir l'Activity (p. ex. després de girar la pantalla)
        actualitzarFletxa()
    }

    private fun actualitzarFletxa() {
        // Si hi ha elements al back stack, no som a la pantalla principal
        val esPantallaSecundaria = supportFragmentManager.backStackEntryCount > 0
        supportActionBar?.setDisplayHomeAsUpEnabled(esPantallaSecundaria)
    }

    // 5. Es crida quan l'usuari prem la fletxa de la Toolbar
    override fun onSupportNavigateUp(): Boolean {
        // Treu l'últim Fragment del back stack i torna a l'anterior.
        // Si el back stack és buit, executa l'acció per defecte del botó Enrere.
        if (!supportFragmentManager.popBackStackImmediate()) {
            onBackPressedDispatcher.onBackPressed()
        }
        return true
    }
}
```

Punts a destacar:

- `onSupportNavigateUp()` és un mètode d'`AppCompatActivity` que es crida quan l'usuari prem la fletxa. És una alternativa a capturar `android.R.id.home` a `onOptionsItemSelected()`, i és més directe perquè només serveix per a això.
- `popBackStackImmediate()` treu l'últim element del back stack i retorna `true`. Si la pila és buida, no fa res i retorna `false`.
- El listener no s'executa en crear l'Activity, només quan la pila canvia. Si l'usuari gira la pantalla mentre és en una pantalla secundària, l'Activity es torna a crear amb la pila plena, i sense la crida del pas 4 la fletxa no apareixeria.
- La condició `savedInstanceState == null` evita afegir un altre `HomeFragment` quan l'Activity es torna a crear. Android ja restaura els Fragments sol.

## 3. Afegir els Fragments al back stack

Perquè el listener funcioni, els Fragments secundaris s'han d'afegir al back stack amb `addToBackStack(null)`. El principal, en canvi, no.

### Fragment principal

No l'afegim al back stack. Si ho féssim, en prémer Enrere des de la pantalla principal es trauria el `HomeFragment` i quedaria la pantalla en blanc.

```kotlin
supportFragmentManager.beginTransaction()
    .replace(R.id.fragment_container, HomeFragment())
    .commit()
```

### Fragments secundaris

Els afegim al back stack. Així el listener sap que ha de mostrar la fletxa, i el botó Enrere del sistema també torna a la pantalla anterior.

```kotlin
supportFragmentManager.beginTransaction()
    .replace(R.id.fragment_container, DetallFragment())
    .addToBackStack(null) // Afegeix la transacció al back stack
    .commit()
```

El paràmetre de `addToBackStack` és un nom opcional per a la transacció. Amb `null` n'hi ha prou si no l'hem de buscar després.

!!! info "El botó Enrere del sistema"
    No cal escriure res perquè el botó Enrere del sistema funcioni: amb `addToBackStack()`, Android ja treu l'últim Fragment de la pila quan l'usuari el prem. El listener s'executa igualment i la fletxa s'amaga quan toca. El codi d'`onSupportNavigateUp()` només serveix perquè la fletxa de la Toolbar faci el mateix.

## 4. Funcionament pas a pas

1. **Inici de l'app:** es carrega el `HomeFragment` sense `addToBackStack()`. El back stack té 0 elements i la fletxa no es mostra.
2. **Navegació:** es carrega el `DetallFragment` amb `addToBackStack(null)`. El back stack passa a tenir 1 element, s'executa el listener i la fletxa es mostra.
3. **Clic a la fletxa:** es crida `onSupportNavigateUp()`, que executa `popBackStackImmediate()`. Es treu el `DetallFragment` i es torna al `HomeFragment`. El back stack torna a 0 elements, s'executa el listener i la fletxa s'amaga.
