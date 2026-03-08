# Càrrega d'imatges amb Glide

## 1. Què és Glide?

**Glide** és una llibreria desenvolupada per Google per a la **càrrega, cache i visualització d'imatges** en aplicacions Android. Permet:

- Carregar imatges des d'una **URL**, un fitxer local o un recurs drawable
- Gestionar automàticament la **memòria cau** (cache en memòria i en disc)
- Redimensionar i transformar imatges de forma eficient
- Mostrar **placeholders** mentre la imatge es carrega i imatges d'error si falla


## 2. Afegir la llibreria al projecte

### Permís d'internet

Si carregarem imatges des d'una URL, cal afegir el permís al fitxer `AndroidManifest.xml`:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

### Dependència

Al fitxer **build.gradle.kts (Module: app)**:

```kotlin
dependencies {
    implementation("com.github.bumptech.glide:glide:4.16.0")
}
```

Després fes **Sync Now**.


## 3. Ús bàsic

### Carregar una imatge des d'una URL en un ImageView

```kotlin
val imageView = findViewById<ImageView>(R.id.imageView)

Glide.with(this)
    .load("https://exemple.com/imatge.jpg")
    .into(imageView)
```

- `with(this)` — rep el context (Activity, Fragment o View).
- `load(url)` — la font de la imatge (URL, fitxer, URI, recurs drawable...).
- `into(imageView)` — l'`ImageView` on es mostrarà la imatge.


## 4. Placeholder i imatge d'error

Podem mostrar una imatge mentre es carrega i una altra si hi ha un error:

```kotlin
Glide.with(this)
    .load("https://exemple.com/imatge.jpg")
    .placeholder(R.drawable.loading)   // Imatge mentre carrega
    .error(R.drawable.error_image)     // Imatge si falla la càrrega
    .into(imageView)
```


## 5. Transformacions d'imatge

### Imatge circular

```kotlin
Glide.with(this)
    .load("https://exemple.com/avatar.jpg")
    .circleCrop()
    .into(imageView)
```

### Canviar la mida

```kotlin
Glide.with(this)
    .load("https://exemple.com/imatge.jpg")
    .override(300, 200)  // Ample x Alt en píxels
    .into(imageView)
```

### centerCrop i fitCenter

```kotlin
Glide.with(this)
    .load("https://exemple.com/imatge.jpg")
    .centerCrop()   // Retalla la imatge per omplir l'ImageView
    .into(imageView)
```

```kotlin
Glide.with(this)
    .load("https://exemple.com/imatge.jpg")
    .fitCenter()    // Escala la imatge perquè càpiga sencera
    .into(imageView)
```


## 6. Ús en un RecyclerView

Un dels usos més habituals de Glide és carregar imatges dins d'un **RecyclerView**. Al mètode `onBindViewHolder` de l'Adapter:

```kotlin
class ItemAdapter(private val items: List<Item>) :
    RecyclerView.Adapter<ItemAdapter.ItemViewHolder>() {

    class ItemViewHolder(view: View) : RecyclerView.ViewHolder(view) {
        val imageView: ImageView = view.findViewById(R.id.itemImage)
        val textView: TextView = view.findViewById(R.id.itemName)
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ItemViewHolder {
        val view = LayoutInflater.from(parent.context)
            .inflate(R.layout.item_layout, parent, false)
        return ItemViewHolder(view)
    }

    override fun onBindViewHolder(holder: ItemViewHolder, position: Int) {
        val item = items[position]
        holder.textView.text = item.nom

        Glide.with(holder.itemView.context)
            .load(item.imatgeUrl)
            .placeholder(R.drawable.loading)
            .error(R.drawable.error_image)
            .into(holder.imageView)
    }

    override fun getItemCount() = items.size
}
```

Glide gestiona automàticament el **reciclatge de vistes**: cancel·la les càrregues anteriors quan una vista es reutilitza.


## 7. Netejar la cache

Si cal forçar la recàrrega d'una imatge ignorant la cache:

```kotlin
Glide.with(this)
    .load("https://exemple.com/imatge.jpg")
    .skipMemoryCache(true)                        // Ignora cache en memòria
    .diskCacheStrategy(DiskCacheStrategy.NONE)    // Ignora cache en disc
    .into(imageView)
```

Els imports necessaris:

```kotlin
import com.bumptech.glide.load.engine.DiskCacheStrategy
```
