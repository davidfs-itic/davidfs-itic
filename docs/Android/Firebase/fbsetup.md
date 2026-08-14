# Configuració de Firebase a Android

Abans d'utilitzar qualsevol servei de Firebase (Firestore, Authentication, Storage, etc.) cal enllaçar el projecte Android amb un projecte de Firebase. Aquest procés es fa un sol cop per projecte i és un requisit previ a tots els altres documents d'aquesta secció.

Documentació oficial: [Add Firebase to your Android project](https://firebase.google.com/docs/android/setup)

## 1. Crear un projecte a la consola de Firebase

1. Accedeix a [Firebase Console](https://console.firebase.google.com/).
2. Fes clic a "Add project" (Afegir projecte).
3. Introdueix el nom del projecte.
4. (Opcional) Pots activar Google Analytics, però no és necessari per començar.
5. Fes clic a "Create project" i espera que Firebase finalitzi la creació del projecte.

## 2. Afegir una aplicació Android al projecte Firebase

1. A la vista del projecte a Firebase, fes clic a la icona d'Android ("Add app").
2. Introdueix el nom del paquet de la teva app Android (exactament el que tens a `AndroidManifest.xml`).
3. (Opcional) Pots afegir un nickname de l'aplicació i el SHA-1 del certificat de depuració, si més endavant vols usar autenticació o notificacions.
4. Fes clic a "Register app".

## 3. Descarregar el fitxer google-services.json

1. Després de registrar l'app, Firebase et donarà un fitxer `google-services.json`.
2. Descarrega aquest fitxer.
3. Col·loca'l a la carpeta `app/` del teu projecte Android (no a la carpeta arrel del projecte).

!!! warning "Credencials sensibles"
    No afegeixis aquest fitxer al repositori públic de GitHub si conté credencials sensibles. Si el repositori és públic, considera ignorar-lo amb `.gitignore`.

## 4. Configuració del projecte a Android Studio

Firebase utilitza el plugin de Google Services. Hauràs d'afegir-lo al projecte.

### Pas 1: Afegir els plugins a build.gradle.kts (Project-level)
```kotlin
plugins {
    id("com.android.application") version "8.1.0" apply false
    id("com.google.gms.google-services") version "4.4.0" apply false
}
```
### Pas 2: Aplicar el plugin a build.gradle.kts (App-level)

A `app/build.gradle.kts`:

```kotlin
plugins {
    id("com.android.application")
    id("com.google.gms.google-services")  // Firebase plugin
}
```

### Pas 3: Afegir dependències específiques (opcional ara)

Quan utilitzem algun dels serveis concrets de firebase, afegirem algunes dependències. Això estarà explicat en el capítol corresponent al servei.

## 5. Sincronització del projecte amb Gradle

A Android Studio, fes clic a "Sync Now" quan t'aparegui la notificació.

Assegura't que no hi ha errors. Si hi ha problemes amb repositoris o versions, comprova que `google()` està inclòs.

## 6. Integració amb GitHub

Si el projecte està a GitHub, tingues en compte:

Afegir `google-services.json` a `.gitignore` per evitar exposar credencials:

```
# Firebase config
/app/google-services.json
```

Si treballes en equip, cada desenvolupador hauria de descarregar el seu fitxer `google-services.json` des de Firebase i col·locar-lo a `app/`.

El fitxer `build.gradle.kts` i la configuració del plugin es poden versionar sense problemes.

## 7. Verificar la integració

Tot i que encara no hem configurat cap servei concret, la millor manera de comprovar que el setup funciona és sincronitzar i compilar el projecte. Si no hi ha errors relacionats amb Firebase, la configuració és correcta.

## 8. Resum dels passos

1. Crear projecte a Firebase Console.
2. Afegir l'aplicació Android al projecte Firebase (package name, nickname opcional, SHA-1 opcional).
3. Descarregar el fitxer `google-services.json` i col·locar-lo a `app/`.
4. Afegir el plugin `com.google.gms.google-services` al `build.gradle.kts` de projecte i d'app.
5. Sincronitzar amb Gradle ("Sync Now") i verificar que no hi ha errors.
6. Afegir `google-services.json` al `.gitignore` si el repositori és públic.
7. Compilar el projecte per confirmar que la integració funciona.