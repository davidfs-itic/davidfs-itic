# Escena, Tiles i Sprites

## Sprite Sheets

Un **Sprite Sheet** és una imatge que conté múltiples sprites (fotogrames o elements) organitzats en una quadrícula. Són molt habituals en jocs 2D per a personatges, enemics, tiles del terreny, etc.

### Importar un Sprite Sheet

1. Arrossegar la imatge a la carpeta **Assets/Sprites** dins la finestra Project.
2. Seleccionar la imatge importada.
3. A l'**Inspector**, configurar les propietats d'importació.

### Configuració a l'Inspector

| Propietat | Valor | Explicació |
|-----------|-------|------------|
| **Sprite Mode** | Multiple | Indica que la imatge conté múltiples sprites. |
| **Pixels Per Unit** | 16 (o el que correspongui) | Quants píxels equivalen a 1 unitat de Unity. |
| **Filter Mode** | Point (no filter) | Per a pixel art, evita el suavitzat (*blur*). |
| **Compression** | None | Per a pixel art, evita artefactes de compressió. |

!!! tip "Point vs Bilinear"
    - **Point (no filter)**: manté els píxels nítids. Ideal per a **pixel art**.
    - **Bilinear**: suavitza la imatge. Millor per a gràfics d'alta resolució.

Després de canviar les propietats, clicar **Apply**.

## Sprite Editor

El Sprite Editor permet retallar (*slice*) un Sprite Sheet en sprites individuals.

1. Seleccionar la imatge → Inspector → clicar **Sprite Editor**.
2. Clicar **Slice** (a dalt a l'esquerra).
3. Triar el mètode de retall:

| Mètode | Ús |
|--------|-----|
| **Automatic** | Unity detecta els sprites automàticament pels seus contorns. |
| **Grid By Cell Size** | Retalla per mida de cel·la (ex: 16x16, 32x32). Ideal per sprite sheets uniformes. |
| **Grid By Cell Count** | Retalla per nombre de files i columnes. |

4. Clicar **Slice** i després **Apply** (a dalt a la dreta).

Un cop retallat, es pot desplegar la imatge a la finestra Project per veure tots els sprites individuals.

## Tile Palette

La **Tile Palette** és l'eina per pintar el món del joc amb tiles (rajoles), com si fos un programa de dibuix.

### Crear una Tile Palette

1. **Window → 2D → Tile Palette**.
2. Clicar **Create New Palette**.
3. Donar-li un nom (ex: "Terreny").
4. Triar la carpeta on guardar-la (ex: Assets/Tiles).
5. Arrossegar els sprites retallats a la paleta.

### Eines de la Tile Palette

| Eina | Funció |
|------|--------|
| **Brush** (B) | Pinta tiles individualment. |
| **Box Fill** (U) | Pinta un rectangle de tiles. |
| **Eraser** (D) | Esborra tiles. |
| **Picker** (I) | Selecciona un tile del mapa per pintar-ne més. |
| **Fill** (G) | Omple una àrea tancada. |

## Tilemap: crear el món

### Configurar el Grid i el Tilemap

1. A la **Hierarchy**, clic dret → **2D Object → Tilemap → Rectangular**.
2. Això crea:
    - Un objecte **Grid** (pare): defineix la quadrícula.
    - Un objecte **Tilemap** (fill): conté els tiles pintats.

3. Seleccionar el Tilemap a la Hierarchy.
4. A la Tile Palette, triar un tile i començar a pintar a la finestra Scene.

### Múltiples Tilemaps

Es poden crear diversos Tilemaps dins del mateix Grid per separar capes:

- **Tilemap_Ground**: el terra i les plataformes.
- **Tilemap_Background**: decoració del fons.
- **Tilemap_Foreground**: elements per davant del jugador.

## Order in Layer

L'**Order in Layer** determina l'ordre de dibuix dels elements 2D. Un valor més alt es dibuixa per davant.

Es configura al component **Sprite Renderer** o **Tilemap Renderer** a l'Inspector.

| Order in Layer | Ús típic |
|----------------|----------|
| -10 | Fons llunyà |
| 0 | Terreny / plataformes |
| 1 | Jugador |
| 2 | Efectes / partícules |
| 10 | UI del món |

!!! note "Sorting Layers"
    A més de l'Order in Layer, Unity permet crear **Sorting Layers** personalitzats (Edit → Project Settings → Tags and Layers) per organitzar millor les capes de renderitzat.
