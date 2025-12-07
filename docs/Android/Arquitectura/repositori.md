# Capa repositori i dades

## 1. Situació inicial


Partint del codi relatiu al Recyclerview, veurem que té algunes mancances o dificultats.
No respecta els principis de clean code, ni té cap arquitectura definida:


### Problemes d'arquitectura

**Absència de separació de capes:** 
 La Activity accedeix directament a les dades (directament a la llista en memòria o através d'una API).Això crea un acoblament fort i dificulta enormement el testing i el manteniment.

**DataSource com a singleton estàtic:** 
Qualsevol canvi en com s'obtenen les dades (API, base de dades local, etc.) requerirà modificar directament la Activity, però també tots els llocs on s'utilitza.

**Lògica de negoci a la UI:** La Activity està fent massa coses: configura el RecyclerView, gestiona el clic, i decideix què mostrar (el Toast o ). No hi ha cap ViewModel ni cap capa intermèdia que gestioni l'estat o la lògica.


## Problemes que pot ocasionar
- Escalabilitat nul·la: Quan el projecte creixi i necessitis múltiples fonts de dades, càrrega asíncrona, caché, o sincronització, aquest codi serà molt difícil de mantenir.
- Impossibilitat de testing automàtic: Aquest codi fa els tests pràcticament impossibles.
- Duplicació de codi futura: Sense una arquitectura clara, cada nova funcionalitat similar acabarà repetint patrons, copiant i enganxant codi.

## 2. Anàlisi codi Incorrecte:(sense separació de capa)

Item: Classe que conté les dades que necessitem (pot ser recepta, pelicula, reserva, itemcompra, ticket, etc..)

```kotlin 
// Fitxer: MyItem.kt
data class MyItem(
    val title: String,
    val subtitle: String
)
```

Datasource: Classe (objecte en aquest cas) que conté la llista d'items

```kotlin 
// Fitxer: DataSource.kt

/**
 * Objecte singleton que proporciona dades en memòria
 * per ser utilitzades al RecyclerView.
 */
object DataSource {

    // Llista de dades en memòria (només de lectura)
    val items: List<MyItem> = listOf(
        MyItem("Element 1", "Subtítol 1"),
        MyItem("Element 2", "Subtítol 2"),
        MyItem("Element 3", "Subtítol 3"),
        MyItem("Element 4", "Subtítol 4")
    )
}
```
Des de l'activity, es crea la llista d'items. Aixó és incorrecte doncs no és la tasca de l'activity.
```kotlin
        // 3. Crear llista de dades (des de DataSource o directament)
        val items = DataSource.items
```



### Objectiu del Clean Clode i arquitectura per capes

Hem de fer independent les dades de la implementació, perque la UI pugui cridar un métode  genèric. Per tant, ha de ser independent 

Aquesta classe s'anomena Repositori, i correspón a la **Capa de domini**

#### D'aquesta manera, tindrem 3 capes:

1. UI (Activities, ViewModels)
2. Domini (Repositori)
3. Dades (Datasources amb Interface)

**Repository**: Coordinador i font de dades. Recupera les dades, pero no directament, sinó a través dels datasources.

Per exemple:
```kotlin 
class ItemsRepository(
    private val dataSource: ItemsDataSource
) {
    
    fun getItems(): List<MyItem> {
         dataSource.getItems()   
    }
}
```

Per què necessites la capa de Domini (amb el Repositori)?
Per diferents responsabilitats

- Repository: S'encarrega de la lògica de negoci relacionada amb les dades:
    - Gestió de caché
    - Combinació de múltiples fonts de dades (local + remota)
    - Decisió de quan usar dades locals vs remotes
    - Transformació de dades entre capes
    - Gestió d'errors i fallbacks
- DataSource: S'encarrega NOMÉS d'obtenir/guardar dades d'una font específica (mock, API, base de dades).

## 3. Implementació correcte (amb separació de capes UI,Domini,Dades)

### Capa Domini
(la classe repositori que hem vist abans), hi ha diferents maneres d'implementar-la i a més es poden afegir els casos d'ús.

```kotlin 
class ItemsRepository(
    private val dataSource: ItemsDataSource
) {
    
    fun getItems(): List<MyItem> {
         dataSource.getItems()   
    }
}
```
### Capa Dades

Interface +  Datasources concrets.

```kotlin
interface ItemDataSource {
   fun getItems(): List<MyItem>
   fun addItem(item: MyItem)
   fun updateItem(item: MyItem)
   fun deleteItem(item: MyItem)
}
```

I ara podem crear múltiples datasources, amb implementacions diferents (Mock, api, bbdd)

!!! note "Què és Mock?"
    Mock es refereix a dades falses, i s'utilitza en programació per provar les classes abans no es tingui una base de dades, api, etc.
    En aquest cas, el MockItemDataSource és una llista precodificada (com havíem fet abans en el nostre object Datasource)


```kotlin
class MockItemDataSource : ItemDataSource {
   private val items = mutableListOf(
       MyItem(1, "Element A"),
       MyItem(2, "Element B"),
       MyItem(3, "Element C")
   )
   override fun getItems(): List<MyItem> = items
   override fun addItem(item: MyItem) { items.add(item) }
   override fun updateItem(item: MyItem) {//implementacio  }
   override fun deleteItem(item: MyItem) {//implementacio }
}


class RetrofitItemDataSource : ItemDataSource {
   override fun getItems(): List<MyItem> { ... }
   override fun addItem(item: MyItem) { ... }
   override fun updateItem(item: MyItem) { ... }
   override fun deleteItem(item: MyItem) {
       RetrofitClient.instance.deleteItem(item.id).execute()
   }
}
```

D'aquesta manera el Repositori no necessita saber com estan implementades (com s'aconsegueixen, es guarden, etc) les dades.

