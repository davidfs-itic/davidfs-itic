# Jugador: Moviment i Salt

A [Tileset, Sprites i Col·lisions](scenesprites.md#components-de-fisiques) ja s'ha vist el **Rigidbody 2D** i els **Colliders 2D**. El jugador també necessita un Rigidbody 2D (Body Type **Dynamic**, amb **Freeze Rotation Z** marcat) i un Collider 2D (normalment **CapsuleCollider2D**) per poder-se moure i col·lidir amb el terreny.

## Crear el GameObject del jugador

1. Importar l'sprite (o sprite sheet) del jugador a `Assets/Sprites` (vegeu [Assets](assets.md) i [Sprite Sheets](scenesprites.md#sprite-sheets)).
2. **Hierarchy → clic dret → Create Empty** (o arrossegar l'sprite directament a l'escena, que ja crea l'objecte amb el **Sprite Renderer**).
3. Anomenar l'objecte "Player".
4. **Add Component → Rigidbody 2D**:
    - Body Type: **Dynamic**.
    - Constraints → **Freeze Rotation Z** marcat.
5. **Add Component → Capsule Collider 2D** i ajustar-ne la mida perquè encaixi amb l'sprite (Edit Collider a l'Inspector).
6. **Tag → Player** (Inspector, desplegable superior).
7. Posicionar el jugador al punt d'inici del nivell.

!!! warning "El tag Player"
    Cal assignar el tag **Player** al jugador. Més endavant, scripts com els dels enemics ([Enemics amb IA bàsica](enemics.md)) o dels objectes interactuables ([Objectes interactuables](interactius.md)) l'utilitzen (`CompareTag("Player")`, `FindWithTag("Player")`) per identificar-lo.

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
