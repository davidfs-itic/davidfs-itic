# Filtre de categories amb un quadre de diàleg

Aquest document continua el de [filtres de categoria a un RecyclerView](./recyclerviewfilter.md). Allà mostràvem les categories amb un [PopupMenu](./recyclerviewfilter.md#4-filtrar-amb-un-popupmenu); aquí farem el mateix amb un quadre de diàleg (`AlertDialog`) que mostra la llista de categories al centre de la pantalla.

El model de dades (`Categoria`, `MyItem`, `DataSource`), l'adapter amb `updateList`, `llistaCompleta` i `applyCategoryFilter` són exactament els mateixos de l'[apartat 3](./recyclerviewfilter.md#3-filtrar-amb-un-submenu) del document anterior. Aquí només veurem el que canvia.

Documentació oficial:

- [Dialogs](https://developer.android.com/develop/ui/views/components/dialogs)
- [AlertDialog.Builder](https://developer.android.com/reference/androidx/appcompat/app/AlertDialog.Builder)

## 1. Què canvia respecte del PopupMenu

- **No cal `popup_categories.xml`.** Les opcions del diàleg es generen a partir de l'enum `Categoria`. Si afegim una categoria nova a l'enum, apareix sola al diàleg.
- **No cal ancoratge.** El diàleg no penja de cap View, apareix centrat. Per tant el botó de la barra pot tornar a ser `ifRoom`.
- **No cal el `when` amb els id.** El diàleg ens diu la **posició** de l'opció triada, i a partir d'aquesta obtenim la categoria.
- **Guardem un índex en lloc d'un id.** `idSeleccionat` passa a ser `seleccionat: Int`, la posició de l'opció marcada.

### Flux

- L'usuari prem el botó "Categories" de la barra superior.
- Android crida `onOptionsItemSelected`.
- Creem i mostrem un `AlertDialog` amb la llista de categories.
- L'usuari tria una categoria i s'executa el listener del diàleg.
- Cridem `applyCategoryFilter`, igual que abans, i tanquem el diàleg.

## 2. Menú de la barra (XML)

Fitxer `res/menu/menu_cerca.xml`. Només té el botó que obre el diàleg:

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <item
        android:id="@+id/action_category_button"
        android:title="Categories"
        app:showAsAction="ifRoom" />
</menu>
```

- `app:showAsAction="ifRoom"`: com que el diàleg no necessita cap View per ancorar-se, no passa res si el botó va a parar al menú de desbordament (els tres punts).

`onCreateOptionsMenu` és igual que en el document anterior.

## 3. Obrir el diàleg

A `onOptionsItemSelected` capturem el botó de la barra i obrim el diàleg. Fixeu-vos que ara no cal passar cap View:

```kotlin
override fun onOptionsItemSelected(item: MenuItem): Boolean {
    return when (item.itemId) {
        R.id.action_category_button -> {
            showCategoryDialog()
            true
        }
        else -> super.onOptionsItemSelected(item)
    }
}
```

## 4. Crear el diàleg

### Les opcions

El diàleg necessita un `Array<String>` amb els textos de les opcions. El construïm amb "Totes les categories" a la primera posició, seguit del nom de cada valor de l'enum:

```kotlin
val opcions = arrayOf("Totes les categories") + Categoria.entries.map { it.nom }
// ["Totes les categories", "Pel·lícules", "Llibres", "Música"]
```

- `Categoria.entries`: llista amb tots els valors de l'enum, en l'ordre en què estan declarats. En versions de Kotlin anteriors a la 1.9 s'utilitza `Categoria.values()`.
- `map { it.nom }`: transforma cada categoria en el seu nom.

### De la posició a la categoria

Quan l'usuari tria una opció, rebem la seva posició. Com que la posició 0 és "Totes les categories", les categories de l'enum estan desplaçades una posició:

| Posició | Opció | Categoria |
|---|---|---|
| 0 | Totes les categories | `null` |
| 1 | Pel·lícules | `Categoria.entries[0]` |
| 2 | Llibres | `Categoria.entries[1]` |
| 3 | Música | `Categoria.entries[2]` |

Per tant: si la posició és 0, la categoria és `null`; si no, és `Categoria.entries[posicio - 1]`.

!!! info "Per què generem les opcions des de l'enum?"
    Si escrivíssim els textos a mà, hauríem de vigilar que l'ordre de la llista coincidís amb el de l'enum. Generant-los des de `Categoria.entries`, l'ordre sempre és el mateix i no ens podem equivocar.

### El codi

Guardem a l'Activity la posició de l'opció marcada:

```kotlin
// Posició de l'opció marcada al diàleg (0 = Totes les categories)
private var seleccionat = 0
```

```kotlin
private fun showCategoryDialog() {
    val opcions = arrayOf("Totes les categories") + Categoria.entries.map { it.nom }

    AlertDialog.Builder(this)
        .setTitle("Categories")
        .setSingleChoiceItems(opcions, seleccionat) { dialog, posicio ->
            seleccionat = posicio
            val categoria = if (posicio == 0) null else Categoria.entries[posicio - 1]
            applyCategoryFilter(categoria)
            dialog.dismiss() // tanquem el diàleg en triar una opció
        }
        .setNegativeButton("Cancel·la", null)
        .show()
}
```

- `AlertDialog.Builder(this)`: crea el diàleg pas a pas. Cada mètode afegeix una part i retorna el mateix `Builder`, per això els podem encadenar.
- `setTitle`: títol que apareix a dalt del diàleg.
- `setSingleChoiceItems(opcions, seleccionat) { ... }`: mostra les opcions amb botons de selecció (radio buttons), on només una pot estar marcada. Fa la mateixa funció que el `checkableBehavior="single"` del menú.
    - `opcions`: els textos.
    - `seleccionat`: la posició que apareix marcada en obrir el diàleg.
    - La lambda rep el diàleg i la posició triada.
- `dialog.dismiss()`: amb `setSingleChoiceItems` el diàleg no es tanca sol en triar una opció; l'hem de tancar nosaltres.
- `setNegativeButton("Cancel·la", null)`: botó per tancar sense canviar res. Amb `null` com a listener, el botó només tanca el diàleg.
- `show()`: crea el diàleg i el mostra.

Igual que el `PopupMenu`, el diàleg es crea de nou cada vegada. Per això cal guardar `seleccionat` i passar-lo a `setSingleChoiceItems`.

## 5. Variant: confirmar amb un botó "D'acord"

Amb el codi anterior, el filtre s'aplica tan bon punt l'usuari toca una opció. Si preferim que l'usuari pugui canviar d'opció i confirmar al final, apliquem el filtre al botó positiu:

```kotlin
private fun showCategoryDialog() {
    val opcions = arrayOf("Totes les categories") + Categoria.entries.map { it.nom }

    // Opció marcada mentre el diàleg està obert (encara no confirmada)
    var triada = seleccionat

    AlertDialog.Builder(this)
        .setTitle("Categories")
        .setSingleChoiceItems(opcions, seleccionat) { _, posicio ->
            triada = posicio // només ho recordem, no apliquem res
        }
        .setPositiveButton("D'acord") { _, _ ->
            seleccionat = triada
            val categoria = if (triada == 0) null else Categoria.entries[triada - 1]
            applyCategoryFilter(categoria)
        }
        .setNegativeButton("Cancel·la", null)
        .show()
}
```

- `triada` és una variable local: si l'usuari prem "Cancel·la", es perd i `seleccionat` continua tenint el valor anterior.
- Els botons positiu i negatiu sí que tanquen el diàleg automàticament, per això ara no cal `dismiss()`.
- `_` indica un paràmetre de la lambda que no fem servir.

## 6. PopupMenu o diàleg?

| | PopupMenu | Diàleg |
|---|---|---|
| Opcions | Fitxer XML | Generades des de l'enum |
| Ancoratge | Cal una View (`always`) | No en cal (`ifRoom`) |
| Opció triada | Id de l'element del menú | Posició a la llista |
| Botons | No | Sí (cancel·lar, confirmar) |
| Experiència d'usuari | Lleuger, apareix al costat del botó | Interromp més, ocupa el centre de la pantalla |

!!! warning "Rotació de la pantalla"
    Un `AlertDialog` creat directament des de l'Activity es tanca si l'usuari gira el dispositiu, perquè l'Activity es torna a crear. Per mantenir-lo obert cal un `DialogFragment`. Vegeu [Diàlegs](./dialegs.md).

## 7. Resum

- El diàleg reutilitza el model de dades, l'adapter i `applyCategoryFilter` del [document anterior](./recyclerviewfilter.md).
- Les opcions es generen des de l'enum, i la posició triada es converteix en categoria amb `Categoria.entries[posicio - 1]`.
- `setSingleChoiceItems` mostra les opcions amb botons de selecció, i cal guardar `seleccionat` perquè el diàleg es crea de nou cada vegada.
- Podem aplicar el filtre en tocar l'opció o esperar que l'usuari confirmi amb un botó positiu.
