# `minSdk`, `targetSdk` i `compileSdk` a Android

## Resum

En un projecte Android hi ha tres versions de l'SDK que tenen funcions diferents:

| Propietat | Què indica? | Quan afecta? |
|-----------|-------------|--------------|
| `minSdk` | La versió mínima d'Android que pot instal·lar l'aplicació. | Instal·lació i execució |
| `targetSdk` | La versió d'Android per a la qual l'aplicació està adaptada. | Execució |
| `compileSdk` | La versió de l'SDK amb què es compila el projecte. | Compilació |

---

# `minSdk`

Indica la versió mínima d'Android compatible.

```kotlin
minSdk = 24
```

L'aplicació només es podrà instal·lar en dispositius amb Android 7.0 (API 24) o superior.

No afecta al codi que pots escriure, sinó als dispositius compatibles.

---

# `compileSdk`

Indica la versió de l'SDK utilitzada pel compilador.

```kotlin
compileSdk = 36
```

Permet utilitzar totes les classes, mètodes i constants disponibles fins a l'API 36.

No determina en quins dispositius funcionarà l'aplicació.

---

# `targetSdk`

Indica a Android per a quina versió s'ha desenvolupat i provat l'aplicació.

```kotlin
targetSdk = 36
```

Android utilitza aquest valor per aplicar els comportaments corresponents a aquella versió (permisos, restriccions, seguretat, etc.).

---

# Per què normalment `compileSdk > minSdk`?

És la configuració més habitual:

```kotlin
compileSdk = 36
targetSdk = 36
minSdk = 24
```

Això permet:

- que l'aplicació funcioni en Android 7 o superior;
- utilitzar APIs modernes quan el dispositiu les suporta.

---

# Pot fallar una aplicació?

Sí.

Si utilitzem una API introduïda després del `minSdk` sense comprovar la versió d'Android, l'aplicació pot fallar en temps d'execució.

Per exemple:

```kotlin
// API disponible només a partir de l'API 33
novaApi()
```

Aquest codi compila perquè `compileSdk` és prou alt.

Però en un dispositiu amb Android 7 podria provocar una excepció perquè aquella API no existeix.

La solució és comprovar la versió:

```kotlin
if (Build.VERSION.SDK_INT >= 33) {
    novaApi()
} else {
    apiAntiga()
}
```

Android Studio ajuda detectant aquests casos amb avisos com:

```
Call requires API level 33 (current min is 24)
```

---

# Què garanteix `minSdk`?

`minSdk` **només garanteix que l'aplicació es pot instal·lar.**

No garanteix que tot el codi es pugui executar.

És responsabilitat del desenvolupador evitar executar APIs que no existeixen en aquella versió.

---

# Per què no posar `compileSdk = minSdk`?

Perquè impediria utilitzar qualsevol API nova.

Per exemple:

```kotlin
compileSdk = 24
```

El compilador ni tan sols coneixeria les APIs introduïdes a Android 13 o Android 16.

No podríem escriure codi com:

```kotlin
if (Build.VERSION.SDK_INT >= 33) {
    usarApiNova()
}
```

encara que només s'executés en dispositius compatibles.

---

# Les llibreries AndroidX

Moltes llibreries modernes requereixen un `compileSdk` elevat.

Per exemple:

```
Requires compileSdk 36
```

Això **no significa** que només funcionin en Android 16.

La pròpia llibreria implementa la compatibilitat amb versions antigues.

Internament fa coses com:

```kotlin
if (Build.VERSION.SDK_INT >= 33) {
    usarApiNova()
} else {
    usarApiAntiga()
}
```

Per això és habitual trobar llibreries que:

- requereixen `compileSdk = 36`
- són compatibles amb `minSdk = 21`

---

# Configuració recomanada

Actualment és habitual configurar els projectes així:

```kotlin
compileSdk = última versió disponible
targetSdk = última versió disponible
minSdk = la versió mínima que es vulgui suportar
```

Això permet:

- aprofitar les APIs més modernes;
- mantenir compatibilitat amb dispositius antics;
- utilitzar les versions més recents de les llibreries AndroidX.

---

# Idees clau

- **`compileSdk`** determina quines APIs coneix el compilador.
- **`minSdk`** determina en quins dispositius es pot instal·lar l'aplicació.
- **`targetSdk`** determina quin comportament aplica Android a l'aplicació.
- Si `compileSdk > minSdk`, el desenvolupador ha de protegir l'ús de les APIs noves amb comprovacions de versió.
- Les llibreries AndroidX ja implementen internament gran part d'aquesta compatibilitat.