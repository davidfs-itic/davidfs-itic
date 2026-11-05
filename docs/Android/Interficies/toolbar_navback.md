# Gestió Centralitzada i Desacoblada del Botó d'Enrere a Android (Kotlin)

Aquesta guia mostra com mostrar o amagar el botó d'enrere (*Back button / Up navigation*) en un `Toolbar` utilitzant els mètodes natius de **`AppCompatActivity`** i **`supportFragmentManager`**. 

Amb aquest enfocament, els fragments queden **100% desacoblats**, ja que tota la lògica es gestiona de manera centralitzada a l'Activity.

---

## 1. Codi de la `MainActivity`

Copia aquest codi a la teva Activity principal. S'encarrega d'escoltar els canvis en la cua de fragments (BackStack) i de gestionar el clic físic a la fletxa.

```kotlin
import android.os.Bundle
import androidx.appcompat.app.AppCompatActivity
import androidx.appcompat.widget.Toolbar

class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // 1. Configurem la Toolbar com a ActionBar de l'Activity
        val toolbar = findViewById<Toolbar>(R.id.toolbar)
        setSupportActionBar(toolbar)

        // 2. Oïdor automàtic del cicle del BackStack
        supportFragmentManager.addOnBackStackChangedListener {
            // Si hi ha fragments a la cua (backStackEntryCount > 0), significa que NO estem al principal
            val teFragmentsSecundaris = supportFragmentManager.backStackEntryCount > 0
            
            supportActionBar?.apply {
                setDisplayHomeAsUpEnabled(teFragmentsSecundaris)
                setDisplayShowHomeEnabled(teFragmentsSecundaris)
            }
        }
    }

    // 3. Callback natiu d'AppCompatActivity per gestionar el clic a la fletxa
    override fun onSupportNavigateUp(): Boolean {
        // Intenta treure l'últim fragment secundari de la cua. 
        // Si la cua ja està buida, executa l'acció per defecte (tancar l'Activity).
        if (!supportFragmentManager.popBackStackImmediate()) {
            onBackPressedDispatcher.onBackPressed() 
        }
        return true
    }
}
```

---

## 2. Requisit indispensable en la navegació

Perquè l'oïdor de l'Activity pugui comptar correctament els fragments, és **obligatori** afegir els fragments secundaris a la cua de navegació mitjançant `.addToBackStack(null)`.

### ❌ Fragment Principal (Home / Base)
*No* l'has d'afegir al BackStack per evitar que quedi una pantalla en blanc en prémer enrere.
```kotlin
supportFragmentManager.beginTransaction()
    .replace(R.id.fragment_container, HomeFragment())
    .commit()
```

###  Fragments Secundaris (Detalls / Formularis)
*Sí* que has d'afegir `.addToBackStack(null)` perquè l'Activity sàpiga que ha d'activar el botó d'enrere.
```kotlin
supportFragmentManager.beginTransaction()
    .replace(R.id.fragment_container, DetallFragment())
    .addToBackStack(null) // CLAU: Activa l'addOnBackStackChangedListener
    .commit()
```

---

## 🔄 Funcionament del Flux

1. **Inici de l'App:** Es carrega el *HomeFragment* (Cua = 0) ➔ L'oïdor **amaga** la fletxa.
2. **Navegació:** Es carrega el *DetallFragment* amb `.addToBackStack()` (Cua = 1) ➔ L'oïdor **mostra** la fletxa.
3. **Clic a la fletxa:** Es llança `onSupportNavigateUp()` ➔ Executa `popBackStackImmediate()` ➔ Es destrueix el detall i es torna al *HomeFragment* (Cua = 0) ➔ L'oïdor **amaga** la fletxa automàticament.
