# De l'arquitectura Android a Clean Architecture

!!! note "Prerequisit"
    Aquest document parteix de l'arquitectura en 2 capes (UI + Dades) explicada a [Capa repositori i dades](capa_dades_android.md). Es recomana llegir-lo primer.

## 1. Punt de partida: Arquitectura Android

Recordem l'arquitectura que teníem:

```
┌──────────────────────────────────────────────────┐
│  UI LAYER                                        │
│  Activity/Fragment → ViewModel                   │
│  (depèn de ItemsRepository, classe concreta)     │
└──────────────┬───────────────────────────────────┘
               │
               ▼
┌──────────────────────────────────────────────────┐
│  DATA LAYER                                      │
│                                                  │
│  ItemsRepository (classe concreta)               │
│  └─→ ItemsDataSource (interface)                 │
│      ├─→ MockItemsDataSource                     │
│      └─→ RetrofitItemsDataSource                 │
└──────────────────────────────────────────────────┘
```

Amb les classes:

```kotlin
// Data Layer
interface ItemsDataSource {
   fun getItems(): List<MyItem>
   fun addItem(item: MyItem)
   fun updateItem(item: MyItem)
   fun deleteItem(item: MyItem)
}

class ItemsRepository(
    private val dataSource: ItemsDataSource
) {
    fun getItems(): List<MyItem> = dataSource.getItems()
    fun addItem(item: MyItem) = dataSource.addItem(item)
    fun updateItem(item: MyItem) = dataSource.updateItem(item)
    fun deleteItem(item: MyItem) = dataSource.deleteItem(item)
}

// UI Layer
class ItemsViewModel(
    private val repository: ItemsRepository  // classe concreta
) : ViewModel() {
    fun getItems(): List<MyItem> = repository.getItems()
    fun addItem(item: MyItem) = repository.addItem(item)
}
```

**Què ja funciona bé**: Gràcies a la interface `ItemsDataSource`, podem canviar la font de dades (Mock, Retrofit, Room) sense modificar ni el Repository ni el ViewModel. Per testejar amb dades falses, n'hi ha prou passant un `MockItemsDataSource` al Repository.

**Limitació**: El ViewModel depèn de la classe concreta `ItemsRepository`. Si volem testejar el ViewModel de forma **aïllada** (sense cap Repository real), no podem crear fàcilment un "fake Repository" perquè no hi ha una interface per implementar. Necessitaríem una llibreria de mocking (com Mockk) per simular la classe concreta.

Amb Clean Architecture, aplicarem DIP **entre capes**: el ViewModel dependrà d'una interface del Repository (definida al Domain), cosa que permetrà substituir el Repository complet per un fake en els tests.

## 2. Pas 1 — Clean Architecture mínima: afegir la interface del Repository

El canvi mínim per passar a Clean Architecture és:

1. Crear una **capa de Domini** (Domain Layer)
2. Crear la **interface del Repository** al Domain
3. Moure l'**entitat** (`MyItem`) al Domain
4. La implementació del Repository queda al Data Layer

### Què canvia i què no canvia

| Classe | Android Architecture | Clean Architecture mínima |
|---|---|---|
| `MyItem` | Data Layer | **→ Domain Layer** |
| `ItemsRepository` (interface) | No existia | **→ Domain Layer (NOVA)** |
| `ItemsRepository` (classe concreta) | Data Layer | **→ Reanomenada a `ItemsRepositoryImpl`**, es queda al Data Layer |
| `ItemsDataSource` (interface) | Data Layer | Data Layer (sense canvis) |
| `MockItemsDataSource` | Data Layer | Data Layer (sense canvis) |
| `RetrofitItemsDataSource` | Data Layer | Data Layer (sense canvis) |
| `ItemsViewModel` | UI Layer | UI Layer (canvia el tipus de dependència) |

### Diagrama resultant

```
┌──────────────────────────────────────────────────┐
│  UI LAYER                                        │
│  ItemsViewModel                                  │
│  (depèn de ItemsRepository, INTERFACE del Domain)│
└──────────────┬───────────────────────────────────┘
               │ depèn de
               ▼
┌──────────────────────────────────────────────────┐
│  DOMAIN LAYER                                    │
│                                                  │
│  ItemsRepository (interface)                     │
│  MyItem (entitat)                                │
└──────────────────────────────────────────────────┘
               ▲
               │ implementa (depèn de)
┌──────────────┴───────────────────────────────────┐
│  DATA LAYER                                      │
│                                                  │
│  ItemsRepositoryImpl (implementa la interface)   │
│  └─→ ItemsDataSource (interface)                 │
│      ├─→ MockItemsDataSource                     │
│      └─→ RetrofitItemsDataSource                 │
└──────────────────────────────────────────────────┘
```

Fixeu-vos en la **direcció de les fletxes**: tant la UI com el Data apunten cap al Domain. El Domain no depèn de ningú. Aquesta és la inversió de dependències entre capes.

### Codi

#### Domain Layer

L'entitat i la interface del Repository:

