# Enemics amb IA bàsica

## Configurar l'enemic

Un enemic bàsic en un joc 2D de plataformes necessita:

| Component | Configuració |
|-----------|-------------|
| **Sprite Renderer** | Sprite de l'enemic |
| **Rigidbody 2D** | Dynamic, Freeze Rotation Z |
| **CapsuleCollider2D** | Ajustat a la forma de l'enemic |
| **Animator** | Amb les animacions de l'enemic |
| **Script Enemy** | La lògica de la IA |
| **Health** | Component de vida (reutilitzable) |

## Orientar-se cap al jugador

### Obtenir la referència al jugador

```csharp
public class Enemy : MonoBehaviour
{
    private Transform player;

    void Start()
    {
        GameObject playerObj = GameObject.FindWithTag("Player");
        if (playerObj != null)
        {
            player = playerObj.transform;
        }
    }
}
```

!!! warning "Assignar el tag"
    Cal assegurar-se que el jugador té assignat el tag **Player** (Inspector → Tag → Player).

### Calcular la direcció

El vector de direcció entre dos objectes és la diferència de les seves posicions:

```csharp
Vector2 direction = player.position - transform.position;
```

- Si `direction.x > 0` → el jugador és a la **dreta**.
- Si `direction.x < 0` → el jugador és a l'**esquerra**.

### Flip amb localScale

```csharp
void Update()
{
    if (player == null) return;

    // Mirar cap al jugador
    if (player.position.x < transform.position.x)
    {
        transform.localScale = new Vector3(-1, 1, 1);  // Mirar a l'esquerra
    }
    else
    {
        transform.localScale = new Vector3(1, 1, 1);   // Mirar a la dreta
    }
}
```

## Detectar distància al jugador

Utilitzem `Mathf.Abs()` per obtenir la distància horitzontal (valor absolut):

```csharp
public float detectionRange = 10f;
public float shootRange = 5f;

void Update()
{
    if (player == null) return;

    float distance = Mathf.Abs(player.position.x - transform.position.x);

    if (distance < detectionRange)
    {
        // El jugador és a prop — mirar-lo
        LookAtPlayer();
    }

    if (distance < shootRange)
    {
        // El jugador és dins del rang de tir
        TryShoot();
    }
}
```

## Disparar

Reutilitzem el sistema de Prefabs i Instantiate de la secció [Prefabs i Instanciació](prefabs.md):

```csharp
public GameObject bulletPrefab;
public Transform firePoint;
public float shootCooldown = 2f;
private float nextShootTime = 0f;

void TryShoot()
{
    if (Time.time >= nextShootTime)
    {
        Shoot();
        nextShootTime = Time.time + shootCooldown;
    }
}

void Shoot()
{
    GameObject bala = Instantiate(bulletPrefab, firePoint.position, Quaternion.identity);
    float direction = transform.localScale.x;
    bala.GetComponent<Bullet>().SetDirection(direction);
}
```

## Script complet de l'enemic

```csharp
public class Enemy : MonoBehaviour
{
    public float detectionRange = 10f;
    public float shootRange = 5f;
    public float shootCooldown = 2f;
    public GameObject bulletPrefab;
    public Transform firePoint;

    private Transform player;
    private float nextShootTime = 0f;

    void Start()
    {
        GameObject playerObj = GameObject.FindWithTag("Player");
        if (playerObj != null)
        {
            player = playerObj.transform;
        }
    }

    void Update()
    {
        if (player == null) return;

        float distance = Mathf.Abs(player.position.x - transform.position.x);

        if (distance < detectionRange)
        {
            LookAtPlayer();
        }

        if (distance < shootRange && Time.time >= nextShootTime)
        {
            Shoot();
            nextShootTime = Time.time + shootCooldown;
        }
    }

    void LookAtPlayer()
    {
        if (player.position.x < transform.position.x)
        {
            transform.localScale = new Vector3(-1, 1, 1);
        }
        else
        {
            transform.localScale = new Vector3(1, 1, 1);
        }
    }

    void Shoot()
    {
        GameObject bala = Instantiate(bulletPrefab, firePoint.position, Quaternion.identity);
        bala.GetComponent<Bullet>().SetDirection(transform.localScale.x);
    }
}
```

## Patrons addicionals

### Patrulla

Un enemic que es mou entre dos punts:

```csharp
public class EnemyPatrol : MonoBehaviour
{
    public Transform pointA;
    public Transform pointB;
    public float speed = 2f;

    private Transform target;
    private Rigidbody2D rb;

    void Start()
    {
        rb = GetComponent<Rigidbody2D>();
        target = pointB;
    }

    void FixedUpdate()
    {
        // Moure's cap al punt objectiu
        Vector2 direction = (target.position - transform.position).normalized;
        rb.velocity = new Vector2(direction.x * speed, rb.velocity.y);

        // Canviar de punt quan s'arriba a prop
        if (Vector2.Distance(transform.position, target.position) < 0.5f)
        {
            target = (target == pointA) ? pointB : pointA;
        }

        // Flip
        transform.localScale = new Vector3(
            direction.x > 0 ? 1 : -1, 1, 1
        );
    }
}
```

!!! tip "Points A i B"
    Es poden crear GameObjects buits com a fills de l'enemic o com a objectes independents a l'escena, i assignar-los des de l'Inspector.

### Persecució

Un enemic que segueix el jugador quan està a prop:

```csharp
public float chaseRange = 5f;
public float chaseSpeed = 3f;

void FixedUpdate()
{
    if (player == null) return;

    float distance = Vector2.Distance(transform.position, player.position);

    if (distance < chaseRange)
    {
        Vector2 direction = (player.position - transform.position).normalized;
        rb.velocity = new Vector2(direction.x * chaseSpeed, rb.velocity.y);
    }
    else
    {
        rb.velocity = new Vector2(0, rb.velocity.y);
    }
}
```
