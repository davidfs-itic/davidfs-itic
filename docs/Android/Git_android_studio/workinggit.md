# Treballant amb git

Un cop fet el [setup inicial de Git](./setupgit.md), aquest document explica el dia a dia de treballar en equip: com organitzar el treball en branques i issues, com es relacionen amb els Pull Requests, i com fer un seguiment de la teva pròpia activitat al repositori.

!!! info "Ets professor?"
    Si vols organitzar i fer seguiment dels repositoris d'un curs sencer, consulta [Treballant amb Git (profes)](./workinggitprofes.md).

## 1. Branques per feature (recomanat)

Quan diversos alumnes treballen sobre la mateixa branca `dev`, és millor crear una **branca curta per cada funcionalitat o tasca** que no pas una branca fixa per persona.

Una branca per persona tendeix a allargar-se massa: acumula diverses funcionalitats barrejades, diverge molt de `dev` amb el temps, i acaba generant Pull Requests enormes i difícils de revisar. Amb una branca per feature, cada Pull Request és petit i clar (una funcionalitat = un PR), els conflictes són menys freqüents perquè la branca viu poc temps, i el nom de la branca ja explica què s'hi està fent.

### 1.1. Nomenclatura

```
feature/nom-curt-de-la-funcionalitat
```

Exemples: `feature/login`, `feature/llista-tasques`, `feature/dark-mode`.

### 1.2. Flux de treball

```bash
git checkout dev
git pull
git checkout -b feature/nom-de-la-funcionalitat
```

