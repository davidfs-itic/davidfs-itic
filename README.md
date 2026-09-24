# Documentació mòduls de programació - Institut TIC

[![Desplegament](https://github.com/davidfs-itic/davidfs-itic/actions/workflows/deploy.yml/badge.svg)](https://github.com/davidfs-itic/davidfs-itic/actions/workflows/deploy.yml)

Apunts dels mòduls de programació de l'Institut TIC, dirigits als estudiants dels cicles formatius de la família d'Informàtica. Cada tema combina l'explicació dels conceptes amb exemples de codi funcionals.

Lloc web: [https://davidfs-itic.github.io/davidfs-itic/](https://davidfs-itic.github.io/davidfs-itic/)

## Continguts

### Desenvolupament mòbil
- [Android](https://davidfs-itic.github.io/davidfs-itic/Android/): Kotlin, Activities, Fragments, layouts, RecyclerView, arquitectura, proves i Firebase.
- [Jetpack Compose](https://davidfs-itic.github.io/davidfs-itic/Android/Interficies/Jetpack_compose/): interfícies declaratives, estats, components, navegació i llistes amb LazyColumn.
- [Exemples de codi Android](https://davidfs-itic.github.io/davidfs-itic/Exemples/): repositoris amb projectes Android complets per consultar i provar.

### Videojocs
- [Unity](https://davidfs-itic.github.io/davidfs-itic/Unity/): desenvolupament de videojocs 2D i 3D amb C#: escenes, físiques, col·lisions, animacions i realitat virtual.

### Sistemes encastats
- [Arduino](https://davidfs-itic.github.io/davidfs-itic/Arduino/): instal·lació de l'entorn i programació de la placa.
- [IoT](https://davidfs-itic.github.io/davidfs-itic/IoT/): arquitectura IoT, IIoT i desplegament de serveis amb contenidors.

### Eines transversals
- [Docker](https://davidfs-itic.github.io/davidfs-itic/Transversal/docker/): contenidors, imatges i desplegament d'aplicacions.
- [Git i Android Studio](https://davidfs-itic.github.io/davidfs-itic/Android/Git_android_studio/): configuració inicial i flux de treball amb branques des d'Android Studio.

## Com executar-ho en local

Els apunts estan fets amb [MkDocs](https://www.mkdocs.org/) i el tema [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/). Per veure'ls en local:

```bash
# Crear i activar l'entorn virtual
python -m venv .venv
source .venv/bin/activate

# Instal·lar les dependències
pip install mkdocs mkdocs-material mkdocs-awesome-pages-plugin

# Servidor local amb recàrrega automàtica (http://127.0.0.1:8000)
mkdocs serve
```

Altres comandes útils:

```bash
# Generar el lloc a la carpeta site/
mkdocs build

# Generar el lloc amb validació estricta (detecta enllaços trencats)
mkdocs build --strict
```

### Desplegament

Cada push a la branca `main` executa el workflow de GitHub Actions (`.github/workflows/deploy.yml`), que genera el lloc i el publica a GitHub Pages.

## Estructura del repositori

```
.
├── docs/                    # Contingut dels apunts (Markdown)
│   ├── index.md             # Pàgina d'inici
│   ├── Android/             # Kotlin, Interficies, Arquitectura, Proves, Llibreries, Firebase...
│   ├── Unity/
│   ├── Arduino/
│   ├── IoT/
│   ├── Transversal/         # Docker
│   ├── Exemples/
│   └── stylesheets/         # CSS propi (extra.css)
├── mkdocs.yml               # Configuració del lloc i menú de navegació (nav)
└── .github/workflows/       # Desplegament automàtic a GitHub Pages
```

Quan s'afegeix un document nou, cal afegir-lo al `nav` de `mkdocs.yml` i a l'`index.md` de la seva carpeta.

## Recursos

- [MkDocs: guia de configuració](https://www.mkdocs.org/user-guide/configuration/)
- [Material for MkDocs: configuració](https://squidfunk.github.io/mkdocs-material/setup/)
- [Material for MkDocs: referència](https://squidfunk.github.io/mkdocs-material/reference/)
- [Guia i demostració de les funcionalitats de Material for MkDocs](https://albrittonanalytics.com/)
- [Cheatsheet interactiu de Git](https://ndpsoftware.com/git-cheatsheet.html#loc=index;)
