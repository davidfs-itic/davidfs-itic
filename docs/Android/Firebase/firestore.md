# 1. Què és Cloud Firestore?
Cloud Firestore és una base de dades de tipus NoSQL allotjada al núvol, que forma part de la plataforma Firebase de Google. 

A diferència de les bases de dades relacionals clàssiques (com MySQL o PostgreSQL), Firestore no fa servir taules ni files, sinó que organitza la informació en Documents i Col·leccions.


## 1. L'Estructura de Dades

L'arquitectura de Firestore es basa en tres pilars:

- Document: És la unitat bàsica de dades. S'assembla a un fitxer JSON i conté parelles de "clau-valor" (per exemple: nom: "Cadirat", preu: 25).

- Col·lecció: És un contenidor de documents. Imagina-ho com una carpeta que guarda molts "fitxers" (documents) del mateix tipus (per exemple, una col·lecció anomenada items).

- Camps (Fields): Són les dades específiques dins d'un document (strings, números, booleans, llistes, etc.).

Jerarquia: 

L'estructura segueix sempre el patró Col·lecció > Document > Col·lecció > Document.

## 2. Característiques Principals

- Temps Real: Firestore pot "avisar" a l'aplicació Android cada vegada que una dada canvia al servidor, actualitzant la interfície automàticament sense haver de refrescar.

- Suport Offline: L'aplicació pot seguir escrivint i llegint dades encara que no hi hagi internet; Firestore sincronitza els canvis automàticament quan es recupera la connexió.

- Escalabilitat: Està dissenyada per créixer sense que el rendiment de les consultes se'n ressenti, ideal tant per a projectes petits com per a aplicacions amb milions d'usuaris.


# 2. Inicialitzar Firestore
Dins del teu Activity, Fragment o ViewModel, obté la instància de la base de dades:


```kotlin
    val db = Firebase.firestore
```

# 3. Guardar Dades
Afegir un nou ítem.

Podem deixar que Firestore generi un ID automàtic o especificar-ne un. Aquí fem servir un ID generat automàticament:


```kotlin

fun guardarItem(nom: String, descripcio: String, categoria: String) {
    // Creem una referència a un nou document (genera l'ID automàticament)
    val nouDocument = db.collection("items").document()
    
    val item = Item(
        id = nouDocument.id, // Guardem l'ID generat dins de l'objecte
        nom = nom,
        descripcio = descripcio,
        categoria = categoria
    )

    nouDocument.set(item)
        .addOnSuccessListener {
            // Èxit en la pujada
        }
        .addOnFailureListener { e ->
            // Error en la pujada
        }
}
```

# 4. Recuperar Dades


## Obtenir la llista completa (One-time fetch)

Aquesta operació llegeix tota la llista d'ítems. 
S'utilitza en llistes que no canvien constantment.



```kotlin
fun obtenirItems() {
    db.collection("items")
        .get()
        .addOnSuccessListener { result ->
            // Convertim tots els documents a una llista d'objectes Item
            val llistaItems = result.toObjects(Item::class.java)
            
            // Aquí pots carregar la llista al teu RecyclerView Adapter
            actualitzarUI(llistaItems)
        }
        .addOnFailureListener { exception ->
            // Gestionar l'error
        }
}
```

## Actualització en temps real (Real-time updates)

Si vols que la llista s'actualitzi automàticament quan hi ha canvis a la base de dades:



```kotlin
fun escoltarCanvis() {
    db.collection("items")
        .addSnapshotListener { snapshot, e ->
            if (e != null) return@addSnapshotListener

            if (snapshot != null) {
                val llistaItems = snapshot.toObjects(Item::class.java)
                actualitzarUI(llistaItems)
            }
        }
}
```

# 5. Filtrar Dades

Firestore permet fer consultes directes sobre els camps dels objectes. 

Per exemple, obtenir només els ítems d'una categoria específica:



```kotlin
fun obtenirPerCategoria(categoria: String) {
    db.collection("items")
        .whereEqualTo("categoria", categoria)
        .get()
        .addOnSuccessListener { documents ->
            val llistaFiltrada = documents.toObjects(Item::class.java)
            actualitzarUI(llistaFiltrada)
        }
}
```

# 6. Integració amb GitHub

Si el projecte està a GitHub, tingues en compte:

Afegir google-services.json a .gitignore per evitar exposar credencials:

```bash
# Firebase config
/app/google-services.json
```


Si treballes en equip, cada desenvolupador hauria de descarregar el seu fitxer google-services.json des de Firebase i col·locar-lo a app/.