Treballa i fes els commits necessaris (vegeu la [secció 2](#2-fer-commits-sovint)) i, quan la funcionalitat estigui llesta:

```bash
git push -u origin feature/nom-de-la-funcionalitat
```

Obre un Pull Request de `feature/nom-de-la-funcionalitat` cap a `dev` a GitHub. Un cop revisat i fusionat, la branca de feature ja no fa falta i es pot esborrar (a GitHub mateix hi ha un botó "Delete branch" un cop fusionat el PR).

!!! tip "Dues persones, mateixa funcionalitat gran"
    Si una funcionalitat és massa gran per a una sola persona, és millor dividir-la en tasques més petites (i per tant en issues i branques més petites — vegeu la [secció 5](#5-issues-i-pull-requests)) que no pas que dues persones treballin alhora sobre la mateixa branca de feature.

## 2. Fer commits sovint

Fes **commits petits i freqüents**, no un únic commit gran al final de la sessió. Cada commit hauria de representar un canvi coherent i petit (una funció, un fix, un ajust d'UI), amb un missatge descriptiu.

Per què és important:

- **Menys risc de perdre feina**: si alguna cosa va malament, sempre pots tornar a l'últim commit en lloc de perdre hores de treball.
- **Historial més útil**: és molt més fàcil entendre (o desfer) "Afegir validació del formulari de login" que un únic commit "Canvis del dia".
- **Conflictes més petits i fàcils de resoldre**: com més petit és el commit, més fàcil és veure exactament què xoca amb el canvi d'un altre company.

!!! warning "Comita sempre abans de fer pull"
    Si tens canvis pendents (encara que no estiguin acabats), **comita'ls abans de fer `git pull`**, encara que sigui amb un missatge senzill com `"WIP: canvis a mig fer"`. Si el `pull` provoca un conflicte, Android Studio et deixarà resoldre'l amb la seva eina visual de fusió; sense haver comitat abans, els teus canvis es poden barrejar amb els entrants i és molt més difícil saber què és de qui.

En resum: **comita sovint** durant la sessió de treball, i **sempre** abans de fer `pull`.

## 3. Resolució de conflictes

Un **conflicte de fusió** (merge conflict) passa quan Git no pot combinar automàticament dos canvis fets sobre les mateixes línies d'un fitxer (per exemple, dues persones han modificat el mateix mètode de maneres diferents).

Quan això passa, Git marca el fitxer com a conflictiu i n'atura la fusió fins que es resol manualment. Android Studio ofereix una eina visual per a això (**Merge** de tres columnes: la teva versió a l'esquerra, el resultat combinat al centre, la versió remota a la dreta), accessible automàticament quan un `pull` o `merge` detecta un conflicte.

!!! tip "Com resoldre'ls bé"
    - No triïs "Accept Both" sense mirar-ho: sovint cal combinar manualment les dues versions perquè el codi resultant tingui sentit.
    - Si el conflicte és en un fitxer que no entens del tot (per exemple, generat per l'IDE), pregunta a qui ha fet l'altre canvi abans de decidir.
    - Un cop resolt, compila i executa l'aplicació abans de fer `commit` — un conflicte "resolt" pot deixar codi que no compila.

## 4. Com fer el merge d'una feature

Un cop la branca de feature (secció 1) està acabada i el Pull Request ha estat revisat, cal fusionar-lo a `dev`:

1. A GitHub, dins del Pull Request, prem **Merge pull request**.
2. Esborra la branca de feature (GitHub mostra un botó **Delete branch** just després de fusionar-la) — ja no fa falta, perquè els seus canvis ja formen part de `dev`.
3. Actualitza la teva còpia local de `dev`:

   ```bash
   git checkout dev
   git pull
   ```

4. Comença la següent tasca creant una nova branca de feature des de `dev` actualitzat (secció 1.2).

!!! info "Tipus de merge a GitHub"
    Al botó "Merge pull request" hi ha diverses opcions:

    - **Create a merge commit**: manté tots els commits originals de la branca i afegeix un commit de fusió. Conserva l'historial complet.
    - **Squash and merge**: combina tots els commits de la branca en un de sol abans de fusionar-lo a `dev`. Recomanat si la branca té molts commits petits tipus "WIP" (secció 2) i es vol que l'historial de `dev` quedi net, amb un commit per feature.
    - **Rebase and merge**: reaplica els commits de la branca sobre `dev` sense crear un commit de fusió. Manté l'historial lineal, però és més avançat i no és necessari per a l'ús habitual en aquest curs.

## 5. Issues i Pull Requests

Un **issue** és una tasca, millora o error que es vol tractar, descrita a GitHub abans de començar-hi a treballar. Un **Pull Request (PR)** és la proposta de canvis concrets (una branca) per resoldre'l. La relació entre tots dos és el que dona traçabilitat al projecte: **què s'havia previst fer** (issue) enfront de **què s'ha acabat entregant** (PR).

### 5.1. Crear un issue

A la pestanya **Issues** del repositori a GitHub:

- Títol curt i descriptiu (p. ex. "Afegir pantalla de login").
- Descripció amb el que cal fer i, si escau, criteris d'acceptació.
- (Opcional) Assignar-lo a la persona que hi treballarà i afegir-hi etiquetes (`bug`, `feature`, etc.).

### 5.2. Vincular la branca i el PR a l'issue

Al crear la branca de feature (secció 1), pots basar el nom en el número o el tema de l'issue (p. ex. `feature/12-login`). Però el vincle **real** es fa al Pull Request: a la descripció del PR, escriu una paraula clau seguida del número de l'issue:

```
Closes #12
```

Quan aquest PR es fusiona a `dev` (o `main`), GitHub **tanca automàticament l'issue #12**. Així no cal tancar-lo manualment i queda registrat quin PR exactament l'ha resolt.

### 5.3. Flux complet

```
Issue #12 "Afegir login"
        │
        ▼
git checkout -b feature/12-login
        │
   (commits...)
        │
        ▼
Pull Request "Closes #12" ──► revisió ──► merge a dev
        │
        ▼
   Issue #12 es tanca automàticament
```

## 6. Fer un seguiment de la teva pròpia activitat

Amb accés de lectura o escriptura normal al repositori (no cal cap permís especial) ja pots veure tota la teva activitat, tant en local com a GitHub.

### 6.1. El teu historial de commits

En local, filtra el `git log` pel teu nom per veure només els teus commits:

```bash
git log --author="El teu nom" --oneline
```

A GitHub, la pestanya **Insights → Contributors** del repositori mostra, per a cada persona (inclòs tu), el nombre de commits i línies afegides/eliminades setmana a setmana.

!!! warning "Els teus commits no hi surten?"
    Aquestes estadístiques agrupen els commits pel correu electrònic configurat a Git (`git config user.email`). Si aquest correu no coincideix amb el del teu compte de GitHub, els teus commits apareixen com d'un usuari desconegut. Comprova-ho i corregeix-ho si cal (vegeu [Comprova la teva identitat de Git](./setupgit.md#11-comprova-la-teva-identitat-de-git)).

### 6.2. Filtrar les teves Pull Requests i Issues

Al cercador de la pestanya **Pull requests** o **Issues** del repositori, pots filtrar pel que és teu:

```
is:pr author:el-teu-usuari
is:issue assignee:el-teu-usuari
```

### 6.3. El gràfic de contribucions del teu perfil

El teu perfil de GitHub (`github.com/el-teu-usuari`) mostra el típic gràfic de quadrets verds amb la teva activitat diària a tots els repositoris.

!!! tip "Repositoris privats"
    Si els repositoris del curs són privats, per defecte l'activitat no hi surt reflectida. Activa-ho a **Settings → Profile → Contributions & Activity → Include private contributions on my profile**: es mostrarà que has fet activitat un dia concret, sense revelar en quin repositori ni què has canviat.
