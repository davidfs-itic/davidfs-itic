# Llibreries

## Què són les llibreries?

Les llibreries són codi creat per altres desenvolupadors que podem reutilitzar als nostres projectes. Ens permeten:

- **Estalviar temps**: no cal reinventar la roda per a problemes ja resolts.
- **Codi provat i mantingut**: la comunitat s'encarrega de corregir errors i millorar el rendiment.
- **Solucions robustes**: aborden problemes comuns de manera fiable.

Alguns exemples de llibreries populars en Android:

| Llibreria | Funció |
|---|---|
| Retrofit | Accés a APIs i xarxa |
| MPAndroidChart | Gràfics i diagrames |
| Coil / Glide | Càrrega i cache d'imatges |
| Room | Base de dades local |

## Com afegir una llibreria al projecte

### Repositoris

Els repositoris són servidors on es publiquen les llibreries perquè altres desenvolupadors les puguin descarregar. Els més habituals són:

- **Google** — llibreries oficials d'Android i Google.
- **Maven Central** — el repositori més gran i utilitzat de l'ecosistema Java/Kotlin.
- **JitPack** — permet publicar qualsevol projecte de GitHub com a llibreria.

Es configuren al fitxer `settings.gradle.kts`, dins del bloc `dependencyResolutionManagement`:

```kotlin
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
        maven { url = uri("https://jitpack.io") }
    }
}
```

### Amb `build.gradle.kts` (mètode clàssic)

Per afegir una llibreria, cal incloure-la al bloc `dependencies` del fitxer `build.gradle.kts` **(Module: app)**:

```kotlin
dependencies {
    implementation("com.squareup.retrofit2:retrofit:2.11.0")
}
```

Després de modificar el fitxer, Android Studio mostrarà un avís **Sync Now** a la part superior. Cal fer clic per descarregar la llibreria.

!!! note "Conversió automàtica a TOML"
    Quan afegim una dependència amb aquest format, Android Studio pot mostrar un avís suggerint migrar-la al catàleg de versions (`libs.versions.toml`). Si acceptem, Android Studio mourà automàticament la versió i la definició de la llibreria al fitxer TOML i actualitzarà el `build.gradle.kts` per utilitzar la referència `libs.xxx`.

### Amb catàleg de versions — `libs.versions.toml` (mètode modern)

El fitxer `gradle/libs.versions.toml` permet centralitzar totes les versions i dependències en un sol lloc. Té tres blocs principals:

**`[versions]`** — Defineix les versions:

```toml
[versions]
retrofit = "2.11.0"
```

**`[libraries]`** — Defineix les llibreries amb el seu grup, nom i referència a la versió:

```toml
[libraries]
retrofit = { group = "com.squareup.retrofit2", name = "retrofit", version.ref = "retrofit" }
```

**`[plugins]`** — Defineix plugins de Gradle (opcional):

```toml
[plugins]
android-application = { id = "com.android.application", version.ref = "agp" }
```

Després, al fitxer `build.gradle.kts` **(Module: app)**, es referencia amb el prefix `libs.`:

```kotlin
dependencies {
    implementation(libs.retrofit)
}
```

!!! tip "Avantatge"
    Amb el catàleg de versions, totes les versions es mantenen en un sol fitxer. Això facilita les actualitzacions i evita inconsistències entre mòduls.

## Llibreries:

- [Retrofit](./retrofit.md)
- [Mp Android Chart](./mpandroidchart.md)
- [Glide](./glide.md)
