# Meta Quest 3 i Desenvolupament VR amb Unity

La realitat virtual (VR) permet crear experiències immersives on l'usuari interactua amb un entorn tridimensional generat per ordinador. Les ulleres **Meta Quest 3** són un dispositiu standalone (no necessiten PC) que també pot funcionar connectat a un ordinador mitjançant Quest Link.

Unity és un dels motors principals per desenvolupar aplicacions VR gràcies al seu ecosistema XR (eXtended Reality) i la integració oficial amb el **Meta XR SDK**.

- [Documentació oficial Meta XR SDK](https://developer.meta.com/documentation/unity/)
- [Unity XR Documentation](https://docs.unity3d.com/Manual/XR.html)

## 1. Requisits previs

Abans de començar, cal tenir preparat el següent:

| Requisit | Detall |
|----------|--------|
| **Unity** | Versió 2022.3 LTS o superior |
| **Meta Quest 3** | Amb el firmware actualitzat |
| **Cable USB-C** | Per connectar les ulleres al PC durant el desenvolupament |
| **Compte Meta Developer** | Registre gratuït a [developer.meta.com](https://developer.meta.com) |
| **Android Build Support** | Mòdul instal·lat a Unity Hub (inclou Android SDK i NDK) |

!!! warning "Mode desenvolupador"
    Cal activar el **mode desenvolupador** a les ulleres Meta Quest 3 per poder instal·lar aplicacions des de Unity. Sense aquest mode, no es pot fer build ni deploy.

### Activar el mode desenvolupador

1. Obrir l'aplicació **Meta Quest** al mòbil.
2. Anar a **Dispositius** → seleccionar les Quest 3.
3. Anar a **Configuració** → **Mode desenvolupador**.
4. Activar l'interruptor.
5. A les ulleres, acceptar el diàleg de **depuració USB** quan es connecti el cable.

## 2. Configurar el projecte Unity

### Crear el projecte

1. Obrir Unity Hub → **New Project**.
2. Seleccionar la plantilla **3D (URP)** (Universal Render Pipeline).
3. Posar un nom al projecte i clicar **Create project**.

!!! info "Per què URP?"
    El Universal Render Pipeline ofereix un bon equilibri entre qualitat visual i rendiment, essencial per a dispositius mòbils com les Quest 3.

### Canviar la plataforma a Android

1. Anar a **File → Build Settings**.
2. Seleccionar **Android** a la llista de plataformes.
3. Clicar **Switch Platform** (pot trigar uns minuts).

### Player Settings

A **Edit → Project Settings → Player** (pestanya Android), configurar:

| Paràmetre | Valor |
|-----------|-------|
| **Minimum API Level** | Android 10.0 (API level 29) |
| **Scripting Backend** | IL2CPP |
| **Target Architectures** | ARM64 (desmarcar ARMv7) |
| **Install Location** | Automatic |
| **Internet Access** | Require (si cal xarxa) |

!!! info "IL2CPP obligatori"
    Meta Quest 3 requereix el backend **IL2CPP** (Intermediate Language to C++). El backend Mono no és compatible amb les Quest. IL2CPP compila el codi C# a C++ natiu, millorant el rendiment.

## 3. Instal·lar el Meta XR SDK

### 3.1 Meta XR All-in-One SDK (recomanat)

La manera més senzilla d'instal·lar tot el necessari és utilitzar el paquet **Meta XR All-in-One SDK**:

1. Anar a **Window → Package Manager**.
2. Clicar el menú **+** → **Add package by name**.
3. Introduir: `com.meta.xr.sdk.all`
4. Clicar **Add**.

Alternativament, es pot descarregar des de l'Asset Store cercant "Meta XR All-in-One SDK".

### 3.2 Configurar XR Plugin Management

1. Anar a **Edit → Project Settings → XR Plug-in Management**.
2. Instal·lar el paquet si no està instal·lat (clicar **Install XR Plugin Management**).
3. A la pestanya **Android**, marcar **Oculus**.
4. A **XR Plug-in Management → Oculus**, seleccionar **Quest 3** com a Target Device.

!!! tip "Project Setup Tool"
    El Meta XR SDK inclou un **Project Setup Tool** (accessible des de **Meta → Tools → Project Setup Tool**) que verifica automàticament la configuració del projecte i corregeix problemes comuns. Es recomana executar-lo després de la instal·lació.

## 4. Configurar l'XR Rig

L'XR Rig és l'objecte que representa el jugador dins l'escena VR. Inclou la càmera, els controladors i els punts d'ancoratge per al seguiment (tracking).

### 4.1 Afegir l'OVR Camera Rig

1. **Eliminar** la Main Camera de l'escena (ja que el rig inclou la seva pròpia càmera).
2. Al panell Project, cercar el prefab **OVRCameraRig**.
3. Arrossegar-lo a la Hierarchy de l'escena.

L'OVRCameraRig conté la següent jerarquia:

```
OVRCameraRig
├── TrackingSpace
│   ├── LeftEyeAnchor      (càmera ull esquerre)
│   ├── CenterEyeAnchor    (càmera central - la principal)
│   ├── RightEyeAnchor     (càmera ull dret)
│   ├── LeftHandAnchor     (posició mà esquerra)
│   │   └── LeftControllerAnchor
│   ├── RightHandAnchor    (posició mà dreta)
│   │   └── RightControllerAnchor
│   └── TrackerAnchor
```

- Els **EyeAnchors** gestionen l'estereoscòpia (visió 3D).
- Els **HandAnchors** segueixen la posició dels controladors o les mans.

### 4.2 Hand Tracking

Les Quest 3 permeten interactuar directament amb les mans, sense controladors. Per activar-ho:

1. Seleccionar l'objecte **OVRCameraRig** a la Hierarchy.
2. A l'Inspector, al component **OVR Manager**:
   - **Hand Tracking Support**: seleccionar **Controllers And Hands**.
   - **Hand Tracking Frequency**: seleccionar **HIGH** per major precisió.

Exemple de codi per detectar un gest de pinça (pinch):

```csharp
using UnityEngine;

public class HandTrackingExample : MonoBehaviour
{
    [SerializeField] private OVRHand leftHand;
    [SerializeField] private OVRHand rightHand;

    void Update()
    {
        // Detectar pinch amb la mà dreta
        if (rightHand.GetFingerIsPinching(OVRHand.HandFinger.Index))
        {
            Debug.Log("Pinch detectat amb la mà dreta!");
            // Obtenir la posició de la pinça
            OVRSkeleton skeleton = rightHand.GetComponent<OVRSkeleton>();
            if (skeleton != null && skeleton.Bones.Count > 0)
            {
                Vector3 indexTip = skeleton.Bones[(int)OVRSkeleton.BoneId.Hand_IndexTip].Transform.position;
                Debug.Log("Posició del dit índex: " + indexTip);
            }
        }

        // Detectar pinch amb la mà esquerra
        if (leftHand.GetFingerIsPinching(OVRHand.HandFinger.Index))
        {
            Debug.Log("Pinch detectat amb la mà esquerra!");
        }
    }
}
```

Per utilitzar aquest script, cal afegir els components **OVRHand** i **OVRSkeleton** als objectes LeftHandAnchor i RightHandAnchor, i assignar les referències al script.

## 5. Interaccions bàsiques

### 5.1 Agafar objectes (Grab)

Per fer que un objecte es pugui agafar amb els controladors o les mans:

1. Afegir un **Rigidbody** a l'objecte (amb `Use Gravity` activat o no, segons el cas).
2. Afegir un **Collider** (BoxCollider, SphereCollider...).
3. Afegir el component **OVRGrabbable** a l'objecte.
4. Afegir el component **OVRGrabber** als objectes dels controladors (LeftHandAnchor, RightHandAnchor).

Exemple d'un objecte agafable personalitzat:

```csharp
using UnityEngine;

public class CustomGrabbable : OVRGrabbable
{
    // Canviar el color quan l'objecte és agafat
    private Renderer objectRenderer;
    private Color originalColor;

    protected override void Start()
    {
        base.Start();
        objectRenderer = GetComponent<Renderer>();
        originalColor = objectRenderer.material.color;
    }

    public override void GrabBegin(OVRGrabber hand, Collider grabPoint)
    {
        base.GrabBegin(hand, grabPoint);
        objectRenderer.material.color = Color.green;
        Debug.Log("Objecte agafat!");
    }

    public override void GrabEnd(Vector3 linearVelocity, Vector3 angularVelocity)
    {
        base.GrabEnd(linearVelocity, angularVelocity);
        objectRenderer.material.color = originalColor;
        Debug.Log("Objecte deixat anar!");
    }
}
```

### 5.2 Teletransport

El teletransport és un mètode de locomoció habitual en VR que evita el mareig (motion sickness):

1. Crear un pla o superfície on es pugui teletransportar.
2. Afegir-li un component **Collider** i un tag identificatiu (per exemple, `TeleportArea`).
3. Configurar un raig (ray) des del controlador que detecti les zones vàlides.
4. Quan l'usuari activi el trigger del controlador, moure l'OVRCameraRig a la posició apuntada.

El Meta XR SDK inclou exemples de teletransport a les escenes de mostra (Locomotion samples).

### 5.3 UI en VR

Per crear interfícies d'usuari en VR:

1. Crear un **Canvas** i canviar el **Render Mode** a **World Space**.
2. Ajustar l'escala del Canvas (valors com `0.001` per a cada eix funcionen bé).
3. Posicionar el Canvas a l'escena com un panell flotant o fixat a una paret.
4. Afegir el component **OVRRaycaster** al Canvas (substituint el GraphicRaycaster per defecte).
5. Afegir botons, textos i altres elements UI normalment.

!!! warning "No utilitzar Screen Space"
    En VR, el Canvas **mai** ha de ser **Screen Space - Overlay** ni **Screen Space - Camera**. Sempre ha de ser **World Space**. Un Canvas en Screen Space es renderitza directament sobre la pantalla i causa problemes greus de visualització i mareig en VR.

## 6. Build i desplegament

### Connectar i compilar

1. Connectar les Quest 3 al PC amb el cable USB-C.
2. Acceptar el diàleg de depuració USB a les ulleres (si és la primera vegada).
3. Anar a **File → Build Settings**.
4. A **Run Device**, seleccionar les Quest 3 (apareixeran com a dispositiu Android).
5. Clicar **Build And Run**.

| Opció | Descripció |
|-------|------------|
| **Build** | Genera l'APK sense instal·lar-lo |
| **Build And Run** | Genera l'APK i l'instal·la directament a les ulleres |
| **Development Build** | Activa logs de depuració i el Profiler |
| **Autoconnect Profiler** | Connecta automàticament el Profiler de Unity |

!!! note "Primera build lenta"
    La primera compilació amb IL2CPP és significativament més lenta (pot trigar 10-20 minuts) perquè ha de compilar tot el codi C# a C++. Les builds posteriors són incrementals i molt més ràpides.

## 7. Depuració i testing

### 7.1 Quest Link

**Quest Link** permet executar l'aplicació directament des de l'editor de Unity renderitzant al PC i enviant la imatge a les ulleres:

1. Connectar les Quest 3 al PC amb USB-C (o via Wi-Fi amb Air Link).
2. Activar **Quest Link** des del menú ràpid de les ulleres.
3. A Unity, clicar **Play** normalment. L'escena es mostrarà a les ulleres.

Això és molt útil per iterar ràpidament sense haver de fer una build cada vegada.

### 7.2 Meta Quest Developer Hub

El **Meta Quest Developer Hub (MQDH)** és una aplicació de PC que facilita la gestió del dispositiu:

- Instal·lar/desinstal·lar APKs.
- Capturar screenshots i vídeos.
- Monitoritzar el rendiment (FPS, CPU, GPU, memòria).
- Gestionar fitxers del dispositiu.

Es descarrega des de [developer.meta.com/downloads](https://developer.meta.com/downloads).

### 7.3 Logs amb adb logcat

Per veure els missatges de `Debug.Log` des del PC:

```bash
# Veure tots els logs de Unity
adb logcat -s Unity

# Filtrar per tag personalitzat
adb logcat -s Unity | grep "MeuTag"

# Netejar els logs anteriors
adb logcat -c
```

!!! tip "Rendiment objectiu"
    Les Quest 3 suporten taxes de refresc de **72, 90 i 120 FPS**. Per una experiència còmoda, cal mantenir un mínim constant de **72 FPS**. Baixar d'aquest llindar causa mareig i malestar. Utilitza el **Profiler** de Unity i el **MQDH** per monitoritzar el rendiment constantment.
