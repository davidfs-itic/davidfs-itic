# Llibreria Retrofit

## 1. Què és


- Retrofit és una llibreria de client HTTP de tipus segur per a Java i Kotlin desenvolupada per Square. 
- Simplifica dràsticament la connexió a APIs REST, 
- Permet serialitzar i deserialitzar automàticament objectes Kotlin/Java a JSON (o XML, etc.) i viceversa, mitjançant Converter Factories (com Gson, Moshi, kotlinx.serialization).


Exemple retrofit.
https://github.com/davidfs-itic/RecyclerView

Explicació pas a pas Retrofit:
https://www.youtube.com/watch?v=2_DnhfQrwXQ



## 2. Setup dependències

Cal accés a internet des de l'aplicació: (AndroidManifest.xml)
```xml
<uses-permission android:name="android.permission.INTERNET" />
```

Afegir dependències en el **build.gradle** (nivell aplicació)

De forma tradicional, present en molta documentació: (ens demanarà si volem transformar-la a format libs.version.toml)

```kotlin
// Retrofit Core
implementation("com.squareup.retrofit2:retrofit:2.9.0")
// Converter per gestionar JSON (Generalment es fa servir GSON)
implementation("com.squareup.retrofit2:converter-gson:2.9.0")
// (Opcional) Interceptor per poder veure els logs de les peticions (molt útil per a debug)
implementation("com.squareup.okhttp3:logging-interceptor:4.12.0")
```

o amb la nova versio, anar a libs.version.toml:
```kotlin
[versions]
retrofit= "2.9.0"
converter-gson = "2.9.0"
[libraries]
rretrofit = { group = "com.squareup.retrofit2", name = "retrofit", version.ref = "retrofit" }
converter-gson= { group = "com.squareup.retrofit2", name = "converter-gson", version.ref = "converter-gson" }
```

i finalment a dependencies del **build.gradle**, amb una sintàxi diferent:

```kotlin
dependencies {
    // retrofit
    implementation(libs.retrofit)
    // gson converter
    implementation (libs.converter.gson)
```
## 3. Implementació


Creem el servei Retrofit, en un arxiu ItemsService:

```kotlin
interface ItemsService {

@GET ("items/")
suspend fun LlistaItems():Response<List<Item>>
}

@GET ("items/categoria/{idcategoria}")
suspend fun LlistaItemsPerCategoria ( @Path("idcategoria") idcagegoria:Int):Response<List<Item>>
}
```

```kotlin
class ItemAPI {
    companion object {
        private var mItemAPI : ItemService? = null

        // L'anotador per a que sigui thread-safe
        // Garanteix que només un fil pugui crear la instància per evitar problemes
        @Synchronized
        fun API(): ItemService {
            if (mItemAPI == null){
                
                // 2. Configuració de Gson per al format de data ISO
                val gsondateformat = GsonBuilder()
                    .setDateFormat("yyyy-MM-dd'T'HH:mm:ss") // Format típic per a dates i hores
                    .create();
                
                // 3. Creació de la instància Retrofit
                mItemAPI = Retrofit.Builder()
                    .addConverterFactory(GsonConverterFactory.create(gsondateformat))
                    .baseUrl("https://oracleitic.mooo.com") // URL base del teu servei d'Items
                    .build()
                    // 4. Creació de la implementació de la interfície ItemService
                    .create(ItemService::class.java)
            }
            return mItemAPI!!
        }
    }
}
```

## 4. Altra informació

[**Veure Kotlin/Anotadors**](https://davidfs-itic.github.io/davidfs-itic/Android/Kotlin/anotadors.md)

[**Veure Kotlin/Corrutines**](https://davidfs-itic.github.io/davidfs-itic/Android/Kotlin/corrutines.md)




[**RETROFIT 2 AVANZADO en KOTLIN - (POST, GET, MULTIPART, HEADER...) - Android Studio 2022**](https://www.youtube.com/watch?v=L3pM5YuxYp4)