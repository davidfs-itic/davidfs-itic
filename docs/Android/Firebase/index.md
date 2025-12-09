# Firebase

Continguts:

- Setup de Firebase a Android: [Android/Firebase/fbsetup.md](./fbsetup.md)
- Autenticació: (Pendent)
- Firestore:  (Pendent)
- Storage:  (Pendent)
- FireCloud Messaging:  (Pendent)

## Les eines de Firebase es poden dividir en 4 blocs principals:

## 1. Back-end en temps real (les més importants)
### Firestore (Base de dades NoSQL en temps real)

La més potent i fàcil per a apps mòbils.

Emmagatzema documents i col·leccions (JSON)

Actualitza automàticament la UI quan hi ha canvis

Permet consultes (filters, orders)

Funciona offline

Permet permisos basats en usuari (rules)

#### Projectes típics:

Xat en temps real

Notes/fites compartides

CRUD complet sense backend propi

Llistes sincronitzades amb RecyclerView

### Realtime Database

La base de dades clàssica de Firebase.

També en temps real

Més senzilla però menys flexible que Firestore

#### Projectes:
CRUD molt simple, jocs o apps col·laboratives.

## 2. Autenticació i seguretat
### Firebase Authentication (imprescindible)

Manera fàcil i moderna d’autenticar usuaris:

Correu + contrasenya

Google, Facebook, Apple

Anònim

Enviament de correus de verificació

#### Perfeta per:
Login + registre complet en 15 minuts, sense backend.

### Cloud Firestore Security Rules

Sistema per definir permisos:

Qui pot llegir?

Qui pot escriure?

Condicions basades en usuari?

#### Projectes:
Apps multi-usuari segures sense servidor.

## 3. Comunicació i notificacions
### Firebase Cloud Messaging (FCM)

Servei professional per a notificacions push.

Enviar notificacions a dispositius Android

Des de consola o backend

Personalització per topic (“novetats”, “promocions”…)

Rebre notificacions en segon pla

#### Perfecte per:
Enviar avisos a moda de “backend real”.

## 4. Eines complementàries (molt útils segons el projecte)
### Firebase Storage

Emmagatzemament d’arxius:

Imatges

PDFs

Audios

Vídeos

#### Projectes:
Perfil d’usuari amb foto, pujar documents.

### Firebase Analytics

Analítica integrada:

Pantalles visitades

Temps en cada vista

Esdeveniments personalitzats


# RECOMANACIÓ: El “Pac de 4” imprescindible

Per a projectes, els quatre serveis més rellevants i aplicables són:

## 1. Firebase Authentication

- login i registre sense servidor.

## 2. Firebase Firestore

- CRUD en temps real i sincronització automàtica.

## 3. Firebase Cloud Messaging (FCM)

- notificacions push reals.

## 4. Firebase Storage

- pujar i descarregar imatges.

## Amb sols aquests 4, els alumnes poden construir:

- Una xarxa social bàsica

- App de xat

- Gestor de tasques multiusuari

- Aplicació de notícies amb notificacions push

- App amb perfils d’usuari i fotos

- Qualsevol CRUD sincronitzat i multi-dispositiu