```kotlin
// domain/model/MyItem.kt
data class MyItem(
    val id: Int,
    val title: String,
    val subtitle: String
)

// domain/repository/ItemsRepository.kt
interface ItemsRepository {
    fun getItems(): List<MyItem>
    fun addItem(item: MyItem)
    fun updateItem(item: MyItem)
    fun deleteItem(item: MyItem)
}
```

El Domain **només conté definicions** (interfaces i entitats). No conté cap implementació ni cap dependència externa.

#### Data Layer

La implementació del Repository i els DataSources (gairebé igual que abans):

```kotlin
// data/datasource/ItemsDataSource.kt
interface ItemsDataSource {
    fun getItems(): List<MyItem>
    fun addItem(item: MyItem)
    fun updateItem(item: MyItem)
    fun deleteItem(item: MyItem)
}

// data/repository/ItemsRepositoryImpl.kt
class ItemsRepositoryImpl(
    private val dataSource: ItemsDataSource
) : ItemsRepository {  // ← implementa la interface del DOMAIN

    override fun getItems(): List<MyItem> = dataSource.getItems()
    override fun addItem(item: MyItem) = dataSource.addItem(item)
    override fun updateItem(item: MyItem) = dataSource.updateItem(item)
    override fun deleteItem(item: MyItem) = dataSource.deleteItem(item)
}

// data/datasource/MockItemsDataSource.kt
class MockItemsDataSource : ItemsDataSource {
    private val items = mutableListOf(
        MyItem(1, "Títol 1", "Subtítol A"),
        MyItem(2, "Títol 2", "Subtítol B"),
        MyItem(3, "Títol 3", "Subtítol C")
    )
    override fun getItems(): List<MyItem> = items
    override fun addItem(item: MyItem) { items.add(item) }
    override fun updateItem(item: MyItem) { /* implementació */ }
    override fun deleteItem(item: MyItem) { /* implementació */ }
}
```

L'únic canvi respecte l'arquitectura Android és que `ItemsRepositoryImpl` ara implementa `ItemsRepository` (interface del Domain) en lloc de ser una classe independent.

#### UI Layer

El ViewModel ara depèn de la **interface** (Domain), no de la classe concreta (Data):

```kotlin
// ui/ItemsViewModel.kt
class ItemsViewModel(
    private val repository: ItemsRepository  // ← INTERFACE, no classe concreta
) : ViewModel() {

    fun getItems(): List<MyItem> = repository.getItems()
    fun addItem(item: MyItem) = repository.addItem(item)
}
```

#### Com es connecta tot

```kotlin
val dataSource: ItemsDataSource = if (isTesting) {
    MockItemsDataSource()
} else {
    RetrofitItemsDataSource()
}

// El tipus declarat és la INTERFACE (Domain), la implementació és del Data
val repository: ItemsRepository = ItemsRepositoryImpl(dataSource)

val viewModel = ItemsViewModel(repository)
```

### Què hem guanyat?

