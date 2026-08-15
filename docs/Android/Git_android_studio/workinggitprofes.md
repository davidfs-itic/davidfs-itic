# Treballant amb Git (profes)

Aquest document complementa [Treballant amb Git](./workinggit.md) i recull com el professorat pot organitzar i fer seguiment dels repositoris d'un curs, sense ser-ne propietari ni haver d'aprovar Pull Requests.

## 1. Accés a un repositori individual

Per a un sol repositori puntual, demana als alumnes que t'afegeixin com a **Collaborator** (a GitHub: Settings → Collaborators) amb el rol:

- **Read**: pots veure tot el codi, l'historial, els issues i els PR, però no pots escriure ni cal que se't demani com a reviewer.
- **Triage**: com Read, però a més pots gestionar issues i PR (assignar, etiquetar, tancar) sense tocar el codi. Útil si vols organitzar el taulell d'issues d'un equip.

Aquest mètode és pràctic per a un projecte aïllat, però no escala bé quan hi ha molts equips: caldria que cadascun et convidés per separat a cada repositori. Per a un curs sencer, és millor fer servir una organització (secció següent).

## 2. Organitzar els repositoris d'un curs amb una organització

Amb un compte GitHub Pro es pot crear una **organització** (per exemple, `itic-alumnes`) que aculli tots els repositoris del curs. És l'equivalent modern al que oferia GitHub Classroom (ja deprecat) per gestionar molts repositoris d'alumnes alhora.

### 2.1. Per què una organització

- Com a **Owner** de l'organització tens accés a tots els repositoris que s'hi creen, automàticament i des del primer moment — no cal que cap equip et convidi individualment.
- Els repositoris queden agrupats en un sol lloc (`github.com/itic-alumnes`), en lloc d'escampats pels comptes personals de cada alumne.
- Pots definir permisos i regles a nivell d'organització que apliquen a tots els repositoris alhora (vegeu la secció 2.4).

### 2.2. Repositori plantilla per a cada activitat

En lloc de fer que cada equip creï el projecte des de zero (secció 1 de [Configuració de Git al projecte](./setupgit.md)), crea un **repositori plantilla** dins l'organització amb l'estructura base ja preparada (`.gitignore`, README amb instruccions, branca `dev` creada, etc.):

1. Crea el repositori base i, a **Settings → General → Template repository**, marca'l com a plantilla.
2. Cada equip, des de la pàgina del repositori, prem **Use this template → Create a new repository** i el crea dins l'organització amb el nom del seu equip (p. ex. `itic-alumnes/equip3-appfitness`).

Els alumnes obtenen un punt de partida consistent sense que hagis de repetir el primer setup manualment amb cada equip, tot i que la creació del repositori individual segueix sent un pas manual (no hi ha una importació automàtica d'una llista de classe com feia GitHub Classroom).

### 2.3. Teams per agrupar alumnes

Dins l'organització pots crear **Teams** (per exemple, un per grup-classe o un per projecte) i afegir-hi els alumnes corresponents. Donar accés a un repositori a un Team en lloc de a cada alumne individualment estalvia feina i manté els permisos endreçats quan hi ha molts equips i repositoris.

### 2.4. Regles de protecció de branca comunes

A **Settings → Rules → Rulesets** de l'organització es poden definir regles que s'apliquen automàticament a tots els repositoris que compleixin un patró de nom (per exemple, `equip*`). És la manera de fer complir tècnicament pràctiques com "no treballar directament a `main`" (que a [Configuració de Git al projecte](./setupgit.md#25-notes-importants) queda només com a recomanació):

- Bloquejar el push directe a `main`, exigint que tot canvi hi arribi via Pull Request.
- Exigir com a mínim una revisió aprovada abans de fer merge.
- Impedir el force-push i l'esborrat de la branca `main`.

D'aquesta manera, el flux `dev → feature/x → PR → dev` i `dev → PR → main` que es descriu a [Treballant amb Git](./workinggit.md) i [Configuració de Git al projecte](./setupgit.md) queda garantit tècnicament a tots els repositoris del curs, en lloc de dependre que cada equip el segueixi pel seu compte.

## 3. Contribucions per usuari

A la pestanya **Insights → Contributors** de cada repositori es veu, per a cada persona, el nombre de commits i línies afegides/eliminades setmana a setmana. **Insights → Pulse** dona un resum de l'activitat recent (PR oberts/tancats, issues, commits).

!!! danger "Perquè els commits comptin bé"
    Aquestes estadístiques agrupen els commits pel correu electrònic configurat a Git (`git config user.email`), no pel nom que escriu l'alumne. Si aquest correu no coincideix amb el del compte de GitHub, els commits apareixen com d'un usuari desconegut i **no es compten** a la gràfica de cap persona real. Val la pena comprovar-ho amb l'equip al principi del projecte (vegeu [Comprova la teva identitat de Git](./setupgit.md#11-comprova-la-teva-identitat-de-git)):
    ```bash
    git config user.email
    ```

## 4. Qui ha fet cada tasca: Issues per assignat

A la pestanya **Issues**, filtra per `assignee:nom-usuari` per veure quantes tasques té assignades cada alumne i quantes ha tancat. Combinat amb els Insights de commits, dona una imatge força completa de qui ha treballat en què.
