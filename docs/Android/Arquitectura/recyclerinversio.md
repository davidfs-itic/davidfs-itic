
# Exemple d'inversió de dependències amb el Recycler view senzill 

## Problema de callbacks:
Si examinem l'exemple de recyclerview tal i com està, (passant funcions de callback), podem trobar els problemes següents:


#### 1. Memory Leaks​
 - La lambda fà referència a l'activity( el this en el Toast)
 - Si l'Activity es destrueix (rotació, backstack), però l'Adapter/ViewHolder segueix viu → leak.
 - L'Activity no es pot recollir per garbage collector.

#### 2. Acoblament fort
 - L'Activity s'ha de conèixer què fa l'Adapter (passar callbacks específics).
 - Difícil reutilitzar l'Adapter en un altre context.

#### 3. Violació Single Responsibility
 - L'Activity fa 3 coses:
 - Presenta UI
 - Gestiona RecyclerView
 - Gestiona negocis (què fer al clic). Aixó últim no ho hauria de fer. No és la serva responsabilitat.


## Sol·lució amb inversió de dependències (Dependency Inversion Principle - DIP)

### 1. Interfície clara i tipada
```kotlin
interface ItemClickListener {
    fun onItemClick(item: MyItem)
}
```
### 2. Adapter rep interfície (NO lambda)
```kotlin
class MyAdapter(
    private val items: List<MyItem>,
    private val listener: ItemClickListener  // ← Millor!
) : RecyclerView.Adapter<MyViewHolder>() {
    // ... mateix codi
}
```

### 3. Activity només implementa interfície
```kotlin
class MainActivity : AppCompatActivity(), ItemClickListener {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        // ...
        adapter = MyAdapter(items, this)  // ← Passa Activity com a listener
        recyclerView.adapter = adapter
    }

    // Lògica de negoci AQUÍ, separada de UI
    override fun onItemClick(item: MyItem) {
        Toast.makeText(this, "Clicat ${item.title}", Toast.LENGTH_SHORT).show()
        // O millor: cridar ViewModel
        // viewModel.onItemSelected(item)
    }
}
```
## Què diu el principi DIP?​
```text
"Les mòduls d'alt nivell NO han de dependre de mòduls de baix nivell.
AMBDOs han de dependre d'abstraccions (interfícies)."
```
### En el nostre cas:

```text
- Incorrecte: Activity (alt nivell) ← DEPEN de MyAdapter (baix nivell)
- Correcte: Activity (alt nivell) → ItemClickListener ← MyAdapter (baix nivell)
```
Beneficis pràctics:


1. L'Activity es pot canviar (Fragment, Compose Screen) sense tocar Adapter
2. L'Adapter es pot reutilitzar en qualsevol Activity/Fragment
3. No té memory leaks (no captura 'this' directament dins lambda)
4. Testejable: mockejar ItemClickListener
5. Arquitectura limpia i professional

