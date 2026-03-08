# Introducció a Unity

Referència principal del curs: [Tutorial COMPLETO Unity 2D desde Cero - JLPM](https://www.youtube.com/watch?v=GbmRt0wydQU)

## Què és Unity?

Unity és un motor de videojocs multiplataforma que permet crear jocs 2D i 3D, aplicacions interactives i simulacions. És un dels motors més utilitzats a la indústria, tant per studios independents com per grans empreses.

Característiques principals:

- **Multiplataforma**: exporta a PC, consoles, mòbils, web...
- **Llenguatge C#**: utilitza C# com a llenguatge de programació principal.
- **Asset Store**: botiga d'assets (models, textures, sons, scripts) gratuïts i de pagament.
- **Editor visual**: interfície gràfica completa per dissenyar escenes sense escriure codi.

## Instal·lació

### 1. Descarregar Unity Hub

Unity Hub és l'aplicació que gestiona les instal·lacions de Unity i els projectes. Es descarrega des de:

[https://unity.com/download](https://unity.com/download)

### 2. Plans de llicència

- **Personal**: gratuït per a estudiants i empreses amb ingressos inferiors a 100.000 $/any.
- **Plus/Pro**: plans de pagament amb funcionalitats addicionals.

Per a ús educatiu, el pla **Personal** és suficient.

### 3. Instal·lar una versió de Unity

Des de Unity Hub:

1. Anar a **Installs** → **Install Editor**.
2. Triar la versió **LTS** (Long Term Support) recomanada.
3. Seleccionar els mòduls necessaris (per 2D no calen mòduls addicionals).

## Unity Hub: gestió de projectes

Unity Hub permet:

- **Crear projectes nous**: escollint plantilla (2D, 3D, URP...).
- **Obrir projectes existents**.
- **Gestionar versions** de Unity instal·lades.
- **Accedir a tutorials i recursos** d'aprenentatge.

### Crear un projecte 2D

1. Obrir Unity Hub → **New Project**.
2. Seleccionar la plantilla **2D (Built-in Render Pipeline)**.
3. Posar un nom al projecte i triar la ubicació.
4. Clicar **Create project**.

## Primer contacte: la interfície

Quan obrim un projecte, Unity mostra diverses finestres:

| Finestra | Funció |
|----------|--------|
| **Scene** | Vista de disseny. Aquí muntem el nostre món. |
| **Game** | Vista del joc tal com el veurà el jugador. |
| **Hierarchy** | Llista d'objectes de l'escena actual (jerarquia pare-fill). |
| **Inspector** | Propietats i components de l'objecte seleccionat. |
| **Project** | Explorador de fitxers del projecte (Assets). |
| **Console** | Missatges de debug, avisos i errors. |

!!! tip "Consell"
    Es poden reorganitzar les finestres arrossegant-les. Si es desordenen, es pot restaurar el layout des de **Window → Layouts → Default**.

## Assets: importar textures i sons

Els **Assets** són tots els recursos del projecte: imatges, sons, scripts, animacions, prefabs...

### Importar assets al projecte

1. Descarregar els assets (textures, sprite sheets, sons).
2. Arrossegar els fitxers directament a la carpeta **Assets** dins la finestra Project.
3. Unity els importa automàticament i genera els metadades (fitxers `.meta`).

!!! warning "Important"
    No moure ni renombrar assets fora de Unity (des de l'explorador de fitxers). Fer-ho sempre des de la finestra **Project** per mantenir les referències.

### Organitzar els assets

Es recomana crear carpetes dins d'Assets per organitzar el projecte:

```
Assets/
├── Sprites/
├── Sounds/
├── Scripts/
├── Animations/
├── Prefabs/
└── Tiles/
```
