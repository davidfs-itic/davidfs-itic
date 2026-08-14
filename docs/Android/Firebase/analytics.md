# Firebase Analytics

## 1. Què és Firebase Analytics?

**Firebase Analytics** és el servei gratuït de Firebase que recull dades d'ús de l'aplicació de forma automàtica i permet registrar esdeveniments personalitzats. Les dades es visualitzen a la consola de Firebase en forma d'informes: pantalles visitades, temps d'ús, esdeveniments, propietats d'usuari, etc.

A diferència de Firestore o Realtime Database, Analytics no serveix per emmagatzemar dades de l'aplicació, sinó per **entendre el comportament dels usuaris**: quines pantalles visiten més, quines accions completen, on abandonen un flux (per exemple, un procés de compra o de registre).

Documentació oficial: [Get started with Google Analytics for Android](https://firebase.google.com/docs/analytics/android/get-started)

## 2. Afegir la dependència

Al fitxer `app/build.gradle.kts`:

```kotlin
dependencies {
    // Firebase BoM (gestiona les versions automàticament)
    implementation(platform("com.google.firebase:firebase-bom:33.7.0"))
    // Analytics
    implementation("com.google.firebase:firebase-analytics")
}
```

Sincronitza el projecte amb **Sync Now**. No cal cap configuració addicional: un cop afegida la dependència, Analytics ja comença a recollir esdeveniments automàtics.

!!! note "Requisit previ"
    Cal tenir el projecte connectat amb Firebase, tal com s'explica a [Configuració de Firebase a Android](./fbsetup.md).

## 3. Esdeveniments automàtics

Firebase Analytics recull automàticament certs esdeveniments sense necessitat d'escriure cap línia de codi, entre d'altres:

- `first_open` — primera vegada que s'obre l'app.
- `session_start` — inici d'una sessió d'ús.
- `screen_view` — canvi de pantalla (si es fa servir Navigation Component o Activities/Fragments estàndard).
- `app_remove` — desinstal·lació de l'app.

Aquests esdeveniments són suficients per obtenir mètriques bàsiques (usuaris actius, retenció, durada de sessió) sense cap integració manual.

## 4. Registrar esdeveniments personalitzats

Per registrar accions concretes de l'usuari (afegir un ítem, completar una compra, fer login) cal obtenir la instància d'`FirebaseAnalytics` i cridar `logEvent`:

```kotlin
val analytics = Firebase.analytics

fun registrarAfegirItem(nomItem: String, categoria: String) {
    val params = Bundle().apply {
        putString(FirebaseAnalytics.Param.ITEM_NAME, nomItem)
        putString(FirebaseAnalytics.Param.ITEM_CATEGORY, categoria)
    }
    analytics.logEvent(FirebaseAnalytics.Event.SELECT_ITEM, params)
}
```

- `FirebaseAnalytics.Event` conté noms d'esdeveniments predefinits (`SELECT_ITEM`, `LOGIN`, `SEARCH`, `SHARE`...). Fer-los servir quan siguin aplicables permet aprofitar informes ja preparats a la consola de Firebase.
- `FirebaseAnalytics.Param` conté noms de paràmetres estàndard (`ITEM_NAME`, `ITEM_CATEGORY`, `VALUE`, `CURRENCY`...).

També es poden definir esdeveniments i paràmetres totalment personalitzats quan cap dels predefinits s'ajusti al cas d'ús:

```kotlin
fun registrarFiltreAplicat(categoria: String) {
    val params = Bundle().apply {
        putString("categoria_seleccionada", categoria)
    }
    analytics.logEvent("filtre_aplicat", params)
}
```

!!! warning "Noms d'esdeveniments i paràmetres"
    Els noms personalitzats han de tenir com a màxim 40 caràcters, començar amb una lletra i contenir només lletres, números i guions baixos (`_`). No es poden fer servir els prefixos reservats per Google (`firebase_`, `google_`, `ga_`).

## 5. Propietats d'usuari

Les propietats d'usuari permeten segmentar els informes segons característiques de l'usuari (per exemple, el seu nivell dins l'app o si té una subscripció activa):

```kotlin
analytics.setUserProperty("nivell_usuari", "avançat")
```

També es pot assignar un identificador d'usuari per relacionar l'activitat d'un mateix usuari entre dispositius (per exemple, després de fer login):

```kotlin
analytics.setUserId(usuariId)
```

!!! warning "Dades personals"
    No s'ha d'utilitzar `setUserId` ni les propietats d'usuari per emmagatzemar informació personal identificable (correu electrònic, nom complet, etc.). Fes servir sempre un identificador intern (per exemple, l'UID de Firebase Authentication).

## 6. Comprovar els esdeveniments amb DebugView

Els esdeveniments normals triguen fins a 24 hores a aparèixer als informes de la consola de Firebase, cosa que fa molt lenta la fase de proves. Per veure els esdeveniments en temps real cal activar el mode de depuració des d'una terminal:

```bash
adb shell setprop debug.firebase.analytics.app <nom_del_paquet>
```

A partir d'aquest moment, la secció **DebugView** de la consola de Firebase (Analytics → DebugView) mostra els esdeveniments del dispositiu connectat a mesura que es produeixen.

Per desactivar el mode de depuració:

```bash
adb shell setprop debug.firebase.analytics.app .none.
```
