# Introducció al Testing en Android

## 1. Què són les proves de software?

Les **proves de software** (testing) són el procés de verificar que el codi funciona correctament. Consisteixen a executar el programa amb unes entrades concretes i comprovar que els resultats són els esperats.

!!! note "Per què són importants?"
    - **Detecten errors aviat**: és molt més barat corregir un error durant el desenvolupament que quan l'aplicació ja està publicada.
    - **Donen confiança per fer canvis**: si tens proves, pots modificar o refactoritzar codi sabent que no has trencat res.
    - **Documenten el comportament**: les proves actuen com a documentació viva de què ha de fer el codi.
    - **Milloren la qualitat**: obliguen a pensar en casos límit i situacions d'error.

## 2. Tipus de proves: la piràmide de testing

En Android, les proves s'organitzen en una **piràmide** amb tres nivells:

```
          /\
         /  \
        / UI \          ← Poques: lentes, costoses
       /------\
      /        \
     /Integració\       ← Algunes: verifiquen connexions
    /------------\
   /              \
  /   Unitàries    \    ← Moltes: ràpides, barates
 /------------------\
```

### Tests unitaris

- Proven **funcions o classes aïllades**, sense necessitat del dispositiu Android.
- S'executen a la **JVM local** (molt ràpids).
- Carpeta: `app/src/test/`
- Exemple: comprovar que una funció de validació retorna `true` o `false` correctament.

### Tests d'integració

- Proven la **interacció entre diversos components** (p.ex. un `ViewModel` que crida un `Repository`).
- Verifiquen que les peces **encaixen correctament** entre elles.
- Poden ser locals (`app/src/test/`) o instrumentats (`app/src/androidTest/`), segons si necessiten el framework d'Android.

### Tests d'UI (end-to-end)

- Proven la **interfície completa** des de la perspectiva de l'usuari: clicar botons, escriure text, verificar resultats a pantalla.
- Sempre **instrumentats**, s'executen en un **dispositiu o emulador** (més lents).
- Carpeta: `app/src/androidTest/`
- S'utilitza la llibreria **Espresso**.

!!! note "Diferència entre integració i UI"
    - **Integració**: comproven que els components **col·laboren bé** (ViewModel ↔ Repository, dades que flueixen entre capes). No es centren en l'aparença visual.
    - **UI**: comproven el que l'usuari **veu i fa** (escriure en un camp, clicar un botó, veure el resultat a pantalla).

!!! tip "Regla general"
    Escriu **molts tests unitaris** (ràpids i barats), **alguns tests d'integració** (verifiquen connexions entre components) i **pocs tests d'UI** (lents però necessaris per verificar la interfície).

## 3. TDD: Test-Driven Development

El **TDD** és una metodologia on **primer escrius les proves i després el codi**. Segueix el cicle:

1. **Red** — Escrius un test que falla (perquè el codi encara no existeix).
2. **Green** — Escrius el codi mínim perquè el test passi.
3. **Refactor** — Millores el codi mantenint els tests verds.

```
    ┌───────────┐
    │   RED     │ ← Escriure test (falla)
    │  (test)   │
    └─────┬─────┘
          │
          ▼
    ┌───────────┐
    │  GREEN    │ ← Escriure codi (passa)
    │  (codi)   │
    └─────┬─────┘
          │
          ▼
    ┌───────────┐
    │ REFACTOR  │ ← Millorar codi (segueix passant)
    │           │
    └─────┬─────┘
          │
          └──────→ Tornar a RED amb el següent test
```

!!! note "Per què TDD?"
    - T'obliga a pensar en **què** ha de fer el codi **abans** de pensar en **com**.
    - Garanteix que tot el codi té proves des del principi.
    - Evita escriure codi innecessari.

## 4. Estructura d'un projecte Android

Quan crees un projecte Android, ja tens les carpetes de proves creades:

```
app/
├── src/
│   ├── main/           ← Codi de l'aplicació
│   │   ├── java/
│   │   └── res/
│   ├── test/           ← Tests UNITARIS (JVM local)
│   │   └── java/
│   │       └── com/example/app/
│   │           └── ExampleUnitTest.kt
│   └── androidTest/    ← Tests INSTRUMENTATS/UI (dispositiu)
│       └── java/
│           └── com/example/app/
│               └── ExampleInstrumentedTest.kt
```

## 5. Dependències necessàries

Els projectes Android nous ja inclouen les dependències bàsiques de testing al fitxer `build.gradle.kts` del mòdul `app`:

```kotlin
dependencies {
    // Tests unitaris
    testImplementation("junit:junit:4.13.2")

    // Tests instrumentats (UI)
    androidTestImplementation("androidx.test.ext:junit:1.2.1")
    androidTestImplementation("androidx.test.espresso:espresso-core:3.6.1")
}
```

!!! tip "No cal afegir res!"
    Si has creat el projecte amb Android Studio, aquestes dependències ja hi són. Només cal que comencis a escriure proves.

## 6. Següents passos

- [Disseny de proves (TDD)](disseny.md) — Com planificar i dissenyar proves abans de programar.
- [Tests unitaris](unitaris.md) — Com escriure tests unitaris amb JUnit.
- [Tests d'integració (UI)](ui.md) — Com escriure tests d'interfície amb Espresso.

## 7. Referències:

https://www.youtube.com/watch?v=_XYHr_dPjlE

https://www.youtube.com/watch?v=_XYHr_dPjlE
