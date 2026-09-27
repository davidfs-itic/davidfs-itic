# Configuració de Temes i Mode Fosc a Android (Views)


Aquesta guia recull el procés pas a pas per crear un tema personalitzat a Android mitjançant l'herència del tema principal de Material Components, configurant tant el mode clar com el mode fosc, incloent colors per a superfícies i textos.

## Referències

Documentació 
https://developer.android.com/develop/ui/views/theming/themes
Codelab:
https://developer.android.com/codelabs/basic-android-kotlin-training-change-app-theme?hl=es-419#0

## Bones pràctiques per començar amb els temes a Android

Abans de posar-te a escriure codi, tingues en compte aquests consells fonamentals:

1. **No reinventis la roda (Utilitza Material Components):** Hereta sempre dels temes oficials (com `Theme.MaterialComponents.DayNight.NoActionBar`). Així ja tindràs integrat el suport per a mode fosc, accessibilitat i estils estàndard.

2. **Canvia només els atributs clau:** Limita't al principi a modificar només els colors essencials (`colorPrimary`, `colorSecondary`, `android:windowBackground`, `colorSurface` i `colorOnSurface`) per mantenir la coherència visual sense complicacions.

3. **Fes servir referències `?attr/` als Layouts:** No posis colors fixos al XML de les vistes (com `#FFFFFF`). Utilitza atributs del tema perquè el sistema pugui alternar entre el mode clar i fosc de manera automàtica.

4. **Mantén els fitxers nets i ordenats:** Separa els valors "bruts" a `colors.xml` i assigna'ls als atributs dins de `themes.xml`, de manera que puguis modificar la paleta global des d'un sol lloc.

## Pas 1: Defineix els colors de la teua marca (`colors.xml`)

Crea o actualitza el fitxer a la ruta `res/values/colors.xml` amb la teua paleta de colors base, afegint també els colors per a superfícies i textos sobre superfícies:

```xml
<resources>
    <!-- ========================================== -->
    <!-- PALETA MODE CLAR (Vibrant Violet Style)    -->
    <!-- ========================================== -->
    
    <!-- Color principal de la marca (Violeta vibrant i elèctric) -->
    <color name="blau_marca">#FF7C3AED</color>
    
    <!-- Color secundari o d'accent (Rosa/Lila complementari) -->
    <color name="blau_secundari">#FFEC4899</color>
    
    <!-- Fons general de la finestra (Gris molt clar amb un subtil toc lila) -->
    <color name="fons_clar">#FFF8FAFC</color>
    
    <!-- Superfícies (Tarjetes, contenidors): Blanc pur -->
    <color name="superficie_clara">#FFFFFFFF</color>
    
    <!-- Text sobre superfícies clares (Gris molt fosc per a una lectura òptima) -->
    <color name="text_sobre_superficie_clar">#FF0F172A</color>


    <!-- ========================================== -->
    <!-- PALETA MODE FOSC (Deep Neon Violet Style)  -->
    <!-- ========================================== -->
    
    <!-- Color principal adaptat per a mode fosc (Violeta més suau i lluminós) -->
    <color name="blau_marca_fosc">#FFC084FC</color>
    
    <!-- Color secundari adaptat per a mode fosc -->
    <color name="blau_secundari_fosc">#FFF472B6</color>
    
    <!-- Fons general de la finestra (Negre fosc modern / Slate profund) -->
    <color name="fons_fosc">#FF09090B</color>
    
    <!-- Superfícies en mode fosc (Gris fosc elevat) -->
    <color name="superficie_fosca">#FF18181B</color>
    
    <!-- Text sobre superfícies fosques (Blanc gairebé pur) -->
    <color name="text_sobre_superficie_fosc">#FFF4F4F5</color>
</resources>
```

## Pas 2: Crea el tema principal heretant de Material (`res/values/themes.xml`)

Crea el tema principal de la teua aplicació fent que **hereti directament** de `Theme.MaterialComponents.DayNight.NoActionBar`, incorporant `colorSurface` i `colorOnSurface`:

```xml
<resources xmlns:tools="http://schemas.android.com/tools">
    
    <!-- El nostre tema hereta directament del tema principal de Material -->
    <style name="Theme.LaMevaApp" parent="Theme.MaterialComponents.DayNight.NoActionBar">
        
        <!-- Definim els atributs personalitzats per al mode clar -->
        <item name="colorPrimary">@color/blau_marca</item>
        <item name="colorSecondary">@color/blau_secundari</item>
        <item name="android:windowBackground">@color/fons_clar</item>

        <!-- Superfícies i textos sobre superfícies -->
        <item name="colorSurface">@color/superficie_clara</item>
        <item name="colorOnSurface">@color/text_sobre_superficie_clar</item>
        
    </style>
    
</resources>
```

## Pas 3: Sobreescriu el tema per al Mode Fosc (`res/values-night/themes.xml`)

Crea una carpeta nova anomenada `values-night` dins de `res/` (si no existeix) i afegeix-hi un fitxer `themes.xml` adaptat al mode fosc:

```xml
<resources xmlns:tools="http://schemas.android.com/tools">
    
    <!-- Mateix nom de tema, mantenint l'herència de Material DayNight -->
    <style name="Theme.LaMevaApp" parent="Theme.MaterialComponents.DayNight.NoActionBar">
        
        <!-- Definim els atributs adaptats per al mode fosc -->
        <item name="colorPrimary">@color/blau_marca_fosc</item>
        <item name="colorSecondary">@color/blau_secundari_fosc</item>
        <item name="android:windowBackground">@color/fons_fosc</item>
        
        <!-- Superfícies i textos sobre superfícies en mode fosc -->
        <item name="colorSurface">@color/superficie_fosca</item>
        <item name="colorOnSurface">@color/text_sobre_superficie_fosc</item>
        
    </style>
    
</resources>
```

### Pas 4: Aplica el tema al Manifest (`AndroidManifest.xml`)

Perquè s'apliqui a tota l'aplicació, assigna el tema a l'etiqueta `<application>`: Normalment ja està aplicat si hem utilitzat el tema per defecte.


```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <application
        android:theme="@style/Theme.LaMevaApp"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true">
        
        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        
    </application>

</manifest>
```