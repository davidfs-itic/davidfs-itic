# Docker

Referències:

Docker en 14 minuts: [https://www.youtube.com/watch?v=6idFknRIOp4](https://www.youtube.com/watch?v=6idFknRIOp4)


Curs de docker més profund: [https://www.youtube.com/watch?v=CV_Uf3Dq-EU](https://www.youtube.com/watch?v=CV_Uf3Dq-EU)


## Comandes docker per recordar:

Realitzem aquestes comandes per posar a punt un servidor web amb ubuntu/apache2:

### 1. Baixar la imatge (docker pull)
Aquest pas descarrega els fitxers de la imatge des del registre oficial (Docker Hub) cap al teu ordinador local.

```Bash
docker pull ubuntu/apache2:2.4-22.04_beta
```

Per què? Així t'assegures de tenir sempre la mateixa versió (si és només per una prova, pots utilitzar :latest per obtenir la última versió)

### 2. Crear el contenidor (docker create)
Aquest pas configura el contenidor (li dona un nom, assigna ports, etc.) però no l'executa encara.

```Bash
docker create --name webserver -p 80:80 ubuntu/apache2:2.4-22.04_beta
```
- **--name el-meu-apache**: Assigna un nom personalitzat perquè no hagis d'utilitzar l'ID alfanumèric.
- **-p 80:80**: Connecta el port 80 del teu ordinador al port 80 del contenidor (on corre Apache). Podràs veure la web entrant a http://ipservidor .

### 3. Iniciar el contenidor (docker start)
Finalment, posem en marxa el contenidor que acabem de crear.

```Bash
docker start webserver
```

Verificació: Un cop executat, pots obrir el teu navegador i visitar http://ipservidor. Hauries de veure la pàgina de benvinguda d'Apache a Ubuntu.

Verificació 2: podem executar:

```bash
docker ps -a
```

I hauriem de veure tots els contenidors engegats:

```bash
root@server ~ $ docker ps -a
CONTAINER ID   IMAGE                            COMMAND                  CREATED          STATUS          PORTS                                                    NAMES
6dfce0d31eeb   ubuntu/apache2:2.4-22.04_beta    "apache2-foreground"     29 seconds ago   Up 18 seconds   0.0.0.0:80->80/tcp, [::]:80->80/tcp                      webserver
```


### 4. Entrar dins el contenidor (executar bash)

Per obrir una sessió de bash dins del teu contenidor d'Apache, executa:

```Bash
docker exec -it webserver bash
```
Els paràmetres són:

**exec**: Indica a Docker que vols executar una nova comanda dins d'un contenidor que ja s'està executant.

**-i (interactive)**: Manté l'entrada estàndard (STDIN) oberta encara que no estiguis connectat. Permet que el contenidor rebi el que escrius.

**-t (tty)**: Assigna una pseudoterminal. Això fa que sembli una terminal real amb colors, format de text i el prompt (el símbol root@id:/#).

**webserver**: És el nom que li vam posar al contenidor en el pas anterior.

**bash**: És el programa que volem obrir (l'intèrpret de comandes).

