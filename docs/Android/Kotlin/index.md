# Llenguatge Kotlin

Continguts:

- Variables i Constants: [Android/Kotlin/variables.md](Android/Kotlin/variables.md)
- Funcions: [Android/Kotlin/funcions.md](Android/Kotlin/funcions.md)
- Classes i Objectes: [Android/Kotlin/classesobjectes.md](Android/Kotlin/classesobjectes.md)
- Null Safety: [Android/Kotlin/nullsafety.md](Android/Kotlin/nullsafety.md)
- Funcions d'extensió: [Android/Kotlin/fextensio.md](Android/Kotlin/fextensio.md)
- LiveData: [Android/Kotlin/livedata.md](Android/Kotlin/livedata.md)
- Coroutines: [Android/Kotlin/coroutines.md](Android/Kotlin/coroutines.md)
- Anotadors: [Android/Kotlin/anotadors.md](Android/Kotlin/anotadors.md)

## Introducció

Kotlin és un llenguatge de programació modern creat per JetBrains i pensat, inicialment, per ser compatible amb Java. 

Avui en dia és el llenguatge recomanat per Android i també s’utilitza en molts altres àmbits. La seva filosofia principal és ser un llenguatge més simple, segur i productiu que els que existien fins al moment, especialment comparat amb Java.

Algunes característiques generals:

### Simplicitat i llegibilitat

- Kotlin busca reduir el codi que cal escriure. Moltes operacions que en altres llenguatges requereixen diverses línies, en Kotlin es resolen de forma concisa.
- Això facilita que el codi sigui més fàcil d’entendre i mantenir.

### Seguretat

- Kotlin incorpora mecanismes per evitar errors típics, sobretot els relacionats amb valors nuls.
- Aquest tipus d’error és molt habitual en programació i en Kotlin es controla de manera explícita.

### Compatibilitat amb Java

- Kotlin funciona sobre la JVM (Java Virtual Machine), i un projecte pot barrejar fitxers Java i Kotlin sense problemes.
- A més, pot utilitzar totes les llibreries i frameworks que existeixen en Java.

Això permet migrar de Java a Kotlin de forma progressiva.

### Versatilitat

Tot i que va néixer orientat a aplicacions Android, avui Kotlin s’utilitza en molts camps:

- Aplicacions mòbils (Android)
- Backend (servidors) amb frameworks com **Ktor** o **Spring Boot**
- Aplicacions d'escriptori
- Desenvolupament web (Kotlin/JS)
- Multiplataforma (Kotlin Multiplatform), compartint codi entre Android, iOS i altres sistemes

### Més eficient:

Alguns motius pels quals Kotlin ajuda a treballar de forma més eficient són:

- Més facilitat per escriure codi que fa el mateix amb menys línies
- Millor integració amb l’entorn Android Studio
- Sistema de tipus modern i més expressiu
- Funcions especials pensades per simplificar tasques habituals