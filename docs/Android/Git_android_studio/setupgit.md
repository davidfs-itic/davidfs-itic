# Configuració de Git al projecte

Guia del flux de treball amb Git per a projectes Android: com fer el primer setup d'un repositori nou i com incorporar-se a un projecte ja existent per treballar-hi en equip.

Documentació oficial: [Use version control systems | Android Studio](https://developer.android.com/studio/intro/version-control)

## 1. Crear un projecte nou (primer setup)

Passos per començar un projecte des de zero i pujar-lo a Git per primera vegada.

### 1.1. Comprova la teva identitat de Git

Abans de fer cap commit, comprova que Git sap qui ets: el `user.name` i el `user.email` s'enganxen a cada commit que facis i en determinen l'autoria.

```bash
git config --global user.name
git config --global user.email
```

Si algun dels dos surt buit, configura'ls (utilitza el mateix correu que el del teu compte de GitHub):

```bash
git config --global user.name "El teu nom"
git config --global user.email "el-teu-correu@exemple.com"
```

!!! warning "Per què ha de coincidir amb el correu de GitHub"
    L'opció `--global` desa la configuració per a tots els projectes de l'ordinador, així no cal repetir-ho cada vegada. Si el `user.email` no coincideix amb el correu del teu compte de GitHub, els teus commits no s'atribueixen correctament al teu usuari: apareixen com d'algú desconegut a l'historial i no es compten a les estadístiques de contribució (vegeu com fer-ne el seguiment a [Treballant amb Git](./workinggit.md#61-el-teu-historial-de-commits) o, com a professor, a [Treballant amb Git (profes)](./workinggitprofes.md#3-contribucions-per-usuari)).

### 1.2. Crear el repositori remot buit

Crea un repositori nou a GitHub (o al servei que useu).

**Important:** no l'inicialitzis amb README, `.gitignore` ni llicència — ha de quedar **completament buit**. Si el remot ja té algun commit, el primer `push` xocarà amb l'historial local (històries divergents) i caldrà fer un `pull`/`merge` abans de res, complicant innecessàriament aquest primer setup.

### 1.3. Crear el projecte a Android Studio

- Crea el projecte nou dins la carpeta de treball amb l'assistent d'Android Studio.
- **No marquis l'opció de crear repositori Git automàticament** (si l'assistent la mostra): el farem manualment al pas següent, per tenir-ne control total.
- Un cop creat, afegeix al `.gitignore` generat per Android Studio les exclusions addicionals per a fitxers locals de `.idea/`:

  ```
  /.idea/gradle.xml
  /.idea/vcs.xml
  /.idea/markdown.xml
  ```

- Espera que Android Studio acabi de crear tota l'estructura i completi el primer **Gradle Sync**. És important fer-ho *abans* del `git add .` del pas següent, perquè Android Studio podria trigar en fer tot el setup.

### 1.4. Configurar Git en local

```bash
git init
git remote add origin <URL_DEL_REPOSITORI>
git add .
git commit -m "Primer commit"
git push -u origin main
```

!!! warning "Comprova el nom de la branca abans del push"
    Segons la configuració de la teva instal·lació de Git, `git init` pot crear la branca inicial com a `master` en lloc de `main`. Comprova-ho amb `git branch` i, si cal, renomena-la abans de pujar-la:
    ```bash
    git branch -M main
    ```