Amb aquest canvi mínim (una interface + moure l'entitat):

- **DIP entre capes**: La UI i el Data depenen del Domain. El Domain no depèn de ningú.
- **Testabilitat del ViewModel**: Podem crear un mock del Repository sense necessitar cap DataSource.
- **Domain estable**: Si canviem tota la capa de dades (de Retrofit a Room, per exemple), el Domain i la UI no es veuen afectats.

## 3. Pas 2 — Clean Architecture completa: afegir Use Cases

El pas anterior ja ens dóna Clean Architecture. Però el ViewModel encara crida directament el Repository. En projectes grans, això pot ser un problema:

- Si dos ViewModels necessiten la mateixa operació amb la mateixa lògica, la dupliquem.
- La lògica de negoci queda escampada entre ViewModels.

La solució són els **Use Cases** (o Interactors): classes al Domain que encapsulen **una única operació de negoci**.

### Què canvia

| Classe | Clean mínima | Clean completa |
|---|---|---|
| `GetItemsUseCase` | No existia | **→ Domain Layer (NOU)** |
| `AddItemUseCase` | No existia | **→ Domain Layer (NOU)** |
| `ItemsViewModel` | Depèn de `ItemsRepository` | **→ Depèn dels Use Cases** |
| Resta de classes | — | Sense canvis |

### Diagrama resultant

```
┌──────────────────────────────────────────────────┐
│  UI LAYER                                        │
│  ItemsViewModel                                  │
│  (depèn dels Use Cases)                          │
└──────────────┬───────────────────────────────────┘
               │ depèn de
               ▼
┌──────────────────────────────────────────────────┐
│  DOMAIN LAYER                                    │
│                                                  │
│  GetItemsUseCase ──┐                             │
│  AddItemUseCase    ├─→ ItemsRepository (interface)│
│  ...               ┘                             │
│  MyItem (entitat)                                │
└──────────────────────────────────────────────────┘
               ▲
               │ implementa (depèn de)
┌──────────────┴───────────────────────────────────┐
│  DATA LAYER                                      │
│                                                  │
│  ItemsRepositoryImpl                             │
│  └─→ ItemsDataSource (interface)                 │
│      ├─→ MockItemsDataSource                     │
│      └─→ RetrofitItemsDataSource                 │
└──────────────────────────────────────────────────┘
```

### Codi

#### Domain Layer — Use Cases

Cada Use Case té una única responsabilitat: executar una operació.

```kotlin
// domain/usecase/GetItemsUseCase.kt
class GetItemsUseCase(
    private val repository: ItemsRepository
) {
    fun execute(): List<MyItem> {
        return repository.getItems()
    }
}

// domain/usecase/AddItemUseCase.kt
class AddItemUseCase(
    private val repository: ItemsRepository
) {
    fun execute(item: MyItem) {
        repository.addItem(item)
    }
}
```

!!! note "Per què una classe per operació?"
    Pot semblar excessiu crear una classe per cada operació. Però els Use Cases són el lloc on afegir lògica de negoci (validacions, transformacions, combinació de repositoris). Si un Use Case només delega al Repository, és perquè encara no té lògica pròpia — però ja tenim l'estructura preparada per quan en necessitem.

#### UI Layer — ViewModel amb Use Cases

El ViewModel ja no coneix el Repository. Només coneix els Use Cases:

```kotlin
// ui/ItemsViewModel.kt
class ItemsViewModel(
    private val getItemsUseCase: GetItemsUseCase,
    private val addItemUseCase: AddItemUseCase
) : ViewModel() {

    fun getItems(): List<MyItem> = getItemsUseCase.execute()

    fun addItem(item: MyItem) = addItemUseCase.execute(item)
}
```

#### Com es connecta tot

```kotlin
// Capa de dades
val dataSource: ItemsDataSource = RetrofitItemsDataSource()
val repository: ItemsRepository = ItemsRepositoryImpl(dataSource)

// Capa de domini
val getItemsUseCase = GetItemsUseCase(repository)
val addItemUseCase = AddItemUseCase(repository)

// Capa UI
val viewModel = ItemsViewModel(getItemsUseCase, addItemUseCase)
```

## 4. Resum de la transició

### Classes per capa a cada pas

```
ANDROID ARCHITECTURE (2 capes):
  UI:     ItemsViewModel → ItemsRepository (concreta)
  Data:   ItemsRepository, ItemsDataSource(I), Mock, Retrofit

CLEAN MÍNIMA (3 capes, sense Use Cases):
  UI:     ItemsViewModel → ItemsRepository (interface)
  Domain: ItemsRepository(I), MyItem
  Data:   ItemsRepositoryImpl, ItemsDataSource(I), Mock, Retrofit

CLEAN COMPLETA (3 capes, amb Use Cases):
  UI:     ItemsViewModel → GetItemsUseCase, AddItemUseCase
  Domain: GetItemsUseCase, AddItemUseCase, ItemsRepository(I), MyItem
  Data:   ItemsRepositoryImpl, ItemsDataSource(I), Mock, Retrofit
```

### Direcció de les dependències

```
Android:        UI ──────▶ Data              (cap avall)
Clean mínima:   UI ──────▶ Domain ◀────── Data  (inversió entre capes)
Clean completa: UI ──────▶ Domain ◀────── Data  (igual, amb Use Cases)
```

### Taula comparativa

| | Android | Clean mínima | Clean completa |
|---|---|---|---|
| **Capes** | 2 (UI + Data) | 3 (UI + Domain + Data) | 3 (UI + Domain + Data) |
| **Repository** | Classe concreta | Interface (Domain) + Impl (Data) | Interface (Domain) + Impl (Data) |
| **ViewModel depèn de** | `ItemsRepository` (classe) | `ItemsRepository` (interface) | Use Cases |
| **Domain conté** | — | Interface Repository + Entitats | Interface Repository + Use Cases + Entitats |
| **DIP entre capes** | No | Sí | Sí |
| **Lògica de negoci** | Al ViewModel | Al ViewModel | Als Use Cases |

## 5. Quan utilitzar cada enfocament?

- **Android Architecture (2 capes)**: Projectes petits o prototips. Suficient quan no necessites testejar el ViewModel de forma aïllada i la lògica de negoci és mínima.

- **Clean mínima (3 capes, sense Use Cases)**: Projectes mitjans. Vols independència entre capes i testabilitat, però no tens lògica de negoci complexa ni compartida entre ViewModels.

- **Clean completa (3 capes, amb Use Cases)**: Projectes grans. Tens lògica de negoci que es reutilitza entre ViewModels, múltiples fonts de dades, o necessites una capa de domini sòlida i testable.

!!! warning "Nota important"
    La documentació oficial d'Android inclou aquesta nota: *"The term 'domain layer' is used in other software architectures, such as 'clean' architecture, and has a different meaning there. Don't confuse the definition of 'domain layer' defined in the Android official architecture guidance with other definitions you may have read elsewhere."*

    El Domain Layer d'Android (opcional, dependències cap avall) **no és el mateix** que el Domain Layer de Clean Architecture (obligatori, independent, amb inversió de dependències).
