# Col·lisions i Triggers

## Colliders vs Triggers

Un **Collider** pot funcionar de dues maneres:

| Mode | Propietat `isTrigger` | Comportament |
|------|-----------------------|-------------|
| **Col·lisió física** | Desmarcat | Els objectes reboten, no es travessen. |
| **Trigger** | Marcat | Els objectes es travessen, però es detecta el contacte. |

Per activar el mode Trigger: seleccionar l'objecte → Inspector → Collider 2D → marcar **Is Trigger**.

!!! note "Requisit"
    Per detectar col·lisions o triggers, almenys un dels dos objectes ha de tenir un **Rigidbody 2D**.

## Col·lisions físiques: OnCollisionEnter2D

Es crida quan dos objectes amb colliders (sense isTrigger) xoquen.

```csharp
void OnCollisionEnter2D(Collision2D collision)
{
    Debug.Log("He xocat amb: " + collision.collider.name);
}
```

Funcions disponibles:

| Funció | Quan es crida |
|--------|--------------|
| `OnCollisionEnter2D` | En el moment del contacte |
| `OnCollisionStay2D` | Mentre es mantenen en contacte |
| `OnCollisionExit2D` | Quan deixen d'estar en contacte |

El paràmetre `Collision2D` conté informació sobre la col·lisió:

- `collision.collider`: el collider de l'altre objecte.
- `collision.gameObject`: el GameObject de l'altre objecte.
- `collision.contacts`: punts de contacte.

## Triggers: OnTriggerEnter2D

Es crida quan un objecte entra en un collider amb `isTrigger` activat.

```csharp
void OnTriggerEnter2D(Collider2D other)
{
    Debug.Log("Ha entrat: " + other.name);
}
```

| Funció | Quan es crida |
|--------|--------------|
| `OnTriggerEnter2D` | En el moment d'entrar |
| `OnTriggerStay2D` | Mentre es mantenen dins |
| `OnTriggerExit2D` | Quan surt del trigger |

!!! tip "Quan usar Trigger?"
    Triggers són ideals per a zones de detecció (àrea d'atac), recollir objectes (monedes, potions), zones que activen events (portes, trampes).

## Identificar amb què hem xocat

### Amb GetComponent

El patró més comú és intentar obtenir un component de l'objecte amb què hem xocat per identificar-lo:

```csharp
void OnTriggerEnter2D(Collider2D other)
{
    // Comprovar si hem tocat el jugador
    PlayerHealth player = other.GetComponent<PlayerHealth>();
    if (player != null)
    {
        player.Hit(1);  // Fer 1 de dany
    }
}
```

### Amb tags

Alternativament, es poden usar **Tags** per identificar objectes:

```csharp
void OnTriggerEnter2D(Collider2D other)
{
    if (other.CompareTag("Player"))
    {
        other.GetComponent<PlayerHealth>().Hit(1);
    }
}
```

Per assignar un tag: seleccionar l'objecte → Inspector → **Tag** → triar o crear un tag.

!!! warning "CompareTag vs ==  "
    Usar `CompareTag("Player")` en lloc de `other.tag == "Player"`. `CompareTag` és més eficient i no genera *garbage collection*.

## Exemple: bala que fa dany

```csharp
public class Bullet : MonoBehaviour
{
    public int damage = 1;

    void OnTriggerEnter2D(Collider2D other)
    {
        // No fer-se dany a un mateix (per tag o per component)
        if (other.CompareTag(gameObject.tag)) return;

        // Intentar fer dany
        Health health = other.GetComponent<Health>();
        if (health != null)
        {
            health.Hit(damage);
        }

        // Destruir la bala en qualsevol cas
        Destroy(gameObject);
    }
}
```

## Sistema de vida

### Component Health reutilitzable

```csharp
public class Health : MonoBehaviour
{
    public int maxHealth = 3;
    private int currentHealth;

    void Start()
    {
        currentHealth = maxHealth;
    }

    public void Hit(int damage)
    {
        currentHealth -= damage;
        Debug.Log(gameObject.name + " health: " + currentHealth);

        if (currentHealth <= 0)
        {
            Die();
        }
    }

    void Die()
    {
        Destroy(gameObject);
    }
}
```

Aquest component es pot afegir tant al jugador com als enemics — qualsevol objecte que pugui rebre dany.

!!! tip "Null check després de Destroy"
    Si un enemic dispara al jugador i el jugador és destruït, les referències al jugador es tornaran `null`. Cal fer comprovacions:
    ```csharp
    if (player != null)
    {
        // El jugador encara existeix
    }
    ```
