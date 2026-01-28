# Ús de la llibreria Retrofit en Android (Kotlin)

## 1. Introducció

En el desenvolupament d'aplicacions Android modernes és molt habitual consumir dades des d'una **API REST**. Per facilitar aquesta tasca, una de les llibreries més utilitzades és **Retrofit**, desenvolupada per Square.

Retrofit permet:
- Fer peticions HTTP de manera senzilla
- Convertir automàticament les respostes JSON en objectes Kotlin
- Integrar-se fàcilment amb **RecyclerView**, **ViewModel** i **Coroutines**

Exemple retrofit.
https://github.com/davidfs-itic/RecyclerView

Explicació pas a pas Retrofit:
https://www.youtube.com/watch?v=2_DnhfQrwXQ


## 2. Què és Retrofit?

Retrofit és una llibreria client HTTP que:
- Defineix les crides a l'API mitjançant **interfícies**
- Utilitza **anotacions** per indicar el tipus de petició (`@GET`, `@POST`, etc.)
- Converteix les respostes (normalment JSON) en objectes Kotlin mitjançant convertidors com **Gson**

Retrofit no és una API REST, sinó una eina per **consumir-ne**.


## 3. Dependències necessàries


Cal accés a internet des de l'aplicació: (AndroidManifest.xml)
```xml
<uses-permission android:name="android.permission.INTERNET" />
```

Afegir dependències en el **build.gradle** (build.gradle Module)

De forma tradicional, present en molta documentació: (ens demanarà si volem transformar-la a format libs.version.toml)

```kotlin
dependencies {
    // Retrofit Core
    implementation("com.squareup.retrofit2:retrofit:2.9.0")
    // Converter per gestionar JSON (Generalment es fa servir GSON)
    implementation("com.squareup.retrofit2:converter-gson:2.9.0")
    // (Opcional) Interceptor per poder veure els logs de les peticions (molt útil per a debug)
    implementation("com.squareup.okhttp3:logging-interceptor:4.12.0")
}
```

Sincronitza el projecte després d'afegir-les.


## 4. Exemple de l'API

Suposem que la nostra API té aquest endpoint:

```
GET https://api.exemple.com/items
```

I retorna una resposta JSON similar a:

```json
[
  {
    "id": 1,
    "nom": "Item 1",
    "descripcio": "Descripció del primer item"
  },
  {
    "id": 2,
    "nom": "Item 2",
    "descripcio": "Descripció del segon item"
  }
]
```


## 5. Model de dades (`Item`)

Creem una **data class** que representi l'objecte que retorna l'API:

```kotlin
data class Item(
    val id: Int,
    val nom: String,
    val descripcio: String
)
```

Els noms dels atributs han de coincidir amb els camps del JSON.



## 6. Implementació 

### 6.1 Interfície `ItemsService`

```kotlin
interface ItemsService {

    @GET("items/")
    suspend fun llistaItems(): Response<List<Item>>

    @GET("items/categoria/{idcategoria}")
    suspend fun llistaItemsPerCategoria(
        @Path("idcategoria") idcategoria: Int
    ): Response<List<Item>>
}
```

#### Explicació pas a pas

- `interface ItemsService`
  - Defineix el **contracte** de comunicació amb l'API.
  - Retrofit crearà automàticament la implementació.

- `@GET("items/")`
  - Indica una petició HTTP **GET** a l'endpoint `/items/`.

- `suspend fun llistaItems()`
  - És una funció **suspend**, per tant s'ha d'executar dins d'una coroutine.
  - Permet fer la crida sense bloquejar el fil principal.

- `Response<List<Item>>`
  - Conté tant les dades com la informació HTTP (codi, error, etc.).

- `@Path("idcategoria")`
  - Substitueix `{idcategoria}` per el valor rebut com a paràmetre.


### 6.2 Classe `ItemAPI` (Singleton)

```kotlin
class ItemAPI {
    companion object {
        private var mItemAPI: ItemService? = null

        @Synchronized
        fun API(): ItemService {
            if (mItemAPI == null) {

                val gsondateformat = GsonBuilder()
                    .setDateFormat("yyyy-MM-dd'T'HH:mm:ss")
                    .create()

                mItemAPI = Retrofit.Builder()
                    .addConverterFactory(GsonConverterFactory.create(gsondateformat))
                    .baseUrl("https://oracleitic.mooo.com/")
                    .build()
                    .create(ItemService::class.java)
            }
            return mItemAPI!!
        }
    }
}
```

#### Què fa aquesta classe?

Aquesta classe s'encarrega de:
- Crear **una sola instància** de Retrofit
- Garantir que totes les crides utilitzen la mateixa configuració

És un patró **Singleton**.


### 6.3 Explicació detallada

- `companion object`
  - Permet accedir a la funció `API()` sense crear un objecte `ItemAPI`.

- `private var mItemAPI`
  - Desa la instància del servei Retrofit.

- `@Synchronized`
  - Evita que dos fils creïn l'objecte al mateix temps.

- `GsonBuilder().setDateFormat(...)`
  - Permet convertir correctament dates en format ISO.

