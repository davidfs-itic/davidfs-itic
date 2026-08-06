# Càmera que segueix el jugador

## Càmera principal

En un projecte 2D, la càmera és **ortogràfica** (sense perspectiva).

### Configuració bàsica

Seleccionar **Main Camera** a la Hierarchy → Inspector:

| Propietat | Descripció |
|-----------|------------|
| **Projection** | Orthographic (per defecte en 2D). |
| **Size** | Mida de la càmera (meitat de l'alçada visible en unitats). Un valor més petit = més zoom. |
| **Position Z** | Ha de ser negativa (ex: -10) perquè la càmera estigui "davant" dels objectes. |

## Càmera follow

Per fer que la càmera segueixi el jugador:

```csharp
public class CameraFollow : MonoBehaviour
{
    public Transform player;

    void LateUpdate()
    {
        if (player != null)
        {
            transform.position = new Vector3(
                player.position.x,
                transform.position.y,   // Mantenir l'alçada de la càmera
                transform.position.z    // Mantenir la Z (-10)
            );
        }
    }
}
```

!!! tip "LateUpdate"
    El follow de càmera es fa a `LateUpdate` per assegurar que el jugador ja s'ha mogut (a `Update` o `FixedUpdate`) abans que la càmera actualitzi la seva posició. Això evita tremolors (*jitter*).

### Accedir a la càmera des de qualsevol script

```csharp
Camera cam = Camera.main;  // Referència a la càmera amb el tag "MainCamera"
```
