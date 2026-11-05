# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Descripció
Lloc web de documentació educativa (MkDocs Material) per a l'Institut TIC. Conté apunts d'Android, Unity, IoT, Arduino i temes transversals.

## Comandes de desenvolupament

```bash
# Activar entorn virtual
source .venv/bin/activate

# Instal·lar dependències
pip install mkdocs mkdocs-material mkdocs-awesome-pages-plugin

# Servidor local amb recàrrega automàtica (http://127.0.0.1:8000)
mkdocs serve

# Construir el site (genera ./site/)
mkdocs build

# Construir amb validació estricta (detecta enllaços trencats, etc.)
mkdocs build --strict
```

## Desplegament
Push a `main` activa automàticament el workflow de GitHub Actions (`.github/workflows/deploy.yml`) que construeix i desplega a GitHub Pages.

## Idioma i estil
- Escriure sempre en **català**. Els termes tècnics es mantenen en anglès (ViewModel, Fragment, LiveData, RecyclerView, etc.).
- To pedagògic, clar i directe, dirigit a estudiants.
- No utilitzar emojis.
- Abans de crear un document nou, llegeix 3-4 documents existents de la mateixa carpeta com a referència d'estil

## Estructura dels documents .md
- Els noms d'arxiu amb minúscules sense guions ni underscores.
- Un sol `# Títol` (H1) per fitxer, a la primera línia.
- Seccions numerades amb `## 1. Nom de la secció` (H2).
- Subseccions amb `### Subsecció` (H3) i `#### Subsubsecció` (H4) quan calgui.
- Començar amb una breu definició o explicació del concepte.
- Incloure un enllaç a la documentació oficial quan existeixi.
- Llistes amb `-` (guions).

## Codi
- Blocs de codi amb syntax highlighting: ` ```kotlin `, ` ```xml `, ` ```java `, etc.
- Comentaris dins del codi en català o anglès (segons conveniència).
- Exemples pràctics i funcionals, no pseudocodi.

## Navegació
- Quan es crea un document nou, cal:
  1. Afegir l'entrada corresponent a `mkdocs.yml` dins la secció `nav:`.
  2. Afegir un enllaç al fitxer `index.md` de la carpeta on s'estigui treballant.
  3. Examinar si hi ha un `index.md` a la carpeta pare, i si aquest és detallat o genèric. Si és detallat, afegir l'enllaç també. (preguntar en cas de dubte)
  4. Evitar duplicar explicacions entre documents: quan un concepte ja està explicat en un altre fitxer (p. ex. derivedStateOf, rememberCoroutineScope), enllaçar-hi amb un ancoratge (./fitxer.md#secció) en lloc de repetir-lo.

## Estructura de directoris
```
docs/
├── Android/
│   ├── Arquitectura/    # Patrons d'arquitectura (ViewModel, SOLID, DIP...)
│   ├── Firebase/        # Integració amb Firebase
│   ├── Interficies/     # Components UI (RecyclerView, Layouts, Fragments...)
│   ├── Kotlin/          # Fonaments del llenguatge Kotlin
│   ├── Llibreries/      # Llibreries externes (Retrofit, MPAndroidChart...)
│   └── Proves/          # Testing (unitaris, UI, TDD)
├── Unity/               # Desenvolupament amb Unity
├── IoT/                 # Internet of Things
├── Arduino/             # Arduino
└── Transversal/         # Docker, Git, etc.
```

## Notes addicionals
- Les imatges es guarden en subcarpetes `Imatges/` dins de cada secció.
- Utilitzar admonitions de MkDocs (`!!! info`, `!!! warning`, etc.) per destacar informació important.
- Referències creuades entre documents amb rutes relatives: `[text](./ruta/fitxer.md)`.