- `baseUrl(...)`
  - URL base del servidor.
  - **Sempre ha d'acabar amb `/`**.

- `create(ItemService::class.java)`
  - Retrofit genera la implementació automàticament.


### 6.4 Exemple d'ús en un ViewModel

```kotlin
viewModelScope.launch {
    try {
        val response = ItemAPI.API().llistaItems()

        if (response.isSuccessful) {
            val items = response.body() ?: emptyList()
        } else {
            Log.e("API", "Error HTTP: ${response.code()}")
        }
    } catch (e: Exception) {
        Log.e("API", "Error de connexió")
    }
}
```

#### Explicació

`viewModelScope.launch { ... }`

- Executa el codi dins d'una coroutine.
- viewModelScope està lligat al cicle de vida del ViewModel.
 - Quan el ViewModel es destrueix, la coroutine es cancel·la automàticament.
 - Evita fuites de memòria (memory leaks).

`try { ... } catch (e: Exception)`

- Permet capturar errors de xarxa o connexió.
- Retrofit llança excepcions en casos com:
- Sense connexió a internet
- Timeout
- Servidor inabastable

`val response = ItemAPI.API().llistaItems()`

- Crida a l'API mitjançant Retrofit.
- Aquesta funció és suspend, per tant:
    - No bloqueja el fil principal
    - S'executa en segon pla
- API() retorna la instància Singleton del servei.


`if (response.isSuccessful)`

- Comprova si el servidor ha retornat un codi HTTP entre 200 i 299.
- Només en aquest cas es considera una resposta correcta.
- val items = response.body() ?: emptyList()
- response.body() conté la llista d'Item rebuda.
- Pot ser null si el servidor no envia cos.
- L'operador ?: evita errors retornant una llista buida.

`else { Log.e("API", "Error HTTP: ${response.code()}") }`

- Gestiona errors HTTP com:
- 404 → recurs no trobat
- 500 → error del servidor
- response.code() retorna el codi HTTP.

`catch (e: Exception)`

- Captura errors que no són HTTP.
- Errors típics:
- Connexió fallida
- Tall de xarxa
- Error DNS


## 7. Retrofit sense certificats (NOMES PER A DEBUG)

Si utlitzem una APi, i aquesta no esta configurada amb cerficats vàlids, Retrofit no ens connectarà.

Caldra afegir una llibreria: okhttp3

```kotlin
dependencies {
    // altres dependències    
    implementation("com.squareup.okhttp3:okhttp:4.12.0")
}
```


Cal fer la seguent modificació a la api:

```kotlin
class ItemAPI {
    companion object {
        private var mItemAPI: ItemService? = null

        @Synchronized
        fun API(): ItemService {
            if (mItemAPI == null) {

                val gsondateformat = GsonBuilder()
                    .setDateFormat("yyyy-MM-dd'T'HH:mm:ss")
                    .create()

                // Client HTTP insegur (només per desenvolupament)
                val unsafeOkHttpClient = getUnsafeOkHttpClient()

                mItemAPI = Retrofit.Builder()
                    .addConverterFactory(GsonConverterFactory.create(gsondateformat))
                    .baseUrl("https://oracleitic.mooo.com/")
                    .client(unsafeOkHttpClient) // Afegeix el client
                    .build()
                    .create(ItemService::class.java)
            }
            return mItemAPI!!
        }

        private fun getUnsafeOkHttpClient(): OkHttpClient {
            try {
                // Crea un trust manager que NO valida certificats
                val trustAllCerts = arrayOf<TrustManager>(
                    object : X509TrustManager {
                        override fun checkClientTrusted(chain: Array<X509Certificate>, authType: String) {}
                        override fun checkServerTrusted(chain: Array<X509Certificate>, authType: String) {}
                        override fun getAcceptedIssuers(): Array<X509Certificate> = arrayOf()
                    }
                )

                // Instal·la el trust manager
                val sslContext = SSLContext.getInstance("SSL")
                sslContext.init(null, trustAllCerts, java.security.SecureRandom())
                val sslSocketFactory = sslContext.socketFactory

                return OkHttpClient.Builder()
                    .sslSocketFactory(sslSocketFactory, trustAllCerts[0] as X509TrustManager)
                    .hostnameVerifier { _, _ -> true } // Accepta qualsevol hostname
                    .build()

            } catch (e: Exception) {
                throw RuntimeException(e)
            }
        }
    }
}

```
 Amb els imports corresponents
 
```kotlin
import okhttp3.OkHttpClient
import java.security.cert.X509Certificate
import javax.net.ssl.SSLContext
import javax.net.ssl.TrustManager
import javax.net.ssl.X509TrustManager
```


## 8. Altra informació

[**Veure Kotlin/Anotadors**](https://davidfs-itic.github.io/davidfs-itic/Android/Kotlin/anotadors.md)

[**Veure Kotlin/Corrutines**](https://davidfs-itic.github.io/davidfs-itic/Android/Kotlin/corrutines.md)




[**RETROFIT 2 AVANZADO en KOTLIN - (POST, GET, MULTIPART, HEADER...) - Android Studio 2022**](https://www.youtube.com/watch?v=L3pM5YuxYp4)