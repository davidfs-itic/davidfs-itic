# El composable Slider

`Slider` permet a l'usuari triar un valor dins d'un rang continu, arrossegant un control sobre una barra. S'utilitza per a valors on interessa més la sensació de "quantitat relativa" (volum, brillantor, un preu màxim...) que introduir una xifra exacta amb el teclat.

Documentació oficial: https://developer.android.com/develop/ui/compose/components/slider

## 1. Ús bàsic

Igual que la resta de components d'entrada vists fins ara, `Slider` és un component controlat: rep el valor actual amb `value` i notifica els canvis amb `onValueChange`.

```kotlin
var volum by remember { mutableStateOf(0.5f) }

Slider(
    value = volum,
    onValueChange = { volum = it }
)
```

Per defecte, `value` es mou dins del rang `0f..1f`. Aquest valor es pot fer servir directament (per exemple, com a percentatge) o escalar-lo manualment segons les necessitats.

## 2. Rang de valors: `valueRange`

Quan el valor representa una magnitud amb un rang propi (per exemple, una edat entre 0 i 100), es pot indicar directament amb `valueRange`, sense necessitat d'escalar-lo manualment:

```kotlin
var edat by remember { mutableStateOf(18f) }

Slider(
    value = edat,
    onValueChange = { edat = it },
    valueRange = 0f..100f
)

Text("Edat: ${edat.toInt()} anys")
```

## 3. Passos discrets: `steps`

Per defecte, el `Slider` permet qualsevol valor continu dins del rang. Si interessa que l'usuari només pugui triar entre un nombre concret de valors intermedis (per exemple, una valoració de l'1 al 5), es fa servir el paràmetre `steps`, que indica **quantes marques intermèdies** hi ha entre el valor mínim i el màxim (sense comptar-los):

```kotlin
var valoracio by remember { mutableStateOf(3f) }

Slider(
    value = valoracio,
    onValueChange = { valoracio = it },
    valueRange = 1f..5f,
    steps = 3 // marques intermèdies: 2, 3, 4 (a més de 1 i 5)
)
```

## 4. `onValueChangeFinished`

`onValueChange` es crida contínuament mentre l'usuari arrossega el control, cosa que és útil per actualitzar la interfície en temps real, però pot no ser el moment adequat per llançar operacions costoses (com desar el valor a una base de dades o fer una petició de xarxa). Per a aquests casos, `Slider` ofereix `onValueChangeFinished`, que només es crida un cop quan l'usuari deixa anar el control:

```kotlin
Slider(
    value = volum,
    onValueChange = { volum = it },
    onValueChangeFinished = {
        // Es crida només un cop, en deixar anar el control
        guardarPreferenciaVolum(volum)
    }
)
```

## 5. `RangeSlider`: selecció d'un interval

Quan cal que l'usuari trii un **interval** en lloc d'un únic valor (per exemple, un rang de preus mínim i màxim), Compose ofereix `RangeSlider`, amb dos controls independents sobre la mateixa barra. El seu valor és un `ClosedFloatingPointRange<Float>` en lloc d'un simple `Float`:

```kotlin
var rangPreu by remember { mutableStateOf(20f..80f) }

RangeSlider(
    value = rangPreu,
    onValueChange = { rangPreu = it },
    valueRange = 0f..100f
)

Text("Entre ${rangPreu.start.toInt()}€ i ${rangPreu.endInclusive.toInt()}€")
```
