# Guia Simplificada: DialogFragment amb ViewModel
## Per estudiants de DAM - Afegir, Modificar i Eliminar



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
│  • Emet LiveData<String?> (errorMessage)                                           │
└───┬─────────────────────────────────────────┬─────────────────┬────────────────────┘
    │                                         ▲                 │
    │  observa tascaActionEvent               │ AfegirTasca VM  │ observa errorMessage
    │                                         │                 │
    ▼                                         │                 ▼
┌──────────────────────────────────┐         ┌──────────────────────────────────────┐
│        HomeFragment              │         │   AfegirTascaDialog                  │
│                                  │         │                                      │
│  • Observa tascaActionEvent      │         │  • Observa errorMessage              │
│  • Actualitza RecyclerView       │────────▶│  • Mostra Toast d'error              │
│  • Mostra Toast de confirmació   │         │  • Crida viewModel                   │
│  • Crida dialogs                 │         │    amb dades del formulari           │
│                                  │         │                                      │
└──────────────────────────────────┘         └──────────────────────────────────────┘


```

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
    
    // Tipus d'acció realitzada
    enum class TascaAction {
        AFEGIDA, MODIFICADA, ELIMINADA, NONE
    }
    
    // Event per notificar canvis
    private val _tascaActionEvent = MutableLiveData<TascaAction>()
    val tascaActionEvent: LiveData<TascaAction> = _tascaActionEvent
    
    // Missatge d'error si la validació falla
    private val _errorMessage = MutableLiveData<String?>()
    val errorMessage: LiveData<String?> = _errorMessage
    
    /**
     * Afegeix una nova tasca amb validació
     */
    fun afegirTasca(title: String, points: String, data: String): Boolean {
        // Validació
        if (title.isBlank()) {
            _errorMessage.value = "El nom és obligatori"
            return false
        }
        
        val pointsInt = points.toIntOrNull()
        if (pointsInt == null) {
            _errorMessage.value = "Els punts han de ser un número"
            return false
        }
        
        val dataLong = data.toLongOrNull()
        if (dataLong == null) {
            _errorMessage.value = "La data no és vàlida"
            return false
        }
        
        // Crear i afegir la tasca
        val novaTasca = Task(
            id = TasksList.getNextId(),
            title = title,
            points = pointsInt,
            data = dataLong
        )
        
        TasksList.addTask(novaTasca)
        _tascaActionEvent.value = TascaAction.AFEGIDA
        _errorMessage.value = null
        return true
    }
    
   
    /**
     * Reseteja l'event
     */
    fun resetEvent() {
        _tascaActionEvent.value = TascaAction.NONE
    }
    
    /**
     * Reseteja el missatge d'error
     */
    fun resetError() {
        _errorMessage.value = null
    }
}
```



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
            
            // Crida al ViewModel (ell valida)
            val success = viewModel.afegirTasca(title, points, data)
            
            if (success) {
                dismiss()
            }
        }

        btnCancelar.setOnClickListener {
            dismiss()
        }
    }

    private fun observeViewModel() {
        viewModel.errorMessage.observe(this) { error ->
            error?.let {
                Toast.makeText(requireContext(), it, Toast.LENGTH_SHORT).show()
                viewModel.resetError()
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

## 5. HomeFragment

```kotlin
// HomeFragment.kt
package cat.ivha.sparklestask.ui.fragments

import android.os.Bundle
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
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
import cat.ivha.sparklestask.ui.dialogs.ModificarTascaDialog
import cat.ivha.sparklestask.viewmodel.TasksViewModel

class HomeFragment : Fragment(R.layout.home_rv) {

    private val viewModel: TasksViewModel by activityViewModels()

    private lateinit var recyclerView: RecyclerView
    private lateinit var adapter: TasksAdapter
    private lateinit var btnAfegir: Button

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        return inflater.inflate(R.layout.home_rv, container, false)
    }

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

## 7. Layouts

### dialog_afegir_tasca.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:padding="24dp">

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
