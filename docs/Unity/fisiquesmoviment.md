# Físiques i Moviment

## Components de físiques

### Rigidbody 2D

El **Rigidbody 2D** és el component que permet que un objecte sigui afectat per les físiques (gravetat, forces, col·lisions).

Per afegir-lo: seleccionar l'objecte → Inspector → **Add Component → Rigidbody 2D**.

| Body Type | Descripció |
|-----------|------------|
| **Dynamic** | Afectat per gravetat i forces. Per al jugador, enemics, projectils. |
| **Kinematic** | No afectat per forces externes, però es pot moure per codi. Per a plataformes mòbils. |
| **Static** | No es mou. Per al terreny i obstacles fixos (no cal afegir-lo, és el comportament per defecte dels colliders sense Rigidbody). |

Propietats importants:

- **Gravity Scale**: multiplicador de la gravetat (per defecte 1). Posar 0 per desactivar-la.
- **Constraints → Freeze Rotation Z**: marcar per evitar que l'objecte roti quan xoca. Imprescindible per a personatges 2D.

### Colliders 2D

Els **Colliders** defineixen la forma física de l'objecte per a les col·lisions.

| Collider | Forma | Ús típic |
|----------|-------|----------|
| **BoxCollider2D** | Rectangle | Caixes, plataformes, parets. |
| **CircleCollider2D** | Cercle | Monedes, boles, projectils. |
| **CapsuleCollider2D** | Càpsula | Personatges (s'adapta millor a la forma humana). |
| **PolygonCollider2D** | Polígon personalitzat | Formes irregulars. |

### Tilemap Collider 2D + Composite Collider 2D

Per afegir col·lisions al Tilemap:

1. Seleccionar el Tilemap a la Hierarchy.
2. **Add Component → Tilemap Collider 2D**.
    - Això crea un collider individual per cada tile (poc eficient).
3. **Add Component → Composite Collider 2D**.
    - Fusiona tots els colliders en un de sol (molt més eficient).
    - Al Tilemap Collider 2D, marcar **Used By Composite**.
4. Unity afegirà automàticament un **Rigidbody 2D**. Canviar el Body Type a **Static**.

## Moviment

### Moviment horitzontal amb velocity

```csharp
public class PlayerMovement : MonoBehaviour
{
    public float speed = 5f;

    private Rigidbody2D rb;

    void Start()
    {
        rb = GetComponent<Rigidbody2D>();
    }

    void FixedUpdate()
    {
        float horizontal = Input.GetAxisRaw("Horizontal");
        rb.velocity = new Vector2(horizontal * speed, rb.velocity.y);
    }
}
```

!!! warning "FixedUpdate vs Update"
    El moviment amb físiques (velocity, AddForce) s'ha de fer dins de **FixedUpdate**, que s'executa a intervals fixos. **Update** s'executa cada frame i pot variar segons el rendiment.

- `Input.GetAxisRaw("Horizontal")` retorna -1 (esquerra), 0 (quiet) o 1 (dreta).
- `rb.velocity.y` es manté per no interferir amb la gravetat/salt.

### Salt amb AddForce

```csharp
public float jumpForce = 10f;

void Update()
{
    if (Input.GetKeyDown(KeyCode.Space) && isGrounded)
    {
        rb.AddForce(Vector2.up * jumpForce, ForceMode2D.Impulse);
    }
}
```

- `ForceMode2D.Impulse`: aplica una força instantània (ideal per a salts).
- `ForceMode2D.Force`: aplica una força contínua (ideal per a propulsió).

### Variables públiques

Les variables `public` apareixen a l'Inspector de Unity, permetent ajustar els valors sense tocar el codi:

```csharp
public float speed = 5f;      // Velocitat de moviment
public float jumpForce = 10f;  // Força del salt
```

Això és molt útil per iterar i trobar els valors que fan que el joc se senti bé (*game feel*).

## Detecció de sòl amb Raycast

Per evitar que el jugador salti infinitament, cal detectar si està tocant el terra.

### Implementació

```csharp
public float rayLength = 1.5f;
public LayerMask groundLayer;

private bool isGrounded;

void FixedUpdate()
{
    // Llançar un raig cap avall des del centre del jugador
    RaycastHit2D hit = Physics2D.Raycast(
        transform.position,   // Origen
        Vector2.down,         // Direcció
        rayLength,            // Longitud
        groundLayer           // Capa a detectar
    );

    isGrounded = hit.collider != null;
}
```

### Debug.DrawRay

Per visualitzar el raycast a la finestra Scene (no es veu al joc):

```csharp
void FixedUpdate()
{
    Debug.DrawRay(transform.position, Vector2.down * rayLength, Color.red);
    // ... resta del codi
}
```

### Configuració important

!!! tip "Queries Start In Colliders"
    Si el raycast detecta el propi collider del jugador, anar a **Edit → Project Settings → Physics 2D** i desmarcar **Queries Start In Colliders**. Això evita que el raig detecti l'objecte des del qual s'origina.

### Layer Mask

Per assegurar que el raycast només detecta el terra:

1. Seleccionar els objectes del terra → Inspector → **Layer → Add Layer** → crear "Ground".
2. Assignar la capa "Ground" als objectes del terra.
3. Al script del jugador, la variable `groundLayer` permetrà seleccionar la capa des de l'Inspector.