Ara podem fer fàcilment un 
```kotlin
val ItemDataSource = if (“Testing”) {
        CreateMockItemDataSource()
    } else {
        CreateRetrofitItemDataSource()
    }
 
val itemRepository = ItemRepository(ItemDataSource)

//i ja es pot fer additem independentment de si es testing o si es api, room, etc…
itemRepository.addItem(newitem)
```
Es poden crear altres datasources per coses diferents, però el repositori no té accés a les dades, ho fa a través del interface.



## 4. Explicació capes

Aquestes són les capes que han intervingut:


![Domain layer](./Imatges/domainlayer.png)

- Presentation (UI) Layer: ViewModel, estats UI i la Activity
- Domain Layer: Conté l'entitat Item, el contracte ItemRepository i els Use Cases per a cada operació CRUD. Els Use Cases els explicarem en un altre apartat.
- Data Layer: Implementa el repository i datasources. Conté l'entitat de base de dades, DAO (si n'hi ha).

Dins el DataLayer:
![Data layer](./Imatges/datalayer.png){ align=left style="height: 30%; width: 30%; border-radius: 5px;" loading=lazy}


### Principis de Clean Code aplicats:

- Single Responsibility: Cada classe té una única responsabilitat
- Dependency Inversion: Les capes superiors no depenen de les inferiors directament
- Separation of Concerns: Cada capa té la seva responsabilitat ben definida


!!! info
    El Domain Layer és el nucli de l'aplicació i roman estable independentment de com implementis la persistència o la presentació. Això és el que fa que l'arquitectura sigui tan robusta i mantenible.

### Esquema general:
```

┌─────────────────────┐
│   PRESENTATION      │  ← Depèn del Domain
│  (Activity/Fragment)│
└──────────┬──────────┘
           │ depèn de (a través dels casos d’ús o repositori)
           ▼
┌─────────────────────┐
│     DOMAIN          │  ← INDEPENDENT (no depèn de res)
│   (Use Cases)       │    Defineix contractes (interfaces)
└──────────▲──────────┘	   i els casos d’us
           │ implementa
           │
┌─────────────────────┐
│      DATA           │  ← Depèn del Domain (implementa les interfaces)
│  (Repository Impl)  │    a través de la inversió de dependències.
└─────────────────────┘
```


!!! info
    Veure apartat d'inversió de dependències

!!! info
    Veure exemple implementació RecyclerView amb inversió de dependències
    (Exemple DIP recycler)