`git add .` és segur en aquest punt perquè els fitxers que no volem versionar (fitxers locals de `.idea/`, `build/`, `local.properties`, etc.) ja estan exclosos pel `.gitignore` (vegeu la [secció 3](#3-el-fitxer-gitignore-en-projectes-android)).

### 1.5. Crear la branca `dev` i sincronitzar-la

```bash
git checkout -b dev
git push -u origin dev
```

A partir d'aquí, `main` i `dev` existeixen tant en local com al remot, i ja es pot compartir el projecte amb la resta de l'equip (vegeu la secció següent).

---

## 2. Començar amb un projecte creat

Instruccions per clonar el projecte i començar a treballar-hi a la branca `dev`.

Si és el primer cop que uses Git en aquest ordinador, comprova primer la teva identitat (`user.name` i `user.email`) tal com s'explica a la [secció 1.1](#11-comprova-la-teva-identitat-de-git).

### 2.1. Clonar el repositori

```bash
git clone <URL_DEL_REPOSITORI>
cd <NOM_DEL_PROJECTE>
```

### 2.2. Canviar a la branca `dev`

Tot el desenvolupament es fa a `dev`, no a `main`.

```bash
git checkout dev
```

Si la branca no apareix amb `git branch`, primer actualitza les referències del remot:

```bash
git fetch origin
git checkout dev
```

### 2.3. Obrir el projecte amb Android Studio

Obre la carpeta clonada des d'Android Studio i espera que faci el **Gradle Sync**.

### 2.4. Flux de treball habitual

Abans de començar a treballar, actualitza la teva còpia local:

```bash
git pull
```

!!! tip "Recomanació"
    Comita sempre els teus canvis pendents *abans* de fer `pull`, encara que sigui amb un missatge senzill com `"WIP: canvis a mig fer"`. Si el `pull` provoca algun conflicte, Android Studio t'ajudarà a resoldre'l amb la seva eina visual de fusió (tres columnes: la teva versió, el resultat, la del remot), molt més clara que fer-ho sense haver comitat abans. I si no hi ha cap conflicte, Git fusiona sol i ja pots continuar treballant amb normalitat. Vegeu la [resolució de conflictes](./workinggit.md#3-resolucio-de-conflictes) a "Treballant amb Git" per a més detall.

Després de fer canvis:

```bash
git add <fitxers>
git commit -m "Missatge descriptiu del canvi"
git push
```

### 2.5. Notes importants

- **No treballis directament a `main`**. `main` es reserva per a versions estables.
- Si Android Studio et proposa afegir fitxers nous dins `.idea/` al control de versions, **revisa'ls abans**: si contenen rutes absolutes del teu ordinador (per exemple dins `<tag ... path="/home/usuari/...">`), no els afegeixis — probablement ja estan (o haurien d'estar) al `.gitignore`.
- Consulta el fitxer `.gitignore` de l'arrel per veure què es considera "compartit" i què es considera "local" dins `.idea/`.

---

## 3. El fitxer .gitignore en projectes Android

Un projecte Android genera molts fitxers que **no s'han de versionar**: o bé es regeneren automàticament (compilats), o bé són específics de la màquina de cada desenvolupador. Android Studio crea un `.gitignore` inicial força complet, però és important entendre **què hi ha i per què**:

| Categoria | Exemples | Per què s'ignora |
|---|---|---|
| Sortida de compilació | `/build`, `*/build`, `.gradle/` | Es regenera automàticament amb cada compilació; ocupa molt espai i provoca conflictes constants |
| Configuració local de l'IDE | `.idea/workspace.xml`, `.idea/gradle.xml`, `.idea/vcs.xml` | Conté preferències i rutes absolutes de cada desenvolupador (finestres obertes, breakpoints, etc.) |
| Ruta de l'SDK | `local.properties` | Conté la ruta absoluta a l'SDK d'Android instal·lat a *cada* ordinador (`sdk.dir=...`); si es comparteix, trencarà el projecte a qualsevol altra màquina |
| Credencials i claus | `google-services.json`, fitxers `*.keystore`, claus d'API | Dades sensibles que mai s'han de fer públiques en un repositori (vegeu [Configuració de Firebase](../Firebase/fbsetup.md)) |

!!! danger "Si ja has pujat per error un fitxer sensible"
    Afegir-lo al `.gitignore` **no l'elimina de l'historial** de Git. Cal treure'l del seguiment amb `git rm --cached <fitxer>`, comitar el canvi, i si el fitxer contenia credencials reals (claus d'API, contrasenyes), considerar-les compromeses i regenerar-les.

## 4. Publicar canvis a `main` (release)

Un cop el desenvolupament a `dev` és estable i es vol publicar com a nova versió, cal fusionar `dev` dins de `main`. En equip, la manera recomanada és mitjançant una **Pull Request** a GitHub, en lloc de fer la fusió directament en local:

1. Assegura't que `dev` està pujat i actualitzat (`git push`).
2. A GitHub, obre una **Pull Request** de `dev` cap a `main`.
3. Un altre membre de l'equip revisa els canvis (code review) abans d'aprovar-la.
4. Es fa el **merge** des de GitHub, que actualitza `main` amb tot el contingut de `dev`.
5. Tothom actualitza la seva còpia local de `main` amb `git checkout main && git pull`.

!!! info "Per què no fusionar directament en local?"
    La Pull Request deixa constància de **què s'ha publicat i quan**, permet una revisió de codi abans d'arribar a `main`, i evita que algú pugi canvis a la branca estable sense que la resta de l'equip se n'assabenti.

## 5. Problemes habituals

- **Vaig pujar `local.properties` o `google-services.json` per error**: afegeix el fitxer al `.gitignore` i treu-lo del seguiment amb `git rm --cached <fitxer>` (vegeu l'avís de la [secció 3](#3-el-fitxer-gitignore-en-projectes-android)).
- **La branca es diu `master` en lloc de `main`**: renombra-la amb `git branch -M main` abans del primer `push` (vegeu la [secció 1.4](#14-configurar-git-en-local)).
- **Els meus commits no surten a les estadístiques de contribució**: comprova que el `user.email` de Git coincideix amb el del teu compte de GitHub (vegeu la [secció 1.1](#11-comprova-la-teva-identitat-de-git)).
- **Android Studio proposa afegir fitxers de `.idea/` desconeguts**: revisa'n el contingut abans d'acceptar-los; si tenen rutes absolutes del teu ordinador, no els versionis.
- **"SDK location not found" en clonar el projecte a un altre ordinador**: normal, ja que `local.properties` no es versiona. Android Studio el regenera automàticament en fer el primer Gradle Sync.
