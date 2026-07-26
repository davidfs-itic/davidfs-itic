# El composable DatePickerDialog

`DatePickerDialog` és el component de Material per triar una data des d'un calendari emergent. Combina un `DatePicker` (el calendari en si) amb l'estructura de botons de confirmació/cancel·lació, seguint el mateix patró que [AlertDialog](./alertdialog.md), del qual `DatePickerDialog` n'és de fet una variant especialitzada.

Documentació oficial: https://developer.android.com/develop/ui/compose/components/datepickers

## 1. Concepte: `DatePickerDialog` embolcalla un `DatePicker`

`DatePickerDialog` no dibuixa el calendari per si mateix: és un contenidor, amb la mateixa forma que `AlertDialog` (`onDismissRequest`, `confirmButton`, `dismissButton`), dins del qual es col·loca un `DatePicker` com a contingut:

```kotlin
DatePickerDialog(
    onDismissRequest = { /* tancar sense triar */ },
    confirmButton = {
        TextButton(onClick = { /* acceptar data triada */ }) {
            Text("D'acord")
        }
    },
    dismissButton = {
        TextButton(onClick = { /* tancar sense triar */ }) {
            Text("Cancel·lar")
        }
    }
) {
    DatePicker(state = /* ... */)
}
```

Igual que a [AlertDialog](./alertdialog.md#3-confirmbutton-i-dismissbutton), els botons no reben text directament sinó una lambda `@Composable` amb el botó ja construït, i cadascun és responsable de tancar el diàleg des del seu propi `onClick`.

## 2. Estat del calendari: `rememberDatePickerState`

El `DatePicker` necessita un estat propi, `DatePickerState`, creat amb `rememberDatePickerState`. Aquest estat exposa la data seleccionada a `selectedDateMillis`, en mil·lisegons des de l'època Unix (el mateix format que fa servir Java/Kotlin per a dates):

```kotlin
val datePickerState = rememberDatePickerState()

DatePickerDialog(
    onDismissRequest = { /* ... */ },
    confirmButton = { /* ... */ }
) {
    DatePicker(state = datePickerState)
}

// datePickerState.selectedDateMillis: Long? — null si encara no s'ha triat cap data
```

`selectedDateMillis` és **nullable**: val `null` fins que l'usuari toca un dia al calendari. Per això, en llegir aquest valor (normalment dins del `confirmButton`), cal comprovar-lo abans d'utilitzar-lo.

## 3. Mostrar i amagar el diàleg

Com qualsevol diàleg, `DatePickerDialog` només s'ha d'incloure a la composició quan toca mostrar-se, controlat per una variable d'estat booleana (vegeu [Estats en Jetpack Compose](./estats.md)):

```kotlin
var mostrarCalendari by remember { mutableStateOf(false) }
var dataSeleccionada by remember { mutableStateOf<Long?>(null) }

Button(onClick = { mostrarCalendari = true }) {
    Text("Triar data")
}

if (mostrarCalendari) {
    val datePickerState = rememberDatePickerState()

    DatePickerDialog(
        onDismissRequest = { mostrarCalendari = false },
        confirmButton = {
            TextButton(onClick = {
                dataSeleccionada = datePickerState.selectedDateMillis
                mostrarCalendari = false
            }) {
                Text("D'acord")
            }
        },
        dismissButton = {
            TextButton(onClick = { mostrarCalendari = false }) {
                Text("Cancel·lar")
            }
        }
    ) {
        DatePicker(state = datePickerState)
    }
}
```

## 4. Convertir els mil·lisegons a una data llegible

`selectedDateMillis` no és directament una data que es pugui mostrar a l'usuari: cal convertir-la, per exemple amb `Instant` i `LocalDate` del paquet `java.time`:

```kotlin
fun formatarData(millis: Long): String {
    val data = Instant.ofEpochMilli(millis)
        .atZone(ZoneOffset.UTC)
        .toLocalDate()
    return data.format(DateTimeFormatter.ofPattern("dd/MM/yyyy"))
}
```

!!! warning "Zona horària"
    El `DatePicker` treballa internament en **UTC**, no en la zona horària del dispositiu. Per evitar que una data es mostri amb un dia de diferència segons la zona horària de l'usuari, cal fer servir `ZoneOffset.UTC` (no `ZoneId.systemDefault()`) en convertir `selectedDateMillis` a `LocalDate`.

## 5. Exemple complet

```kotlin
@Composable
fun SelectorDeData() {
    var mostrarCalendari by remember { mutableStateOf(false) }
    var dataSeleccionada by remember { mutableStateOf<Long?>(null) }

    Column(modifier = Modifier.padding(16.dp)) {
        Button(onClick = { mostrarCalendari = true }) {
            Text("Triar data")
        }

        dataSeleccionada?.let { millis ->
            Text("Data triada: ${formatarData(millis)}")
        }
    }

    if (mostrarCalendari) {
        val datePickerState = rememberDatePickerState()

        DatePickerDialog(
            onDismissRequest = { mostrarCalendari = false },
            confirmButton = {
                TextButton(
                    onClick = {
                        dataSeleccionada = datePickerState.selectedDateMillis
                        mostrarCalendari = false
                    },
                    enabled = datePickerState.selectedDateMillis != null
                ) {
                    Text("D'acord")
                }
            },
            dismissButton = {
                TextButton(onClick = { mostrarCalendari = false }) {
                    Text("Cancel·lar")
                }
            }
        ) {
            DatePicker(state = datePickerState)
        }
    }
}
```

Aquí el `confirmButton` es desactiva (`enabled = false`) mentre no s'ha triat cap data, evitant que l'usuari pugui confirmar una selecció buida (`selectedDateMillis == null`).
