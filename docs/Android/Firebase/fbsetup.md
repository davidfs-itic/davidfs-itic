## 1. Crear un projecte a la consola de Firebase

1. Accedeix a Firebase Console
2. Fes clic a “Add project” (Afegir projecte).
3. Introdueix el nom del projecte.
4. (Opcional) Pots activar Google Analytics, però no és necessari per començar.
5. Fes clic a “Create project” i espera que Firebase finalitzi la creació del projecte.

## 2. Afegir una aplicació Android al projecte Firebase

1. A la vista del projecte a Firebase, fes clic a Android icon (“Add app”).

2. Introdueix el nom del paquet de la teva app Android (exactament el que tens a AndroidManifest.xml).

3. (Opcional) Pots afegir un nickname de l’aplicació i el SHA-1 del certificat de depuració, si més endavant vols usar autenticació o notificacions.

Fes clic a “Register app”.

## 3. Descarregar el fitxer google-services.json

1. Després de registrar l’app, Firebase et donarà un fitxer google-services.json.

2. Descarrega aquest fitxer.

3. Col·loca’l a la carpeta app/ del teu projecte Android (no a la carpeta arrel del projecte).

```
Important: No afegeixis aquest fitxer al repositori públic de GitHub si conté credencials sensibles. Si és públic, considera utilitzar [GitHub Secrets] o ignorar-lo amb .gitignore.
```

## 4. Configuració del projecte a Android Studio

Firebase utilitza el plugin de Google Services. Hauràs d’afegir-lo al projecte.

### Pas 1: Afegir els repositoris i dependències a build.gradle (Project-level)
```kotlin
buildscript {
    repositories {
        google()  // Necessari per Firebase
        mavenCentral()
    }
    dependencies {
        classpath "com.android.tools.build:gradle:8.1.0"  // Versió d’exemple
        classpath "com.google.gms:google-services:4.4.0"   // Plugin de Google Services
    }
}

allprojects {
    repositories {
        google()
        mavenCentral()
    }
}
```
### Pas 2: Aplicar el plugin a build.gradle (App-level)

A app/build.gradle:

```kotlin
plugins {
    id 'com.android.application'
    id 'com.google.gms.google-services'  // Firebase plugin
}
```

### Pas 3: Afegir dependències específiques (opcional ara)

Quan utilitzem algun dels serveis concrets de firebase, afegirem algunes dependències. Això estarà explicat en el capítol corresponent al servei.

## 5. Sincronització del projecte amb Gradle

A Android Studio, fes clic a “Sync Now” quan t’aparegui la notificació.

Assegura’t que no hi ha errors. Si hi ha problemes amb repositoris o versions, comprova que google() està inclòs.

## 6. Integració amb GitHub

Si el projecte està a GitHub, tingues en compte:

**Afegir google-services.json a .gitignore per evitar exposar credencials**

```
# Firebase config
/app/google-services.json
```

Si treballes en equip, cada desenvolupador hauria de descarregar el seu fitxer google-services.json des de Firebase i col·locar-lo a app/.

El fitxer build.gradle i la configuració del plugin es poden versionar sense problemes.

## 7. Verificar la integració

Tot i que encara no hem configurat cap servei concret, la millor manera de comprovar que el setup funciona és sincronitzar i compilar el projecte. Si no hi ha errors relacionats amb Firebase, la configuració és correcta.

## 8. Resum dels passos:

```
1. Crear projecte a Firebase Console
   ├─ Accedir a: https://console.firebase.google.com/
   ├─ "Add project"
   ├─ Nom del projecte
   └─ (Opcional) Activar Google Analytics

2. Afegir aplicació Android al projecte Firebase
   ├─ Cliquer sobre el logo Android "Add app"
   ├─ Introduir el nom del paquet (package name)
   ├─ (Opcional) Nickname i SHA-1
   └─ "Register app"

3. Descarregar fitxer google-services.json
   ├─ Guardar-lo a la carpeta: app/
   └─ ⚠️ No pujar-lo a repositoris públics

4. Configuració del projecte a Android Studio
   ├─ build.gradle (Project-level):
   │   ├─ Afegir repositoris: google(), mavenCentral()
   │   └─ Afegir classpath: com.google.gms:google-services
   ├─ build.gradle (App-level):
   │   └─ Aplicar plugin: id 'com.google.gms.google-services'
   └─ (Opcional) Afegir dependències Firebase

5. Sincronització amb Gradle
   └─ "Sync Now" → verificar que no hi hagi errors

6. Integració amb GitHub
   ├─ Afegir google-services.json a .gitignore
   ├─ Cada desenvolupador descarrega el seu google-services.json
   └─ Versionar build.gradle i plugins sense problemes

7. Verificació
   └─ Compilar projecte → si no hi ha errors, configuració correcta
```