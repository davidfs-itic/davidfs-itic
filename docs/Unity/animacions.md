# Animacions

## Crear Animation Clips

Un **Animation Clip** és una seqüència de sprites que reproduïts en ordre creen l'efecte d'animació.

### Crear una animació des de sprites

1. Seleccionar els sprites que formen l'animació (a la finestra Project).
2. Arrossegar-los sobre l'objecte a la finestra Scene o Hierarchy.
3. Unity crea automàticament:
    - Un **Animation Clip** (`.anim`).
    - Un **Animator Controller** (`.controller`) assignat a l'objecte.

### Configurar la velocitat

Seleccionar l'Animation Clip → a la finestra **Animation** (Window → Animation → Animation):

- **Samples**: nombre de fotogrames per segon. Valors baixos = animació més lenta.
- Es poden arrossegar els sprites a la línia de temps per ajustar el ritme.

### Loop

Per defecte, les animacions es reprodueixen en bucle. Per desactivar-ho:

1. Seleccionar l'Animation Clip a la finestra Project.
2. A l'Inspector, desmarcar **Loop Time**.

## Animator Controller

L'**Animator Controller** és un diagrama d'estats que defineix quina animació es reprodueix en cada moment i com es transiciona entre elles.

### Obrir l'Animator

- Seleccionar l'objecte amb l'Animator → **Window → Animation → Animator**.
- Es mostra un diagrama amb els estats (cada estat = una animació).

### Elements del diagrama

| Element | Descripció |
|---------|------------|
| **Entry** | Punt d'entrada. Connecta automàticament a l'estat per defecte (taronja). |
| **Estat** (rectangle) | Cada estat representa una animació. L'estat taronja és el per defecte. |
| **Transició** (fletxa) | Connexió entre estats. Defineix quan es canvia d'animació. |
| **Any State** | Permet crear transicions que funcionen des de qualsevol estat. |

### Crear transicions

1. Clic dret sobre un estat → **Make Transition**.
2. Clicar sobre l'estat de destí.
3. Seleccionar la fletxa de transició per configurar-la a l'Inspector.

### Condicions de transició

Per controlar les transicions, s'utilitzen **paràmetres**:

1. A la finestra Animator, pestanya **Parameters** → clicar **+**.
2. Crear un paràmetre (tipus Bool, Int, Float o Trigger).
3. Seleccionar la transició → Inspector → **Conditions** → afegir el paràmetre.

Exemple amb un Bool `isRunning`:

- Transició Idle → Run: condició `isRunning = true`.
- Transició Run → Idle: condició `isRunning = false`.

### Configuració de transicions

Per a transicions immediates (sense *blending*), configurar a l'Inspector de la transició:

- **Has Exit Time**: **desmarcar** (no espera que l'animació acabi per transicionar).
- **Fixed Duration**: **desmarcar**.
- **Transition Duration**: **0**.

## Controlar animacions des de codi

```csharp
private Animator animator;

void Start()
{
    animator = GetComponent<Animator>();
}

void Update()
{
    float horizontal = Input.GetAxisRaw("Horizontal");

    // Activar/desactivar l'animació de córrer
    animator.SetBool("isRunning", horizontal != 0);
}
```

Mètodes disponibles:

| Mètode | Paràmetre | Ús |
|--------|-----------|-----|
| `SetBool("nom", valor)` | Bool | Estats on/off (córrer, saltar) |
| `SetFloat("nom", valor)` | Float | Velocitat, blend trees |
| `SetInteger("nom", valor)` | Int | Selecció d'estat per nombre |
| `SetTrigger("nom")` | Trigger | Accions puntuals (atacar, morir) |

## Flip del sprite

Per girar el personatge quan canvia de direcció, es modifica `localScale`:

```csharp
void Update()
{
    float horizontal = Input.GetAxisRaw("Horizontal");

    if (horizontal > 0)
    {
        transform.localScale = new Vector3(1, 1, 1);     // Mirant a la dreta
    }
    else if (horizontal < 0)
    {
        transform.localScale = new Vector3(-1, 1, 1);    // Mirant a l'esquerra
    }
}
```

!!! tip "Alternativa amb SpriteRenderer"
    També es pot usar `SpriteRenderer.flipX`:
    ```csharp
    GetComponent<SpriteRenderer>().flipX = horizontal < 0;
    ```
    Però `localScale` afecta tots els fills de l'objecte (útil si el personatge té fills com punts de tir).

## Events d'animació

Els **Animation Events** permeten cridar funcions des d'un moment concret d'una animació.

### Crear un event

1. Obrir la finestra **Animation** amb l'objecte seleccionat.
2. Seleccionar l'Animation Clip.
3. Posicionar el cursor de temps al fotograma desitjat.
4. Clicar la icona d'**Add Event** (sobre la línia de temps).
5. A l'Inspector de l'event, seleccionar la **funció** a cridar.

### Exemple: destruir objecte al final d'una animació

```csharp
// Aquesta funció es crida des d'un Animation Event
public void DestroyObject()
{
    Destroy(gameObject);
}
```

Casos d'ús:

- Destruir un projectil quan acaba l'animació d'explosió.
- Reproduir un so en un moment concret de l'animació.
- Activar un collider en el moment exacte d'un atac.
