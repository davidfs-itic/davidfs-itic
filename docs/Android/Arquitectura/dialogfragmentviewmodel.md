# Guia Simplificada: DialogFragment amb ViewModel

En aquesta guia veurem com un `DialogFragment` (un quadre de diàleg amb un formulari) afegeix una tasca a una llista que es mostra en un RecyclerView d'un altre Fragment.

La idea clau és que **el diàleg i el Fragment no es comuniquen directament**. Tots dos comparteixen el mateix ViewModel: el diàleg li envia les dades del formulari, i el Fragment observa els canvis que el ViewModel publica.

Documentació oficial:

- [DialogFragment](https://developer.android.com/guide/fragments/dialogs)
- [ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel)

Abans de començar, convé tenir clars aquests documents:

- [ViewModel](./viewmodel.md), especialment l'apartat [ViewModel compartit (activityViewModels())](./viewmodel.md#viewmodel-compartit-activityviewmodels).
- [LiveData](../Kotlin/livedata.md).
- [RecyclerView amb llista en memòria](../Interficies/recyclerview.md).
- [Dialogs i DialogFragment](../Interficies/dialegs.md).

## Estructura del Projecte

```
app/
├── data/
│   └── TasksList.kt
├── model/
│   └── Task.kt
├── viewmodel/
│   └── TasksViewModel.kt
├── ui/
│   ├── fragments/
│   │   └── HomeFragment.kt
│   └── dialogs/
│       ├── AfegirTascaDialog.kt
└── adapter/
    └── TasksAdapter.kt
```

Cada carpeta (paquet) agrupa les classes segons la seva responsabilitat:

- `model/` → les classes que representen les dades (`Task`).
- `data/` → on es guarden les dades (`TasksList`).
- `viewmodel/` → la lògica i la validació (`TasksViewModel`).
- `ui/` → les pantalles: Fragments i diàlegs.
- `adapter/` → l'adapter del RecyclerView.

## Flux de Treball

```

                ┌─────────────────────────────────────────┐
                │           TasksList (Object)            │
                │      MutableList<Task> (Singleton)      │
                └──────────────────┬──────────────────────┘
                                   ▲
                                   │ modifica
                                   │
┌──────────────────────────────────┴─────────────────────────────────────────────────┐
│                          TasksViewModel                                            │
│  • Valida les dades                                                                │
│  • Modifica TasksList                                                              │
│  • Emet LiveData<TascaAction> (AFEGIDA/MODIFICADA/ELIMINADA)                       │
│  • Emet LiveData<AfegirTascaState> (Idle/Success/Error)                            │
└───┬─────────────────────────────────────────┬─────────────────┬────────────────────┘
    │                                         ▲                 │
    │  observa tascaActionEvent               │ AfegirTasca VM  │ observa afegirState
    │                                         │                 │
    ▼                                         │                 ▼
┌──────────────────────────────────┐         ┌──────────────────────────────────────┐
│        HomeFragment              │         │   AfegirTascaDialog                  │
│                                  │         │                                      │
│  • Observa tascaActionEvent      │         │  • Observa afegirState               │
│  • Actualitza RecyclerView       │────────▶│  • Mostra Toast d'error              │
│  • Mostra Toast de confirmació   │         │  • Crida viewModel                   │
│  • Crida dialogs                 │         │    amb dades del formulari           │
│                                  │         │  • Es tanca si l'estat és Success    │
└──────────────────────────────────┘         └──────────────────────────────────────┘


```

### Què passa quan l'usuari afegeix una tasca

1. L'usuari prem el botó **Afegir** del `HomeFragment`, que obre l'`AfegirTascaDialog`.
2. L'usuari omple el formulari i prem **Guardar**. El diàleg llegeix els camps i crida `viewModel.afegirTasca(...)`.
3. El ViewModel valida les dades:
    - Si hi ha un error, posa `afegirState` a `Error`. El diàleg ho observa, mostra un Toast amb el missatge i **es queda obert** perquè l'usuari ho corregeixi.
    - Si tot és correcte, afegeix la tasca a `TasksList` i:
        - posa `afegirState` a `Success` → el diàleg es tanca.
        - posa `tascaActionEvent` a `AFEGIDA` → el `HomeFragment` actualitza el RecyclerView i mostra un Toast de confirmació.

Fixeu-vos que el diàleg no sap res del `HomeFragment` ni del RecyclerView, i el `HomeFragment` no sap res dels camps del formulari. L'únic punt en comú és el ViewModel.

## 1. Model de Dades

```kotlin
// Task.kt
package cat.ivha.sparklestask.model

import android.os.Parcelable
import kotlinx.parcelize.Parcelize

@Parcelize
data class Task(
    val id: Int,
    val title: String,
    val points: Int,
    val data: Long
) : Parcelable
```

### Explicació

- `id` identifica cada tasca de manera única. És important perquè, en una `data class`, dues tasques amb els mateixos valors es consideren iguals (`equals()`). Amb l'`id` podem distingir-les i buscar-les (vegeu `getTaskById` a la secció següent).
- `data` es guarda com a `Long`: el nombre de mil·lisegons des de l'1 de gener de 1970 (*timestamp*), que és el que retorna `System.currentTimeMillis()`.
- `@Parcelize` i `: Parcelable` permeten guardar una `Task` dins d'un `Bundle`, per exemple als `arguments` d'un Fragment o d'un diàleg. En aquest exemple encara no es fa servir, però serà necessari per obrir un diàleg de **modificar** una tasca existent: li haurem de passar la tasca que volem editar.
- `@Parcelize` genera automàticament el codi de `Parcelable`. Cal el plugin `kotlin-parcelize` (vegeu la [secció 8](#8-dependencies-buildgradle)).


## 2. Llista de Tasques (Object)

```kotlin
// TasksList.kt
package cat.ivha.sparklestask.data

import cat.ivha.sparklestask.model.Task

object TasksList {
    val items = mutableListOf<Task>(
        Task(1, "Tasca 1", 10, System.currentTimeMillis()),
        Task(2, "Tasca 2", 20, System.currentTimeMillis()),
        Task(3, "Tasca 3", 30, System.currentTimeMillis())
    )
    
    private var nextId = 4
    
    fun getNextId(): Int = nextId++
    
    fun addTask(task: Task) {
        items.add(task)
    }
    
    
    fun getTaskById(id: Int): Task? {
        return items.find { it.id == id }
    }
}
```

### Explicació

`TasksList` és un `object` (Singleton), igual que el `DataSource` de l'[exemple de RecyclerView](../Interficies/recyclerview.md#llista-de-dades-en-un-object-kotlin): hi ha una única llista compartida per tota l'app mentre està en execució.

- `items` és una `MutableList` perquè s'hi han de poder afegir tasques.
- `nextId` és `private`: des de fora no es pot modificar, només es pot demanar un id nou amb `getNextId()`.
- `getNextId()` retorna `nextId++`: primer retorna el valor actual i **després** l'incrementa. La primera crida retorna 4, la següent 5, etc. Així no es repeteixen ids.
- `getTaskById(id)` utilitza `find`, que retorna el primer element que compleix la condició, o `null` si no n'hi ha cap. Per això el tipus de retorn és `Task?`.

Qui modifica `TasksList` és el ViewModel. Els Fragments només la llegeixen.


## 3. ViewModel

```kotlin
// TasksViewModel.kt
package cat.ivha.sparklestask.viewmodel

import androidx.lifecycle.LiveData
import androidx.lifecycle.MutableLiveData
import androidx.lifecycle.ViewModel
import cat.ivha.sparklestask.data.TasksList
import cat.ivha.sparklestask.model.Task

class TasksViewModel : ViewModel() {
    
    // Tipus d'acció realitzada (l'observa HomeFragment)
    enum class TascaAction {
        AFEGIDA, MODIFICADA, ELIMINADA, NONE
    }

    // Estat del formulari d'afegir (l'observa AfegirTascaDialog)
    sealed class AfegirTascaState {
        object Idle : AfegirTascaState()
        object Success : AfegirTascaState()
        data class Error(val missatge: String) : AfegirTascaState()
    }
    
    // Event per notificar canvis a la llista
    private val _tascaActionEvent = MutableLiveData<TascaAction>()
    val tascaActionEvent: LiveData<TascaAction> = _tascaActionEvent
    
    // Estat del formulari: Idle, Success o Error
    private val _afegirState = MutableLiveData<AfegirTascaState>(AfegirTascaState.Idle)
    val afegirState: LiveData<AfegirTascaState> = _afegirState
    
    /**
     * Afegeix una nova tasca amb validació.
     * El resultat es comunica a través de afegirState.
     */
    fun afegirTasca(title: String, points: String, data: String) {
        // Validació
        if (title.isBlank()) {
            _afegirState.value = AfegirTascaState.Error("El nom és obligatori")
            return
        }
        
        val pointsInt = points.toIntOrNull()
        if (pointsInt == null) {
            _afegirState.value = AfegirTascaState.Error("Els punts han de ser un número")
            return
        }
        
        val dataLong = data.toLongOrNull()
        if (dataLong == null) {
            _afegirState.value = AfegirTascaState.Error("La data no és vàlida")
            return
        }
        
        // Crear i afegir la tasca
        val novaTasca = Task(
            id = TasksList.getNextId(),
            title = title,
            points = pointsInt,
            data = dataLong
        )
        
        TasksList.addTask(novaTasca)
        _afegirState.value = AfegirTascaState.Success
        _tascaActionEvent.value = TascaAction.AFEGIDA
    }
    
   
    /**
     * Reseteja l'event
     */
    fun resetEvent() {
        _tascaActionEvent.value = TascaAction.NONE
    }
    
    /**
     * Torna el formulari a l'estat inicial
     */
    fun resetAfegirState() {
        _afegirState.value = AfegirTascaState.Idle
    }
}
```

### Dos LiveData amb funcions diferents

El ViewModel publica dues informacions diferents, cadascuna per a un observador diferent:

| LiveData | Qui l'observa | Què indica |
|----------|---------------|------------|
| `tascaActionEvent` | `HomeFragment` | Què ha passat amb la llista (s'ha afegit, modificat o eliminat una tasca) |
| `afegirState` | `AfegirTascaDialog` | En quin estat està el formulari (inicial, guardat correctament o amb error) |

Si féssim servir un sol LiveData per a les dues coses, el `HomeFragment` hauria de saber interpretar errors de formulari que no li interessen, i el diàleg rebria canvis de la llista que no necessita.

### `enum class` o `sealed class`?

- `TascaAction` és una `enum class` perquè cada valor és només una etiqueta (`AFEGIDA`, `MODIFICADA`...), sense dades associades. `NONE` representa "no ha passat res" i serveix per reiniciar l'event.
- `AfegirTascaState` és una `sealed class` perquè l'estat `Error` ha de portar **el missatge** d'error, i amb una `enum class` no podríem tenir un missatge diferent per a cada error.
    - `Idle` i `Success` són `object` perquè no porten cap dada: n'hi ha prou amb una única instància.
    - `Error` és una `data class` perquè cada error té el seu `missatge`.
    - Com que la classe és `sealed`, el compilador coneix tots els casos possibles. Si en un `when` ens oblidem d'algun, ens avisarà.

### `MutableLiveData` privat i `LiveData` públic

```kotlin
private val _afegirState = MutableLiveData<AfegirTascaState>(AfegirTascaState.Idle)
val afegirState: LiveData<AfegirTascaState> = _afegirState
```

Només el ViewModel pot canviar el valor (`_afegirState`, privat i mutable). Els Fragments reben la versió `LiveData`, que només permet observar. Així ens assegurem que cap Fragment pugui posar, per exemple, `Success` sense haver passat per la validació. Vegeu [Tipus de LiveData](../Kotlin/livedata.md#4-tipus-de-livedata).

### La funció `afegirTasca`

- Rep els valors com a `String`, tal com surten dels `EditText`. La conversió i la validació són responsabilitat del ViewModel, no del diàleg.
- `toIntOrNull()` i `toLongOrNull()` retornen `null` si el text no és un número, en lloc de llançar una excepció com farien `toInt()` i `toLong()`.
- Cada validació que falla posa l'estat `Error` i fa `return`: la funció s'atura i no s'arriba a crear la tasca.
- Si tot és correcte, afegeix la tasca i actualitza **els dos** LiveData: `Success` per al diàleg i `AFEGIDA` per al `HomeFragment`.

### Per què cal reiniciar els estats?

Un LiveData **guarda l'últim valor**, i quan un observador nou comença a observar-lo, rep immediatament aquest valor.

Si després d'un `Success` no tornéssim l'estat a `Idle`, la propera vegada que s'obrís el diàleg rebria `Success` només començar a observar i es tancaria tot sol. El mateix passaria amb `tascaActionEvent`: si el `HomeFragment` es tornés a crear (per exemple, en girar la pantalla), tornaria a mostrar el Toast "Tasca afegida!".

Per això `resetEvent()` i `resetAfegirState()` tornen els valors a `NONE` i `Idle` un cop s'han tractat.


## 4. DialogFragment per AFEGIR

```kotlin
// AfegirTascaDialog.kt
package cat.ivha.sparklestask.ui.dialogs

import android.app.Dialog
import android.os.Bundle
import android.view.ViewGroup
import android.widget.Button
import android.widget.EditText
import android.widget.Toast
import androidx.appcompat.app.AlertDialog
import androidx.fragment.app.DialogFragment
import androidx.fragment.app.activityViewModels
import cat.ivha.sparklestask.R
import cat.ivha.sparklestask.viewmodel.TasksViewModel

class AfegirTascaDialog : DialogFragment() {

    private val viewModel: TasksViewModel by activityViewModels()
    
    private lateinit var etNom: EditText
    private lateinit var etPoints: EditText
    private lateinit var etData: EditText
    private lateinit var btnGuardar: Button
    private lateinit var btnCancelar: Button

    override fun onCreateDialog(savedInstanceState: Bundle?): Dialog {
        val builder = AlertDialog.Builder(requireContext())
        val inflater = requireActivity().layoutInflater
        val view = inflater.inflate(R.layout.dialog_afegir_tasca, null)

        initViews(view)
        setupListeners()
        observeViewModel()

        builder.setView(view)
        return builder.create()
    }

    private fun initViews(view: android.view.View) {
        etNom = view.findViewById(R.id.etNom)
        etPoints = view.findViewById(R.id.etPoints)
        etData = view.findViewById(R.id.etData)
        btnGuardar = view.findViewById(R.id.btnGuardar)
        btnCancelar = view.findViewById(R.id.btnCancelar)
        
        // Valor per defecte
        etData.setText(System.currentTimeMillis().toString())
    }

    private fun setupListeners() {
        btnGuardar.setOnClickListener {
            val title = etNom.text.toString()
            val points = etPoints.text.toString()
            val data = etData.text.toString()
            
            // Crida al ViewModel (ell valida i actualitza afegirState)
            viewModel.afegirTasca(title, points, data)
        }

        btnCancelar.setOnClickListener {
            dismiss()
        }
    }

    private fun observeViewModel() {
        viewModel.afegirState.observe(this) { state ->
            when (state) {
                is TasksViewModel.AfegirTascaState.Error -> {
                    Toast.makeText(requireContext(), state.missatge, Toast.LENGTH_SHORT).show()
                    viewModel.resetAfegirState()
                }
                TasksViewModel.AfegirTascaState.Success -> {
                    viewModel.resetAfegirState()
                    dismiss()
                }
                TasksViewModel.AfegirTascaState.Idle -> {
                    // No fer res
                }
            }
        }
    }

    override fun onStart() {
        super.onStart()
        dialog?.window?.setLayout(
            ViewGroup.LayoutParams.MATCH_PARENT,
            ViewGroup.LayoutParams.WRAP_CONTENT
        )
        dialog?.window?.setBackgroundDrawableResource(android.R.color.transparent)
    }
}
```

### Obtenir el ViewModel compartit

```kotlin
private val viewModel: TasksViewModel by activityViewModels()
```

`activityViewModels()` retorna el ViewModel associat a l'**Activity**, no al diàleg. Com que el `HomeFragment` també el declara amb `activityViewModels()`, tots dos reben **la mateixa instància**. Si féssim servir `viewModels()`, cadascun tindria el seu propi ViewModel i el `HomeFragment` no s'assabentaria mai de les tasques afegides. Vegeu [ViewModel compartit](./viewmodel.md#viewmodel-compartit-activityviewmodels).

### Crear el diàleg: `onCreateDialog`

- `AlertDialog.Builder` construeix el diàleg, i `setView(view)` hi posa el nostre layout personalitzat (`dialog_afegir_tasca.xml`).
- El layout s'infla amb `null` com a pare perquè encara no tenim cap `ViewGroup` on posar-lo: és el diàleg qui el col·locarà.
- El codi es divideix en tres funcions amb noms clars (`initViews`, `setupListeners`, `observeViewModel`), igual que farem al `HomeFragment`.

### Per què botons propis i no `setPositiveButton`?

L'`AlertDialog.Builder` permet afegir botons amb `setPositiveButton` i `setNegativeButton`, però aquests botons **tanquen el diàleg automàticament** en prémer-los. Si les dades són incorrectes, el diàleg es tancaria igualment i l'usuari perdria el que ha escrit.

Amb botons propis dins del layout, som nosaltres qui decidim quan tancar: només quan el ViewModel respon `Success`.

### Els listeners

- **Guardar** → llegeix el text dels tres camps i el passa al ViewModel. El diàleg no valida res: només recull dades.
- **Cancel·lar** → `dismiss()` tanca el diàleg sense fer res més.

### Observar l'estat del formulari

- `Error` → mostra el missatge en un Toast i torna l'estat a `Idle`. El diàleg continua obert.
- `Success` → torna l'estat a `Idle` i tanca el diàleg amb `dismiss()`.
- `Idle` → no cal fer res. Com que `AfegirTascaState` és una `sealed class`, el `when` ens obliga a tractar tots els casos.

!!! info "Per què `observe(this)` i no `observe(viewLifecycleOwner)`?"
    Quan el diàleg es crea amb `onCreateDialog`, el `DialogFragment` no té una vista pròpia creada a `onCreateView`, i accedir a `viewLifecycleOwner` llançaria una excepció. Per això s'observa amb el cicle de vida del mateix Fragment (`this`). El `HomeFragment`, en canvi, sí que té vista i utilitza `viewLifecycleOwner`.

### Ajustar la finestra: `onStart`

La finestra del diàleg (`dialog?.window`) encara no existeix dins de `onCreateDialog`. Per això els ajustos es fan a `onStart`, quan el diàleg ja s'ha creat i està a punt de mostrar-se.

- `setLayout(MATCH_PARENT, WRAP_CONTENT)` → el diàleg ocupa tota l'amplada de la pantalla i l'alçada que necessiti el contingut.
- `setBackgroundDrawableResource(android.R.color.transparent)` → elimina el fons per defecte del diàleg (el rectangle blanc amb marges). A partir d'aquí, l'únic fons que es veu és el del layout (`android:background` a `dialog_afegir_tasca.xml`, vegeu la [secció 7](#7-layouts)).

## 5. HomeFragment

```kotlin
// HomeFragment.kt
package cat.ivha.sparklestask.ui.fragments

import android.os.Bundle
import android.view.View
import android.widget.Button
import android.widget.Toast
import androidx.fragment.app.Fragment
import androidx.fragment.app.activityViewModels
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import cat.ivha.sparklestask.R
import cat.ivha.sparklestask.adapter.TasksAdapter
import cat.ivha.sparklestask.data.TasksList
import cat.ivha.sparklestask.ui.dialogs.AfegirTascaDialog
import cat.ivha.sparklestask.viewmodel.TasksViewModel

class HomeFragment : Fragment(R.layout.home_rv) {

    private val viewModel: TasksViewModel by activityViewModels()

    private lateinit var recyclerView: RecyclerView
    private lateinit var adapter: TasksAdapter
    private lateinit var btnAfegir: Button

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        initComponents(view)
        setupRecyclerView()
        setupListeners()
        observeViewModel()
    }

    private fun initComponents(view: View) {
        recyclerView = view.findViewById(R.id.rvTasques)
        btnAfegir = view.findViewById(R.id.btnAfegir)
    }

    private fun setupRecyclerView() {
        recyclerView.layoutManager = LinearLayoutManager(requireContext())
        
        adapter = TasksAdapter(
            itemsComplets = TasksList.items
        )
        
        recyclerView.adapter = adapter
    }

    private fun setupListeners() {
        btnAfegir.setOnClickListener {
            AfegirTascaDialog().show(parentFragmentManager, "AfegirTascaDialog")
        }
    }

    private fun observeViewModel() {
        viewModel.tascaActionEvent.observe(viewLifecycleOwner) { action ->
            when (action) {
                TasksViewModel.TascaAction.AFEGIDA -> {
                    adapter.updateTasks(TasksList.items)                    
                    Toast.makeText(requireContext(), "Tasca afegida!", Toast.LENGTH_SHORT).show()
                    viewModel.resetEvent()
                }
                TasksViewModel.TascaAction.MODIFICADA -> {
                }
                TasksViewModel.TascaAction.ELIMINADA -> {
                }
                TasksViewModel.TascaAction.NONE -> {
                    // No fer res
                }
            }
        }
    }
}
```

### Explicació

- `Fragment(R.layout.home_rv)` → passant el layout al constructor, el Fragment l'infla automàticament i no cal sobreescriure `onCreateView`. El codi d'inicialització va a `onViewCreated`, quan la vista ja existeix.
- `by activityViewModels()` → obté la mateixa instància del ViewModel que el diàleg.
- `setupRecyclerView()` → configura el RecyclerView com a l'[exemple base](../Interficies/recyclerview.md#6-activity), però amb `requireContext()` en lloc de `this`, perquè un Fragment no és un `Context`.
- `AfegirTascaDialog().show(parentFragmentManager, "AfegirTascaDialog")` → mostra el diàleg. El segon paràmetre és una etiqueta (*tag*) que permet trobar el diàleg més endavant amb `findFragmentByTag`.

### Observar els canvis de la llista

- S'observa amb `viewLifecycleOwner` perquè la vista d'un Fragment es pot destruir i tornar a crear (per exemple, en navegar a una altra pantalla i tornar) mentre el Fragment continua existint. Amb `viewLifecycleOwner`, l'observador s'elimina quan es destrueix la vista i no s'acumulen observadors repetits.
- `AFEGIDA` → actualitza l'adapter, mostra el Toast i reinicia l'event amb `resetEvent()` (vegeu [Per què cal reiniciar els estats?](#per-que-cal-reiniciar-els-estats)).
- `MODIFICADA` i `ELIMINADA` estan buits: queden preparats per quan s'afegeixin aquestes operacions al ViewModel.

!!! info "Simplificació"
    El `HomeFragment` llegeix les dades directament de `TasksList.items`. En una arquitectura més completa, el ViewModel exposaria també la llista com a `LiveData<List<Task>>` i el Fragment no accediria mai a la capa de dades. Vegeu [Capa repositori i dades](./capa_dades_android.md).



## 6. Adapter

```kotlin
// TasksAdapter.kt
package cat.ivha.sparklestask.adapter

import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.TextView
import androidx.recyclerview.widget.RecyclerView
import cat.ivha.sparklestask.R
import cat.ivha.sparklestask.model.Task

class TasksAdapter(
    private var itemsComplets: List<Task>
) : RecyclerView.Adapter<TasksAdapter.TaskViewHolder>() {

    class TaskViewHolder(view: View) : RecyclerView.ViewHolder(view) {
        val tvTitle: TextView = view.findViewById(R.id.tvTitle)
        val tvPoints: TextView = view.findViewById(R.id.tvPoints)
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): TaskViewHolder {
        val view = LayoutInflater.from(parent.context)
            .inflate(R.layout.item_task, parent, false)
        return TaskViewHolder(view)
    }

    override fun onBindViewHolder(holder: TaskViewHolder, position: Int) {
        val task = itemsComplets[position]
        
        holder.tvTitle.text = task.title
        holder.tvPoints.text = "Points: ${task.points}"
    }

    override fun getItemCount(): Int = itemsComplets.size

    fun updateTasks(newTasks: List<Task>) {
        itemsComplets = newTasks
        notifyDataSetChanged()
    }
}
```

### Explicació

- `itemsComplets` és `var` perquè `updateTasks` la pugui substituir per una llista nova.
- El `TaskViewHolder` està declarat **dins** de l'adapter i només guarda les referències a les vistes. Les dades les escriu directament `onBindViewHolder`. A l'[exemple base de RecyclerView](../Interficies/recyclerview.md#5-adapter-i-viewholder-myadapterkt), en canvi, el ViewHolder és una classe separada amb una funció `bind()`. Totes dues maneres són vàlides; aquí s'ha optat per la versió més curta.
- `updateTasks(newTasks)` substitueix la llista i crida `notifyDataSetChanged()`, que torna a pintar totes les files.

!!! info "`notifyDataSetChanged()` o `notifyItemInserted()`?"
    `notifyDataSetChanged()` és senzill i sempre funciona, però redibuixa tota la llista i no fa animacions. Com que en aquest cas sabem que s'ha afegit una sola tasca al final, també es podria fer servir `notifyItemInserted(itemsComplets.size - 1)`. Vegeu [Mètodes de notificació de l'Adapter](../Interficies/recyclerviewaddelete.md#4-metodes-de-notificacio-de-ladapter).

## 7. Layouts

### dialog_afegir_tasca.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:padding="24dp"
    android:background="?attr/colorSurface">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Afegir Nova Tasca"
        android:textSize="20sp"
        android:textStyle="bold"
        android:layout_marginBottom="16dp"/>

    <EditText
        android:id="@+id/etNom"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Nom de la tasca"
        android:layout_marginBottom="12dp"/>

    <EditText
        android:id="@+id/etPoints"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Points"
        android:inputType="number"
        android:layout_marginBottom="12dp"/>

    <EditText
        android:id="@+id/etData"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Data (timestamp)"
        android:inputType="number"
        android:layout_marginBottom="16dp"/>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:gravity="end">

        <Button
            android:id="@+id/btnCancelar"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Cancel·lar"
            style="@style/Widget.Material3.Button.TextButton"
            android:layout_marginEnd="8dp"/>

        <Button
            android:id="@+id/btnGuardar"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Guardar"/>
    </LinearLayout>

</LinearLayout>
```

### Explicació

- `android:background="?attr/colorSurface"` → dona fons al diàleg. És necessari perquè a `onStart` hem fet transparent el fons de la finestra; sense aquest atribut, els camps es veurien directament sobre la pantalla enfosquida. `?attr/colorSurface` agafa el color de superfície del tema, de manera que s'adapta automàticament al mode clar i al mode fosc.
- Els `id` dels camps (`etNom`, `etPoints`, `etData`) i dels botons (`btnGuardar`, `btnCancelar`) són els que es busquen a `initViews` del diàleg.
- `android:inputType="number"` fa que aparegui el teclat numèric, però **no garanteix** que el valor sigui correcte (el camp pot quedar buit). Per això el ViewModel continua validant amb `toIntOrNull()`.
- El botó **Cancel·lar** té l'estil `Widget.Material3.Button.TextButton` (botó sense fons), que és el que s'utilitza habitualment per a l'acció secundària d'un diàleg. Cal que el tema de l'app sigui Material 3.
- El `LinearLayout` horitzontal amb `android:gravity="end"` alinea els dos botons a la dreta.

### Altres layouts

Els layouts `home_rv.xml` i `item_task.xml` no es mostren perquè són equivalents als de l'[exemple de RecyclerView](../Interficies/recyclerview.md#3-layout-de-cada-element-item_rowxml). Només cal que tinguin aquests `id`:

- `home_rv.xml` → un RecyclerView (`rvTasques`) i un botó (`btnAfegir`).
- `item_task.xml` → dos TextView (`tvTitle` i `tvPoints`).





## 8. Dependències (build.gradle)

```gradle
dependencies {
    // ViewModel
    implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:2.6.2"
    implementation "androidx.fragment:fragment-ktx:1.6.2"
    
    // LiveData
    implementation "androidx.lifecycle:lifecycle-livedata-ktx:2.6.2"
    
    // Material Design
    implementation "com.google.android.material:material:1.10.0"
}

// Al principi del fitxer
plugins {
    id 'kotlin-parcelize'
}
```

### Explicació

| Dependència | Per a què serveix |
|-------------|-------------------|
| `lifecycle-viewmodel-ktx` | La classe `ViewModel` |
| `fragment-ktx` | Els delegats `by viewModels()` i `by activityViewModels()` |
| `lifecycle-livedata-ktx` | `LiveData` i `MutableLiveData` |
| `material` | Els estils Material, com el `TextButton` del diàleg |
| plugin `kotlin-parcelize` | L'anotació `@Parcelize` de `Task` |

!!! info "Sintaxi Kotlin DSL"
    L'exemple utilitza la sintaxi Groovy (`build.gradle`). Els projectes nous d'Android Studio fan servir Kotlin DSL (`build.gradle.kts`), on cal posar parèntesis i cometes dobles:

    ```kotlin
    plugins {
        id("kotlin-parcelize")
    }

    dependencies {
        implementation("androidx.fragment:fragment-ktx:1.6.2")
        // ...
    }
    ```

## 9. Resum

1. El diàleg i el Fragment comparteixen el mateix ViewModel gràcies a `activityViewModels()`.
2. El diàleg només recull les dades del formulari i les envia al ViewModel. No valida ni modifica la llista.
3. El ViewModel valida, modifica `TasksList` i publica el resultat amb dos LiveData: `afegirState` per al diàleg i `tascaActionEvent` per al `HomeFragment`.
4. Els estats amb dades (com un missatge d'error) es representen amb una `sealed class`.
5. Un cop tractat un estat o un event, cal reiniciar-lo perquè no es torni a rebre en tornar a observar.
