# Prefabs i Instanciació

## Què és un Prefab?

Un **Prefab** és una plantilla reutilitzable d'un GameObject. Permet crear un objecte una vegada (amb tots els seus components, valors i fills) i reutilitzar-lo tantes vegades com calgui.

Exemples d'ús: bales, enemics, monedes, efectes de partícules.

## Crear un Prefab

1. Configurar el GameObject a l'escena amb tots els seus components (Sprite, Collider, Script, etc.).
2. Arrossegar l'objecte des de la **Hierarchy** a una carpeta d'Assets (ex: `Assets/Prefabs`).
3. L'objecte a la Hierarchy es torna **blau**, indicant que és una instància d'un Prefab.
4. Es pot eliminar l'objecte de la Hierarchy — el Prefab queda guardat als Assets.

## Editar un Prefab

- **Doble clic** sobre el Prefab a la finestra Project → s'obre el mode d'edició del Prefab.
- Fer els canvis necessaris.
- Clicar la **fletxa ←** (a la part superior de la Hierarchy) per sortir del mode d'edició.

!!! note "Instàncies vs Prefab"
    Els canvis al Prefab s'apliquen a totes les instàncies. Si es modifica una instància individualment, es crea un **override** (sobreescriptura) que només afecta aquella instància.

## Instantiate: crear objectes en temps d'execució

`Instantiate()` crea una còpia d'un Prefab durant el joc.

### Exemple bàsic

```csharp
public GameObject bulletPrefab;   // Assignar el Prefab des de l'Inspector
public Transform firePoint;       // Punt de tir (un GameObject fill buit)

void Disparar()
{
    Instantiate(bulletPrefab, firePoint.position, Quaternion.identity);
}
```

| Paràmetre | Descripció |
|-----------|------------|
| `bulletPrefab` | El Prefab a copiar. |
| `firePoint.position` | Posició on es crea la còpia. |
| `Quaternion.identity` | Sense rotació. |

### Accedir al component del nou objecte

```csharp
void Disparar()
{
    GameObject bala = Instantiate(bulletPrefab, firePoint.position, Quaternion.identity);
    bala.GetComponent<Bullet>().SetDirection(transform.localScale.x);
}
```

## Destroy: destruir objectes

```csharp
Destroy(gameObject);          // Destruir l'objecte actual
Destroy(gameObject, 3f);      // Destruir després de 3 segons
Destroy(otherObject);         // Destruir un altre objecte
```

!!! warning "gameObject vs this"
    `Destroy(gameObject)` destrueix el GameObject sencer (amb tots els seus components). `Destroy(this)` només destrueix el component script, no l'objecte.

## Exemple complet: sistema de bales

### Script de la bala

```csharp
public class Bullet : MonoBehaviour
{
    public float speed = 10f;
    private float direction = 1f;
    private Rigidbody2D rb;

    void Start()
    {
        rb = GetComponent<Rigidbody2D>();
        rb.velocity = new Vector2(speed * direction, 0);
    }

    public void SetDirection(float dir)
    {
        direction = dir;
        // Invertir l'sprite si va cap a l'esquerra
        if (dir < 0)
        {
            transform.localScale = new Vector3(-1, 1, 1);
        }
    }
}
```

### Script del jugador (disparar)

```csharp
public class PlayerShoot : MonoBehaviour
{
    public GameObject bulletPrefab;
    public Transform firePoint;
    public float cooldown = 0.5f;
    private float nextFireTime = 0f;

    void Update()
    {
        if (Input.GetKeyDown(KeyCode.F) && Time.time >= nextFireTime)
        {
            GameObject bala = Instantiate(bulletPrefab, firePoint.position, Quaternion.identity);
            bala.GetComponent<Bullet>().SetDirection(transform.localScale.x);
            nextFireTime = Time.time + cooldown;
        }
    }
}
```

### Configuració del Prefab de la bala

1. Crear un GameObject amb:
    - **Sprite Renderer** amb l'sprite de la bala.
    - **Rigidbody 2D** (Gravity Scale = 0).
    - **BoxCollider2D** o **CircleCollider2D**.
    - Script **Bullet**.
2. Arrossegar a `Assets/Prefabs`.
3. Al jugador, assignar el Prefab de la bala i el firePoint a l'Inspector.
