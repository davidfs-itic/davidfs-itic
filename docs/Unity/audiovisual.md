# Fons i So

## Backgrounds (fons)

### Importar i configurar

1. Importar la imatge del fons a `Assets/Sprites`.
2. Crear un GameObject amb **Sprite Renderer** i assignar-li la imatge.
3. Posicionar-lo darrere dels altres elements (Z positiva o Order in Layer baix).

### Order in Layer per als fons

Configurar l'**Order in Layer** del Sprite Renderer per assegurar que el fons queda darrere:

| Element | Order in Layer |
|---------|---------------|
| Fons llunyà | -20 |
| Fons proper | -10 |
| Tilemap terreny | 0 |
| Jugador | 1 |

### Organitzar amb GameObjects buits

Per mantenir la Hierarchy ordenada, agrupar els elements del fons dins de GameObjects buits:

```
Backgrounds (GameObject buit)
├── Sky
├── Mountains
└── Trees
```

Per crear un GameObject buit: Hierarchy → clic dret → **Create Empty**.

!!! tip "Repetir el fons"
    Si el fons no cobreix tot el nivell, duplicar-lo (Ctrl+D) i posicionar les còpies una al costat de l'altra.

## So

### Components de so a Unity

| Component | Funció |
|-----------|--------|
| **AudioListener** | "Orella" que capta el so. Per defecte està a la Main Camera. Només n'hi pot haver un a l'escena. |
| **AudioSource** | Component que reprodueix so. S'afegeix a qualsevol GameObject. |
| **AudioClip** | El fitxer de so (`.wav`, `.mp3`, `.ogg`). |

### Música de fons

1. Seleccionar la **Main Camera** (o crear un GameObject dedicat).
2. **Add Component → Audio Source**.
3. Assignar l'AudioClip de la música a la propietat **AudioClip**.
4. Configurar:

| Propietat | Valor |
|-----------|-------|
| **Play On Awake** | Marcat (comença a sonar automàticament) |
| **Loop** | Marcat (es repeteix) |
| **Volume** | Ajustar al gust (0 a 1) |

### Efectes de so des de codi

Per reproduir sons puntuals (tirs, salts, impactes):

```csharp
public class PlayerSounds : MonoBehaviour
{
    public AudioClip shootSound;
    public AudioClip jumpSound;

    private AudioSource audioSource;

    void Start()
    {
        audioSource = GetComponent<AudioSource>();
    }

    public void PlayShoot()
    {
        audioSource.PlayOneShot(shootSound);
    }

    public void PlayJump()
    {
        audioSource.PlayOneShot(jumpSound);
    }
}
```

- `PlayOneShot(clip)`: reprodueix un clip una vegada, sense interrompre altres sons que estiguin sonant.
- `Play()`: reprodueix el clip assignat al component AudioSource. Si ja s'estava reproduint, el reinicia.

!!! note "Volum"
    `PlayOneShot` accepta un segon paràmetre per ajustar el volum:
    ```csharp
    audioSource.PlayOneShot(shootSound, 0.5f);  // A la meitat de volum
    ```
