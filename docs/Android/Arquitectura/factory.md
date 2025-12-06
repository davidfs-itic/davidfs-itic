## 1. Què és?

El patró Factory és un patró de disseny que proporciona una manera de crear objectes **sense especificar** la classe exacta de l'objecte que crearà.
En comptes d'aixó, es confia a un métode "factory" que crei i retorni l'objecte, tipicament basat en paràmetres d'entrada o configuració.


El principi fonamental és que el codi que necessita un objecte, interactua amb una interfície comuna i no necessita saber exactament quina classe concreta s'està instant. El "Factory" és l'entitat responsable de decidir i crear la instància correcta.

### Components del patró:

- Una Interfície/Classe Base del "Producte": Defineix el contracte comú que tots els objectes creats pel Factory han de seguir. A Kotlin, això sol ser una interface o una abstract class.

- Productes Concrets: Són les classes que implementen la interfície del Producte. Són les instàncies que el Factory crearà.

- El Factory: La classe o funció que conté la lògica per decidir quin Producte Concret instanciar i retornar.

## Com s'implementa?

Imagina que necessitem mostrar diferents tipus de Notificacions (per exemple, una notificació simple, una amb imatge, o una de progrés). 

En lloc que cada part del teu codi d'Android (com una Activity o ViewModel) sàpiga com construir cada tipus de notificació amb els seus paràmetres específics, podem usar un Factory.

### 1. Interfície del "Producte" (Notification)
Definim la interfície comuna que totes les notificacions han d'implementar.

```Kotlin
interface AppNotification {
    fun show(context: Context)
}
```
### 2. Productes Concrets (Simple, Image, Progress Notification)
Implementem les classes específiques de notificacions.

```Kotlin
class SimpleNotification(private val message: String) : AppNotification {
    override fun show(context: Context) {
        // Lògica de construcció d'una notificació simple d'Android
        println("Mostrant notificació simple: $message")
    }
}

class ImageNotification(private val imageUrl: String) : AppNotification {
    override fun show(context: Context) {
        // Lògica de construcció amb imatge (més complexa)
        println("Mostrant notificació amb imatge des de: $imageUrl")
    }
}
// Podria haver-n'hi més, com ProgressNotification, etc.
```
### 3. El Factory (NotificationFactory)
Creem un objecte **object** (que actua com un Singleton a Kotlin) amb una funció que decideix quin tipus de producte instanciar basant-se en el parametre type

```Kotlin
// Usem una 'sealed class' o 'enum' per definir els tipus
sealed class NotificationType {
    data class Simple(val message: String) : NotificationType()
    data class Image(val url: String) : NotificationType()
    object Progress : NotificationType() // Exemple d'un tipus simple
}

object NotificationFactory {

    fun createNotification(type: NotificationType): AppNotification = when (type) {
        is NotificationType.Simple -> SimpleNotification(type.message)
        is NotificationType.Image -> ImageNotification(type.url)
        NotificationType.Progress -> ProgressNotification() // Assumint que existeix
    }
}
```

### 4. Ús a Activity/ViewModel
Ara, en el codi d'Android, només cal cridar al Factory i treballar amb la interfície AppNotification, sense preocupar-se pels detalls de creació.

```Kotlin
// En una Activity o ViewModel
fun showNotification(data: MessageData, context: Context) {
    val type = if (data.hasImage) {
        NotificationType.Image(data.imageUrl)
    } else {
        NotificationType.Simple(data.text)
    }

    // El codi de la Activity només interactua amb el Factory i la interfície
    val notification = NotificationFactory.createNotification(type)
    notification.show(context)
}
```
## Resum

El Patró Factory no s'utilitza per amagar què vols crear, sinó COM es crea.

S'oculta:
#### 1. La Classe Concreta
El Factory amaga l'existència (i el nom) de les classes que implementen la interfície del producte.

- Sense Factory: El l'Activity hauria de fer:
        
    ```Kotlin
    val notification: AppNotification = SimpleNotification(data.text) // Necessita saber el nom exacte
    ```
- Amb Factory: El l'Activity només veu el tipus d'entrada (NotificationType) i el tipus de sortida (AppNotification).
    ```Kotlin
    val notification: AppNotification = NotificationFactory.createNotification(type) // No coneix SimpleNotification
    ```
    
#### 2. La Lògica Complexa de Construcció
l Factory s'utilitza per amagar tota la lògica de configuració interna necessària per construir la instància.
- Sense Factory

    ```kotlin 
    // El client s'ha de preocupar: 
    val builder = NotificationCompat.Builder(context, CHANNEL_ID) 
        .setContentTitle("New message") 
        .setContentText(data.text) 
        .setSmallIcon(R.drawable.ic_noti) // ... 10 línies de configuració més 
    ```kotlin

 - Amb Factory (Lògica oculta)
    ```kotlin 
    // El client només subministra les dades:
    val notification = NotificationFactory.createNotification(type)
    ```


### Exemple de Més Ocultació

Imagina que el Factory rep només les dades en brut (`MessageData`) i decideix internament.

```kotlin
data class MessageData(
    val title: String,
    val text: String,
    val hasImage: Boolean,
    val imageUrl: String?
)

object SmartNotificationFactory {
    // El Factory rep DADES GENÈRIQUES, no un tipus de Producte pre-escollit.
    fun createNotificationFromData(data: MessageData): AppNotification {
        return if (data.hasImage) {
            // El Factory decideix la classe i fa la instanciació
            ImageNotification(data.imageUrl!!) 
        } else {
            SimpleNotification(data.text)
        }
    }
}