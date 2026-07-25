# El composable TextField

`TextField` és el composable que permet a l'usuari introduir i editar text. Equival a l'`EditText` del sistema de Views, però amb una diferència important de disseny: a Compose, `TextField` és un component **controlat** (*controlled component*), és a dir, no guarda ell mateix el text que mostra, sinó que sempre el rep de fora i notifica els canvis cap enfora.

Documentació oficial: https://developer.android.com/develop/ui/compose/text/user-input

## 1. Ús bàsic: component controlat

`TextField` necessita sempre dos paràmetres imprescindibles: `value` (el text actual a mostrar) i `onValueChange` (què fer quan l'usuari escriu). Si no s'implementa `onValueChange` actualitzant `value`, el camp de text sembla "bloquejat": l'usuari prem tecles però la pantalla no canvia, exactament pel mateix motiu explicat a [Estats en Jetpack Compose](./estats.md): sense un estat observable, Compose no sap que ha de recompondre's.

```kotlin
@Composable
fun CampNom() {
    var nom by remember { mutableStateOf("") }

    TextField(
        value = nom,
        onValueChange = { nom = it }
    )
}
```

Aquest patró (una variable d'estat amunt, i el `TextField` que només mostra el valor i demana canvis) és el mateix *state hoisting* vist als apunts d'Estats: si un altre composable necessita conèixer el text introduït, n'hi ha prou de pujar `nom` un nivell més amunt i passar-lo com a paràmetre.

## 2. Paràmetres que són composables

Abans d'entrar en detall, val la pena fixar-se en una particularitat important: paràmetres com `label`, `placeholder`, `leadingIcon`, `trailingIcon`, `prefix`, `suffix` o `supportingText` **no accepten un `String`**, sinó una lambda `@Composable () -> Unit`. Per això, als exemples, sempre s'escriuen com `label = { Text("Nom") }` i mai com `label = "Nom"`.

Aquest disseny és deliberat: en comptes de limitar l'etiqueta o la icona a un simple text, `TextField` podem col·locar **qualsevol composable**. Això permet, per exemple, un `leadingIcon` que sigui una `Row` amb diverses icones, un `label` que canviï d'estil segons l'estat, o un `supportingText` que mostri alguna cosa més que un text pla, com es veu a l'exemple següent:

```kotlin
var nom by remember { mutableStateOf("") }
val maxCaracters = 30

TextField(
    value = nom,
    onValueChange = { if (it.length <= maxCaracters) nom = it },
    label = { Text("Nom") },
    supportingText = {
        Row(
            modifier = Modifier.fillMaxWidth(),
            horizontalArrangement = Arrangement.SpaceBetween
        ) {
            Text("Nom complet")
            Text("${nom.length}/$maxCaracters")
        }
    }
)
```

Aquí `supportingText` ja no és un simple avís d'error: és una `Row` sencera que combina un text fix amb un comptador de caràcters que es recalcula a cada canvi de `nom`, gràcies al mateix mecanisme de recomposició explicat a [Estats en Jetpack Compose](./estats.md).

Els paràmetres composables més habituals de `TextField` són:

- **`label`**: etiqueta flotant que identifica el camp.
- **`placeholder`**: contingut que només es mostra quan el camp és buit.
- **`leadingIcon`** / **`trailingIcon`**: contingut a l'inici o al final del camp (normalment una `Icon`, però pot ser qualsevol composable).
- **`prefix`** / **`suffix`**: text o contingut fix enganxat a l'interior del camp, abans o després del text editable (per exemple, `prefix = { Text("+34") }` per a un número de telèfon, o `suffix = { Text("€") }` per a un import).
- **`supportingText`**: contingut auxiliar sota el camp.

## 3. Etiquetes, icones i text d'ajuda

`TextField` admet diversos paràmetres per fer el formulari més comprensible per a l'usuari:

```kotlin
TextField(
    value = nom,
    onValueChange = { nom = it },
    label = { Text("Nom") },
    placeholder = { Text("Introdueix el teu nom") },
    leadingIcon = { Icon(Icons.Default.Person, contentDescription = null) },
    trailingIcon = {
        if (nom.isNotEmpty()) {
            IconButton(onClick = { nom = "" }) {
                Icon(Icons.Default.Clear, contentDescription = "Esborrar")
            }
        }
    },
    supportingText = { Text("Aquest camp és obligatori") },
    singleLine = true
)
```

- **`label`**: etiqueta flotant que identifica el camp (es desplaça a sobre quan el camp té focus o contingut).
- **`placeholder`**: text d'exemple que només es veu quan el camp és buit.
- **`leadingIcon`** / **`trailingIcon`**: icones a l'inici o al final del camp; és habitual fer servir `trailingIcon` per a una icona d'esborrar o de mostrar/amagar contrasenya.
- **`supportingText`**: text auxiliar sota el camp, útil per a instruccions o missatges d'error.
- **`singleLine`**: força que el camp ocupi una sola línia (amb scroll horitzontal si cal), en lloc de créixer verticalment.

## 4. Validació i estat d'error

El paràmetre `isError` posa el `TextField` en un estil visual d'error (habitualment vora i etiqueta en vermell), sense que calgui gestionar manualment els colors:

```kotlin
var correu by remember { mutableStateOf("") }
val esCorreuInvalid = correu.isNotEmpty() && !correu.contains("@")

TextField(
    value = correu,
    onValueChange = { correu = it },
    label = { Text("Correu electrònic") },
    isError = esCorreuInvalid,
    supportingText = {
        if (esCorreuInvalid) Text("El correu no és vàlid")
    }
)
```

La validació en si (`esCorreuInvalid`) és lògica normal de Kotlin: no forma part del `TextField`, simplement es calcula a partir de l'estat i es fa servir per decidir com es mostra el component.

## 5. Tipus de teclat i accions del teclat

El paràmetre `keyboardOptions` permet indicar quin tipus de teclat mostrar (numèric, de correu, de contrasenya...) i quina acció apareix al botó del teclat (Fet, Següent, Cerca...):

```kotlin
TextField(
    value = correu,
    onValueChange = { correu = it },
    label = { Text("Correu electrònic") },
    keyboardOptions = KeyboardOptions(
        keyboardType = KeyboardType.Email,
        imeAction = ImeAction.Next
    ),
    keyboardActions = KeyboardActions(
        onNext = { focusManager.moveFocus(FocusDirection.Down) }
    )
)
```

`keyboardType` només canvia l'aspecte del teclat (per exemple, mostra el símbol `@` a l'abast a `KeyboardType.Email`), però **no valida res**: la validació del contingut sempre s'ha de fer a part, com a l'exemple anterior.

## 6. Camp de contrasenya

Per amagar el text introduït (per exemple, en un camp de contrasenya) es fa servir `visualTransformation`, que transforma com es *mostra* el text sense alterar el valor real que es guarda a l'estat:

* cal afegir la dependència :   implementation("androidx.compose.material:material-icons-extended")

```kotlin
var contrasenya by remember { mutableStateOf("") }
var mostrarContrasenya by remember { mutableStateOf(false) }

TextField(
    value = contrasenya,
    onValueChange = { contrasenya = it },
    label = { Text("Contrasenya") },
    visualTransformation = if (mostrarContrasenya) VisualTransformation.None else PasswordVisualTransformation(),
    keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Password),
    trailingIcon = {
        IconButton(onClick = { mostrarContrasenya = !mostrarContrasenya }) {
            Icon(
                imageVector = if (mostrarContrasenya) Icons.Default.VisibilityOff else Icons.Default.Visibility,
                contentDescription = "Mostrar o amagar contrasenya"
            )
        }
    }
)
```

## 7. `TextField` vs `OutlinedTextField`

Compose Material ofereix dues variants visuals amb exactament la mateixa API: `TextField` (fons ple, subratllat) i `OutlinedTextField` (vora dibuixada al voltant, sense fons). L'elecció és purament d'estil visual segons el disseny de la app; el comportament (`value`, `onValueChange`, `label`, etc.) és idèntic en tots dos.

```kotlin
OutlinedTextField(
    value = nom,
    onValueChange = { nom = it },
    label = { Text("Nom") }
)
```
