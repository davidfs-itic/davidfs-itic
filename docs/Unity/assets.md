# Assets: importació i organització

Els **Assets** són tots els recursos del projecte: imatges, sons, scripts, animacions, prefabs...

## Importar assets al projecte

1. Descarregar els assets (textures, sprite sheets, sons).
2. Arrossegar els fitxers directament a la carpeta **Assets** dins la finestra Project.
3. Unity els importa automàticament i genera les metadades (fitxers `.meta`).

!!! warning "Important"
    No moure ni renombrar assets fora de Unity (des de l'explorador de fitxers). Fer-ho sempre des de la finestra **Project** per mantenir les referències.

## Organitzar els assets

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

## On trobar assets gratuïts

Per practicar no cal dibuixar ni compondre música pròpia. Hi ha bancs d'assets gratuïts (molts amb llicència *Creative Commons* o similar, cal comprovar sempre les condicions d'ús):

| Font | Contingut |
|------|-----------|
| [Kenney.nl](https://kenney.nl/assets) | Sprites, tilesets i sons per a jocs 2D, domini públic (CC0). |
| [itch.io/game-assets](https://itch.io/game-assets/free) | Sprite sheets, tilesets i efectes de so gratuïts d'artistes independents. |
| [OpenGameArt.org](https://opengameart.org/) | Art, música i efectes de so, llicències variades. |
| [Unity Asset Store](https://assetstore.unity.com/) | Assets oficials de Unity, alguns gratuïts, altres de pagament. |

!!! tip "Llicència"
    Abans d'usar un asset (fins i tot en un projecte de classe), comprovar la seva llicència. La majoria d'aquests bancs indiquen clarament si cal atribució o si l'ús comercial és permès.
