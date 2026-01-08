# Gràfics amb MPAndroidChart


## 1. Què és MPAndroidChart?
**MPAndroidChart** és una llibreria molt utilitzada per mostrar **gràfics estadístics** en aplicacions Android, com ara:
- Gràfics de barres (BarChart)
- Gràfics de línies (LineChart)
- Gràfics de sectors (PieChart)


## 2. Afegir la llibreria al projecte

Al fitxer **build.gradle (Module: app)**:

```gradle
dependencies {
    implementation("com.github.PhilJay:MPAndroidChart:v3.1.0")
}
```

Després fes **Sync Now**.

## 3. Afegir un gràfic al layout (XML)

Exemple d’un **LineChart** a `activity_main.xml`:

```xml
<com.github.mikephil.charting.charts.LineChart
    android:id="@+id/lineChart"
    android:layout_width="match_parent"
    android:layout_height="300dp"
    android:layout_margin="16dp" />
```

Altres opcions habituals:
- `BarChart`
- `PieChart`



## 4. Preparar les dades

En un arxiu **Datasource**

```kotlin
object dades {
    val entries = listOf<Entry>(
    Entry(1f, 10f),
    Entry(2f, 15f),
    Entry(3f, 12f),
    Entry(4f, 20f))
}
```

📌 Cada `Entry(x, y)` representa un punt del gràfic.



## 5. Crear el DataSet per al gràfic.

En l'activity
```kotlin
val lineChart = findViewById<LineChart>(R.id.lineChart)
val dataSet = LineDataSet(dades.entries, "Vendes mensuals")
dataSet.color = Color.BLUE
dataSet.setCircleColor(Color.RED)
dataSet.valueTextSize = 12f
dataSet.lineWidth = 2f
```



## 6. Assignar les dades al gràfic.

```kotlin

val data = LineData(dataSet)
lineChart.data = data
lineChart.description.text = "Informe de vendes"
lineChart.invalidate()
```

