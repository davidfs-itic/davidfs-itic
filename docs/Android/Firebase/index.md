# Firebase

Continguts:

- Setup de Firebase a Android: [Android/Firebase/fbsetup.md](./fbsetup.md)
- Firestore:  [Android/Firebase/firestore.md](./firestore.md)
- Autenticació: (Pendent)
- Storage:  (Pendent)
- FireCloud Messaging:  (Pendent)

## Les eines de Firebase es poden dividir en 4 blocs principals:

## 1. Back-end Database
### 1.1. Firestore (Base de dades NoSQL)

- Base de Dades NoSQL Documental
    - Estructura flexible: Documents → Col·leccions → Subcol·leccions
    - Sense esquema fix: Pots afegir camps dinàmicament
    - Millor escalabilitat que bases de dades SQL per a certs casos d'ús
- Sincronització en Temps Real
    - Listeners automàtics: Rebes canvis immediatament
    - Multiplataforma: Tots els clients reben les dades actualitzades
    - Offline-first: Funciona sense connexió i sincronitza després
- Seguretat Integrada
    - Regles de seguretat: Control d'accés a nivell de document
    - Validació de dades: A les mateixes regles
    - Autenticació integrada: Amb Firebase Auth
- Limitacions clau:
    - Sense JOINs: Has de desnormalitzar dades
    - Sense consultes entre col·leccions: Tot en una consulta ha d'estar a la mateixa col·lecció
    - Restriccions en filtres: No pots filtrar per dos camps diferents sense índex compost

#### Projectes típics:

App de Tasques (Todoist-like)
App de Fitness (Strava-like)

#### En general:
- Firestore és ideal quan:
    - Necessites temps real
    - Tens dades jeràrquiques o estructurades
    - Vols escalar automàticament
    - El teu model de dades no necessita JOINs complexos
    - Pots desnormalitzar les dades
- Millor evitar Firestore quan:
    - Necessites moltes agregacions (SUM, AVG, GROUP BY)
    - Tens relacions complexes entre entitats
    - Necessites consistència estricta en tot moment
    - Tens volums massius d'escriptura contínua

### 1.2 Realtime Database

- Estructura Única: Arbre JSON Gran
    - Un gran JSON: Totes les dades en un sol arbre
    - Anidació profunda: nodes/ins/outre/loques/sigui/profund
    - Sense esquema: Totalment flexible

- Sincronització Ultra Ràpida
    - Baixíssima latència: Mil·lisegons per actualitzacions
    - WebSocket persistent: Connexió contínua
    - Broadcast automàtic: Tots els clients reben canvis immediatament

- Consultes Limitades Simples
    - Consultes bàsiques: .orderByChild(), .orderByValue(), .orderByKey()
    - Un sol filtre per consulta: No pots combinar múltiples where()
    - Sense índexs complexos: Més simple de gestionar

#### Projectes:
- Aplicacions de Xat en Temps Real
- Jocs Multijugador i Competició
- Aplicacions de Presència i Estat (connectat, escrivint...)
- Control i Monitorització IoT

#### En general:

- Tria Realtime Database quan:
    - Necessites latència ultra baixa (<100ms)
    - Tens dades que canvien constantment (cada segon)
    - El teu model de dades és plà o arbre simple
    - Necessites broadcast a tots els clients simultàniament
    - Tens pocs usuaris concurrents (o acceptes el límit)

- Millor evitar RTDB quan:
    - Necessites consultes complexes amb múltiples filtres
    - Esperes creixement massiu (>200k usuaris concurrents)
    - Tens dades amb estructura molt jeràrquica
    - Necessites transaccions complexes
    - Vols paginació avançada i ordenació

## 2. Autenticació i seguretat
### 2.1, Firebase Authentication (imprescindible)

- Manera fàcil i moderna d’autenticar usuaris:
- Correu + contrasenya
- Google, Facebook, Apple
- Anònim
- Enviament de correus de verificació

#### Perfeta per:
- Login + registre complet en 15 minuts, sense backend.

### 2.2, Cloud Firestore Security Rules

- Sistema per definir permisos:
    - Qui pot llegir?
    - Qui pot escriure?
    - Condicions basades en usuari?

#### Projectes:

- Apps multi-usuari segures sense servidor.

## 3. Comunicació i notificacions
### Firebase Cloud Messaging (FCM)

- Servei professional per a notificacions push.
- Enviar notificacions a dispositius Android
- Des de consola o backend
- Personalització per topic (“novetats”, “promocions”…)
- Rebre notificacions en segon pla

#### Perfecte per:

- Enviar avisos a moda de “backend real”.

## 4. Eines complementàries (molt útils segons el projecte)
### Firebase Storage

- Emmagatzemament d’arxius:
    - Imatges
    - PDFs
    - Audios
    - Vídeos

#### Projectes:

- Perfil d’usuari amb foto, pujar documents.

### Firebase Analytics

- Analítica integrada:
    - Pantalles visitades
    - Temps en cada vista
    - Esdeveniments personalitzats
