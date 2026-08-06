# Objectes interactuables

Un cop el jugador es mou i la càmera el segueix, cal omplir el nivell d'objectes amb els quals pugui interactuar: monedes per puntuar, power-ups que donen vida i trampes que fan mal. Tots segueixen el mateix patró: un **Collider 2D** amb **Is Trigger** marcat i un script que reacciona a `OnTriggerEnter2D` (vegeu [Col·lisions i Triggers](collisions.md)).

## Marcador en pantalla: Canvas i Text

Abans de fer les monedes, cal poder mostrar la puntuació en pantalla.

### Crear el Canvas

1. **Hierarchy → clic dret → UI → Text - TextMeshPro**.
2. Si és el primer cop, Unity demana importar **TMP Essentials**: clicar **Import**.
3. Això crea automàticament:
    - Un **Canvas** (l'element arrel de tota la UI).
    - Un **EventSystem** (gestiona la interacció amb la UI, encara que aquí no calgui).
    - El text, com a fill del Canvas.

### Configuració del Canvas

| Propietat | Valor | Explicació |
|-----------|-------|------------|
| **Render Mode** | Screen Space - Overlay | La UI es dibuixa per damunt de tota l'escena, independent de la càmera. |
| **UI Scale Mode** (component Canvas Scaler) | Scale With Screen Size | La UI s'adapta a diferents resolucions de pantalla. |

Posicionar el text amb el **Rect Transform** (per exemple, ancorat a la cantonada superior esquerra).

### Actualitzar el text des de codi

```csharp
using TMPro;
using UnityEngine;

public class ScoreManager : MonoBehaviour
{
    public static ScoreManager Instance;

    public TMP_Text scoreText;
    private int score = 0;

    void Awake()
    {
        Instance = this;
    }

    public void AddScore(int amount)
    {
        score += amount;
        scoreText.text = "Monedes: " + score;
    }
}
```

!!! note "Patró Singleton simple"
    `Instance` és una referència estàtica al `ScoreManager`, així qualsevol script (com la moneda) hi pot accedir amb `ScoreManager.Instance.AddScore(1)` sense necessitat de cercar-lo cada cop.

Cal afegir aquest script a un GameObject buit a l'escena (ex: "GameManager") i assignar el `scoreText` des de l'Inspector.

## Monedes: recollir i puntuar

### Configurar la moneda

| Component | Configuració |
|-----------|-------------|
| **Sprite Renderer** | Sprite de la moneda |
| **CircleCollider2D** | **Is Trigger** marcat |
| **Script Coin** | La lògica de recollida |

### Script de la moneda

```csharp
public class Coin : MonoBehaviour
{
    public int value = 1;

    void OnTriggerEnter2D(Collider2D other)
    {
        if (other.CompareTag("Player"))
        {
            ScoreManager.Instance.AddScore(value);
            Destroy(gameObject);
        }
    }
}
```

Un cop provada una moneda, convertir-la en [Prefab](prefabs.md) per poder-ne escampar còpies per tot el nivell.

## Vides i power-ups

Els power-ups de vida reutilitzen el component `Health` ja creat a [Col·lisions i Triggers](collisions.md#sistema-de-vida), afegint-hi un mètode per curar:

```csharp
// Dins de Health.cs
public void Heal(int amount)
{
    currentHealth = Mathf.Min(currentHealth + amount, maxHealth);
    Debug.Log(gameObject.name + " health: " + currentHealth);
}
```

Script del power-up:

```csharp
public class HealthPickup : MonoBehaviour
{
    public int healAmount = 1;

    void OnTriggerEnter2D(Collider2D other)
    {
        Health health = other.GetComponent<Health>();
        if (health != null)
        {
            health.Heal(healAmount);
            Destroy(gameObject);
        }
    }
}
```

!!! tip "GetComponent en lloc de CompareTag"
    Aquí es fa servir `GetComponent<Health>()` en lloc de comprovar el tag "Player", perquè així el power-up també funcionaria si algun dia un enemic el pogués recollir. Tots dos patrons són vàlids; cal triar el que encaixi amb el disseny del joc.

## Trampes

Una trampa (punxes, lava...) fa dany al jugador quan hi entra en contacte:

```csharp
public class Trap : MonoBehaviour
{
    public int damage = 1;

    void OnTriggerEnter2D(Collider2D other)
    {
        if (other.CompareTag("Player"))
        {
            other.GetComponent<Health>().Hit(damage);
        }
    }
}
```

| Component | Configuració |
|-----------|-------------|
| **Sprite Renderer** | Sprite de la trampa (punxes, etc.) |
| **Collider 2D** | **Is Trigger** marcat, ajustat a la forma perillosa |
| **Script Trap** | La lògica de dany |

!!! warning "Dany continu"
    Amb `OnTriggerEnter2D`, el dany només es rep un cop en entrar al trigger. Si el jugador es queda dins de la trampa i es vol dany repetit, cal usar `OnTriggerStay2D` combinat amb un cooldown (vegeu `Time.time` a [Scripting C# a Unity](scripting.md#timetime-per-a-cooldowns)) per evitar que rebi dany cada frame.
