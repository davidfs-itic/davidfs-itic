
## 4. Anotadors
### Què són
Els anotadors són una característica del llenguatge Java (i Kotlin) que permeten afegir metadades al codi.

Els anotadors (o Annotations) són una forma de metadada que es pot afegir al codi font.

1. Metadada, la clau
La metadada és "data sobre data". En programació, un anotador no conté lògica ni executa cap acció per si mateix. Simplement etiqueta o descriu altres parts del codi, com classes, mètodes, variables o paràmetres.

2. Finalitat Principal
El seu propòsit principal és ser llegits i processats per eines o frameworks en temps de compilació, en temps d'execució, o durant la generació de documentació.

3. Com Funcionen?
Quan un anotador es posa sobre un element de codi, el framework que fa servir aquest codi pot:
    - Llegir-lo (Reflection): El framework (com Retrofit, Spring, o JUnit) inspecciona el codi durant l'execució i detecta la presència i el valor dels anotadors.

    - Modificar el Comportament: Basant-se en l'anotador, el framework canvia la manera com gestiona l'element.

**Exemples comuns en programació:**

- Retrofit: L'anotador @GET("ruta") li diu a la llibreria que "aquest mètode ha de generar una petició HTTP GET a la ruta especificada."

- Testing (JUnit): L'anotador @Test li diu a l'eina de proves que "aquest mètode és una prova i ha de ser executat."

- Android: L'anotador @Override li diu al compilador que "aquest mètode està substituint un mètode de la classe pare, per la qual cosa s'ha de comprovar la signatura."

### Anotadors de Retrofit

Es divideixen principalment en dues categories: els que defineixen el mètode HTTP i la URL i els que defineixen els paràmetres que s'envien amb aquesta petició.

#### Anotadors de Mètode HTTP i URL Base
Anotador	| Categoria |	Funció |	Exemple d'Ús
:-----------|:----------|:---------|:----------------
@GET	| HTTP	| Sol·licita dades d'un recurs especificat. | S'utilitza per obtenir (llegir) dades.	@GET("users/list")
@POST	| HTTP	| Envia dades a un recurs per crear-ne un de nou o processar-les. | 	@POST("users/create")
@PUT	| HTTP| 	S'utilitza per actualitzar (modificar completament) un recurs existent amb les dades proporcionades.	| @PUT("users/{id}")
@DELETE | 	HTTP	| Sol·licita l'eliminació d'un recurs especificat.| 	@DELETE("users/{id}")
@PATCH	| HTTP	| S'utilitza per aplicar modificacions parcials a un recurs.	| @PATCH("users/{id}")
@HEAD	| HTTP| 	Demana només les capçaleres de resposta sense el cos de la resposta.|  Útil per comprovar l'existència.	@HEAD("health")
@HTTP| 	HTTP	| Mètode més general per fer peticions HTTP sense els anotadors específics (@GET, @POST, etc.). Poc comú.| 	@HTTP(method = "DELETE", path = "users/{id}", hasBody = false)

#### Anotadors de Paràmetres (Com s'Envia la Informació)

**Per la URL**

Anotador	| Funció	| Descripció	|Exemple
:-----------|:----------|:--------------|:----------------
@Path	|Ruta dinàmica	|Substitueix un paràmetre entre claus {} a la URL de la petició. Obligatori per a rutes dinàmiques.	|@GET("items/{id}") fun getItem(@Path("id") itemId: Int)
@Query	|Paràmetre de consulta	|Afegeix un paràmetre a la URL després del signe d'interrogació (?). Útil per a filtres i paginació.	|@GET("items") fun filterItems(@Query("type") type: String) Genera: /items?type=Electronics
@QueryMap	|Consulta múltiple	|Permet enviar múltiples paràmetres de consulta utilitzant un mapa de Kotlin (Map<String, Any>).|	fun getItems(@QueryMap options: Map<String, String>)

**Per al Cos de la Petició (Dades enviades)**
Anotador	| Funció	| Descripció	|Exemple
:-----------|:----------|:--------------|:----------------
@Body|Cos de la petició|"Utilitzat en peticions @POST, @PUT o @PATCH. El paràmetre de la funció (un objecte Kotlin) es serialitza (normalment a JSON) i s'envia com a cos de la petició."|fun createItem(@Body item: ItemRequest)
@FormUrlEncoded|Format de formulari|Utilitzat juntament amb @Field quan s'envien dades amb el format tradicional application/x-www-form-urlencoded.|Cal usar-lo a la funció!
@Field|Camp de formulari|S'utilitza només en combinació amb @FormUrlEncoded. Envia dades com a parelles clau-valor en el cos.|"fun login(@Field(""user"") u: String, @Field(""pass"") p: String)"

**Per a Capçaleres (Metadata)**
Anotador	| Funció	| Descripció	|Exemple
:-----------|:----------|:--------------|:----------------
@Header|Capçalera dinàmica|Permet afegir una capçalera HTTP de manera dinàmica (el seu valor prové d'un paràmetre de la funció). Útil per a tokens d'autenticació.|"fun secureCall(@Header(""Authorization"") token: String)"
@Headers|Capçalera estàtica|Permet afegir capçaleres HTTP fixes directament a la funció sense que depenguin d'un paràmetre.|"@Headers(""Cache-Control: max-age=640000"") Cal usar-lo a la funció!"

**Per a Multipart (Fitxers)**
Anotador	| Funció	| Descripció	|Exemple
:-----------|:----------|:--------------|:----------------
Anotador,Funció,Descripció,Exemple
@Multipart|Fitxers múltiples|Indica que la petició és de tipus multipart i s'utilitza per pujar fitxers amb dades addicionals.|Cal usar-lo a la funció!
@Part|Part de dades/fitxer|Defineix una part individual d'una petició multipart. Pot ser un camp de dades simple o un fitxer (utilitzant RequestBody).|fun uploadFile(@Part file: RequestBody)
