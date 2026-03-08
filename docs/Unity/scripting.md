# Scripting C# a Unity

## Estructura d'un script

Tots els scripts de Unity hereten de `MonoBehaviour`, la classe base que permet associar un script a un GameObject.

```csharp
using UnityEngine;

public class MeuScript : MonoBehaviour
{
    void Start()
    {
        // S'executa una vegada quan l'objecte s'activa
    }

    void Update()
    {
        // S'executa cada frame
    }
}
```

!!! warning "Nom del fitxer"
    El nom del fitxer `.cs` ha de coincidir exactament amb el nom de la classe. Si la classe es diu `PlayerMovement`, el fitxer ha de ser `PlayerMovement.cs`.

## Cicle de vida

Unity crida automàticament certes funcions en un ordre determinat:

| Funció | Quan s'executa | Ús |
|--------|---------------|-----|
| `Awake()` | Quan l'objecte es crea (abans de Start) | Inicialitzar referències internes |
| `Start()` | Abans del primer frame | Inicialització general |
| `Update()` | Cada frame | Lectura d'inputs, lògica del joc |
| `FixedUpdate()` | Cada pas de físiques (interval fix) | Moviment amb Rigidbody, forces |
| `LateUpdate()` | Després de tots els Update | Càmera follow |
| `OnDestroy()` | Quan l'objecte es destrueix | Neteja de recursos |

!!! tip "Update vs FixedUpdate"
    - **Update**: depèn del framerate (pot ser 30, 60, 144 vegades/segon). Usar per a inputs i lògica visual.
    - **FixedUpdate**: s'executa a intervals fixos (per defecte 50 vegades/segon). Usar per a tot el que impliqui físiques (`velocity`, `AddForce`).

## Variables

### Tipus bàsics

```csharp
int vides = 3;             // Nombre enter
float velocitat = 5.5f;    // Nombre decimal (la 'f' és obligatòria)
bool estaViu = true;       // Cert o fals
string nom = "Jugador";    // Text
```

### public vs private

```csharp
public float speed = 5f;    // Visible a l'Inspector, accessible des d'altres scripts
private int health = 100;   // No visible a l'Inspector, només accessible dins d'aquest script
```

- Les variables **public** apareixen a l'Inspector i es poden ajustar sense tocar el codi.
- Les variables **private** (per defecte si no s'indica res) queden ocultes.

!!! tip "SerializeField"
    Si vols que una variable privada aparegui a l'Inspector sense fer-la pública:
    ```csharp
    [SerializeField] private float speed = 5f;
    ```

## GetComponent

`GetComponent<T>()` permet accedir als components d'un GameObject des del codi.

```csharp
private Rigidbody2D rb;
private Animator animator;

void Start()
{
    rb = GetComponent<Rigidbody2D>();
    animator = GetComponent<Animator>();
}
```

!!! warning "Rendiment"
    No cridar `GetComponent` dins d'`Update()`. Guardar la referència a `Start()` i reutilitzar-la.

## Input

### Sistema d'input clàssic

```csharp
// Eixos (retorna valors entre -1 i 1)
float h = Input.GetAxisRaw("Horizontal");  // A/D o fletxes esquerra/dreta
float v = Input.GetAxisRaw("Vertical");    // W/S o fletxes amunt/avall

// Tecles
if (Input.GetKeyDown(KeyCode.Space))    // En el moment de prémer
if (Input.GetKey(KeyCode.Space))        // Mentre es manté premuda
if (Input.GetKeyUp(KeyCode.Space))      // En el moment de deixar anar
```

- `GetAxisRaw`: retorna -1, 0 o 1 (sense interpolació, resposta immediata).
- `GetAxis`: retorna valors suavitzats (amb acceleració/desacceleració).

!!! note "Nou Input System"
    Unity té un sistema d'input més modern (paquet **Input System**) que ofereix més flexibilitat i suport per a múltiples dispositius. El sistema clàssic (`Input.GetAxisRaw`) segueix funcionant però es considera *legacy*.

## Transform

Tot GameObject té un component `transform` accessible directament (sense `GetComponent`):

```csharp
// Posició
transform.position = new Vector3(0, 5, 0);
Vector3 pos = transform.position;

// Escala (útil per fer flip del sprite)
transform.localScale = new Vector3(-1, 1, 1);  // Invertir horitzontalment

// Rotació
transform.rotation = Quaternion.identity;  // Sense rotació
```

## Time.time per a cooldowns

`Time.time` retorna el temps en segons des que el joc ha començat. Es pot usar per gestionar cooldowns:

```csharp
public float cooldown = 0.5f;
private float nextFireTime = 0f;

void Update()
{
    if (Input.GetKeyDown(KeyCode.F) && Time.time >= nextFireTime)
    {
        Disparar();
        nextFireTime = Time.time + cooldown;
    }
}
```

## Debug.Log i la Consola

### Escriure a la consola

```csharp
Debug.Log("Missatge informatiu");          // Icona blanca
Debug.LogWarning("Atenció!");              // Icona groga
Debug.LogError("Alguna cosa ha fallat!");  // Icona vermella
```

### Llegir errors

Quan hi ha un error, la consola mostra:

1. **Missatge d'error**: descripció del problema.
2. **Nom del script i línia**: on s'ha produït l'error (clicable per anar directament al codi).
3. **Stack trace**: la cadena de crides que ha portat a l'error.

Errors comuns:

| Error | Causa | Solució |
|-------|-------|---------|
| `NullReferenceException` | S'intenta usar un objecte que és `null` | Comprovar que la referència existeix |
| `MissingComponentException` | `GetComponent` no troba el component | Verificar que l'objecte té el component |
| `IndexOutOfRangeException` | Accés a un índex fora dels límits d'un array | Comprovar la mida de l'array |

## Null checks

Abans d'usar una referència que podria ser `null`, cal comprovar-la:

```csharp
private Transform player;

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
    if (player != null)
    {
        // Segur d'usar player
        float distance = Vector2.Distance(transform.position, player.position);
    }
}
```

!!! warning "Objectes destruïts"
    A Unity, quan es destrueix un GameObject amb `Destroy()`, les referències a ell es tornen `null`. Sempre cal fer null check si l'objecte podria haver estat destruït.
