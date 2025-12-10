
# Layout general de les activities.

Referència:https://developer.android.com/design/ui/mobile/guides/foundations/system-bars

Llegui també: https://developer.android.com/develop/ui/views/layout/edge-to-edge

En un layout modern (API 24+), tenim tres elements principals que afecten el layout: el StatusBar, el contingut (View) i el NavigationBar (o Gesture bar). 

Totes dues es coneixen amb el nom de **System bars**

Tant el SystemBar com el Navigationbar formen part del sistema operatiu. Es poden amagar (full screen), fer transparents o fer opacs, però no eliminar.

| Layout normal| amb EdgetoEdge()
|:--:|:---:|
![System bars](./Imatges/systembar1.png)|![System bars](./Imatges/systembar2.png)


Quan creem una nova activity, per defecte, s'executa una funció 

```kotlin
enableEdgeToEdge() 
```

que fà que el contingut es posi per sota els systembars.

Per evitar que el nostre contingut (View) es confongui amb les System bars, s'apliquen uns paddings amb la funció:

```kotlin
/ViewCompat.setOnApplyWindowInsetsListener(findViewById(R.id.main)) { v, insets ->
    val systemBars = insets.getInsets(WindowInsetsCompat.Type.systemBars())
    v.setPadding(systemBars.left, systemBars.top, systemBars.right, systemBars.bottom)
    insets
}
```



