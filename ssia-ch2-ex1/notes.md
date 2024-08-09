## 2.1 Starting your first project

Le premier projet est également le plus petit. C'est une simple application exposant un point de terminaison (endpoint) REST que vous pouvez appeler pour recevoir une réponse. Ce projet est suffisant pour vous permettre d'apprendre les premières étapes du développement d'une application en utilisant Spring Security et Spring Boot. Il présente les bases de l'architecture de Spring Security pour l'authentification et l'autorisation.

````shell
curl -u user:pass http://localhost:8080/hello
````

Nous commençons à apprendre Spring Security en créant un projet vide et en le nommant **ssia-ch2-ex1**. Les seules dépendances dont vous avez besoin pour notre premier projet sont **spring-boot-starter-web** et **spring-boot-starter-security**, comme indiqué dans la liste 2.1. Après avoir créé le projet, assurez-vous d'ajouter ces dépendances à votre fichier **pom.xml**. L'objectif principal de ce projet est de voir le comportement d'une application configurée par défaut avec Spring Security. Nous voulons également comprendre quels composants font partie de cette configuration par défaut, ainsi que leur objectif.

Dans l'exemple suivant, la classe **HelloController** définit un contrôleur REST et un point de terminaison REST pour notre exemple.

````java
@RestController
public class HelloController {
    
    @GetMapping("/hello")
    public String hello() {
        return "Hello!";
    }
}
````

L'annotation **@RestController** enregistre le bean dans le contexte et indique à Spring que l'application utilise cette instance comme contrôleur web. De plus, l'annotation spécifie que l'application doit définir le corps de la réponse HTTP à partir de la valeur de retour de la méthode. L'annotation **@GetMapping** associe le chemin **/hello** à la méthode implémentée via une requête GET. Une fois que vous exécutez l'application, en plus des autres lignes dans la console, vous devriez voir quelque chose qui ressemble à :

`Using generated security password: 93a01cf0-794b-4b98-86ef-54860f36f7f3`

Chaque fois que vous exécutez l'application, elle génère un nouveau mot de passe et l'affiche dans la console, comme présenté dans l'extrait de code précédent. Vous devez utiliser ce mot de passe pour appeler n'importe quel point de terminaison (endpoint) de l'application avec l'authentification HTTP Basic. D'abord, essayons d'appeler le point de terminaison sans utiliser l'en-tête **Authorization** :

````shell
curl http://localhost:8080/hello
````

La réponse à l'appel est :
````json
{
  "timestamp":"2024-08-09T11:37:26.174+00:00",
  "status":401,
  "error":"Unauthorized",
  "message":"",
  "path":"/hello"
}
````

Le statut de la réponse est **HTTP 401 Unauthorized**. Nous nous attendions à ce résultat, car nous n'avons pas utilisé les identifiants corrects pour l'authentification. Par défaut, Spring Security s'attend au nom d'utilisateur par défaut (**user**) avec le mot de passe fourni (dans mon cas, celui commençant par **93a01**). Essayons à nouveau, mais cette fois avec les identifiants corrects :

````shell
curl -u user:ef84f072-cde5-4a2c-85cd-a120cf586353 http://localhost:8080/hello
````

La réponse à l'appel est :
````text
Hello!
````

**NOTE** Le code d'état **HTTP 401 Unauthorized** est un peu ambigu. En général, il est utilisé pour représenter un échec d'authentification plutôt qu'une erreur d'autorisation. Les développeurs l'utilisent dans la conception de l'application pour des cas tels que des identifiants manquants ou incorrects. Pour une erreur d'autorisation, on utiliserait probablement le statut **403 Forbidden**. En règle générale, un **HTTP 403** signifie que le serveur a identifié l'appelant de la requête, mais que celui-ci ne possède pas les privilèges nécessaires pour l'appel qu'il essaie de faire.

Une fois que nous envoyons les identifiants corrects, vous pouvez voir dans le corps de la réponse exactement ce que la méthode **HelloController** que nous avons définie précédemment renvoie.

**Appel du point de terminaison avec l'authentification HTTP Basic**  

Avec cURL, vous pouvez définir le nom d'utilisateur et le mot de passe HTTP Basic avec l'option **-u**. En coulisses, cURL encode la chaîne **\<username\>:\<password\>** en Base64 et l'envoie comme valeur de l'en-tête **Authorization**, précédée de la chaîne **Basic**.  
Avec cURL, il est probablement plus facile d'utiliser l'option **-u**. Mais il est également important de savoir à quoi ressemble la requête réelle. Essayons donc et créons manuellement l'en-tête **Authorization**.  
Dans un premier temps, prenez la chaîne **\<username\>:\<password\>** et encodez-la en Base64. Lorsque notre application envoie l'appel, nous devons savoir comment former la valeur correcte pour l'en-tête **Authorization**. Vous pouvez le faire en utilisant l'outil Base64 dans une console Linux. Vous pouvez également trouver une page web qui encode les chaînes en Base64, comme [https://www.base64encode.org](https://www.base64encode.org). Cet extrait montre la commande dans une console Linux ou Git Bash (le paramètre **-n** signifie qu'aucune nouvelle ligne ne doit être ajoutée) :

```bash
echo -n user:ef84f072-cde5-4a2c-85cd-a120cf586353 | base64
```

L'exécution de cette commande renvoie la chaîne encodée en Base64 suivante :
```
dXNlcjplZjg0ZjA3Mi1jZGU1LTRhMmMtODVjZC1hMTIwY2Y1ODYzNTM=
```

Vous pouvez maintenant utiliser la valeur encodée en Base64 comme valeur de l'en-tête **Authorization** pour l'appel. Cet appel devrait générer le même résultat que celui utilisant l'option **-u** :
```bash
curl -H "Authorization: Basic dXNlcjplZjg0ZjA3Mi1jZGU1LTRhMmMtODVjZC1hMTIwY2Y1ODYzNTM=" localhost:8080/hello
```

Le résultat de l'appel est :
```
Hello!
```


## 2.2 The big picture of Spring Security class design


### HTTP vs. HTTPS

Vous avez peut-être remarqué que dans les exemples présentés, j'utilise uniquement HTTP. En pratique, cependant, vos applications communiquent uniquement via HTTPS. Pour les exemples que nous discutons dans ce livre, les configurations liées à Spring Security ne sont pas différentes, que nous utilisions HTTP ou HTTPS. Nous ne configurerons pas HTTPS pour les points de terminaison dans les exemples afin que vous puissiez vous concentrer sur les exemples liés à Spring Security. Mais si vous le souhaitez, vous pouvez activer HTTPS pour n'importe lequel des points de terminaison, comme présenté dans cette section.

Il existe plusieurs façons de configurer HTTPS dans un système. Dans certains cas, les développeurs configurent HTTPS au niveau de l'application ; dans d'autres, ils peuvent utiliser un service mesh, ou choisir de configurer HTTPS au niveau de l'infrastructure. Avec Spring Boot, vous pouvez facilement activer HTTPS au niveau de l'application, comme vous l'apprendrez dans le prochain exemple de cette section.

Dans tous ces scénarios de configuration, vous avez besoin d'un certificat signé par une autorité de certification (CA). En utilisant ce certificat, le client qui appelle le point de terminaison sait si la réponse provient du serveur d'authentification et que personne n'a intercepté la communication. Vous pouvez acheter un tel certificat si vous en avez besoin. Si vous devez seulement configurer HTTPS pour tester votre application, vous pouvez générer un certificat auto-signé en utilisant un outil tel qu'OpenSSL (https://www.openssl.org/). Générons notre certificat auto-signé, puis configurons-le dans le projet :

```bash
openssl req -newkey rsa:2048 -x509 -keyout key.pem -out cert.pem -days 365
```

mot de passe : ggin
Country Name (2 letter code) [AU]:FR
State or Province Name (full name) [Some-State]:Vaucluse
Locality Name (eg, city) []:Malaucene
Organization Name (eg, company) [Internet Widgits Pty Ltd]:GGN
Organizational Unit Name (eg, section) []:NO
Common Name (e.g. server FQDN or YOUR name) []:Georges GINON
Email Address []:georgesginon@gmail.com


Après avoir exécuté la commande OpenSSL dans un terminal, un mot de passe et des informations sur votre CA vous seront demandés. Comme il s'agit seulement d'un certificat auto-signé pour un test, vous pouvez saisir n'importe quelle donnée ; assurez-vous simplement de vous souvenir du mot de passe. La commande génère deux fichiers : **key.pem** (la clé privée) et **cert.pem** (un certificat public). Nous utiliserons ces fichiers pour générer notre certificat auto-signé afin d'activer HTTPS. Dans la plupart des cas, le certificat est au format Public Key Cryptography Standards #12 (PKCS12). Plus rarement, nous utilisons un format Java KeyStore (JKS). Continuons notre exemple avec un format PKCS12 (pour une excellente discussion sur la cryptographie, je recommande *Real-World Cryptography* de David Wong [Manning, 2020]) :

```bash
openssl pkcs12 -export -in cert.pem -inkey key.pem -out certificate.p12 -name "certificate"
```

La deuxième commande que nous utilisons reçoit en entrée les deux fichiers générés par la première commande et produit le certificat auto-signé.

Notez que si vous exécutez ces commandes dans un shell Bash sur un système Windows, vous devrez peut-être ajouter **winpty** avant la commande :

```bash
winpty openssl req -newkey rsa:2048 -x509 -keyout key.pem -out cert.pem -days 365
winpty openssl pkcs12 -export -in cert.pem -inkey key.pem -out certificate.p12 -name "certificate"
```

Enfin, avec le certificat auto-signé, vous pouvez configurer HTTPS pour vos points de terminaison. Copiez le fichier **certificate.p12** dans le dossier **resources** du projet Spring Boot, et ajoutez les lignes suivantes à votre fichier **application.properties** :

```properties
server.ssl.key-store-type=PKCS12
server.ssl.key-store=classpath:certificate.p12
server.ssl.key-store-password=ggin
```

Le mot de passe (dans mon cas, **ggin**) a été demandé dans l'invite après l'exécution de la commande pour générer le certificat. C'est pourquoi vous ne le voyez pas dans la commande. Ajoutons maintenant un point de terminaison de test à notre application, puis appelons-le en utilisant HTTPS :

```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "Hello!";
    }
}
```

Si vous utilisez un certificat auto-signé, vous devez configurer l'outil que vous utilisez pour appeler le point de terminaison afin qu'il ignore la vérification de l'authenticité du certificat. Si l'outil vérifie l'authenticité du certificat, il ne le reconnaîtra pas comme authentique et l'appel échouera. Avec cURL, vous pouvez utiliser l'option **-k** pour ignorer la vérification de l'authenticité du certificat :

```bash
curl -k -u user:c200f648-5690-423f-a5cd-3ccdc5fd0428 https://localhost:8080/hello
```

La réponse à l'appel est :

```bash
Hello!
```

Rappelez-vous que même si vous utilisez HTTPS, la communication entre les composants de votre système n'est pas infaillible. J'ai souvent entendu des gens dire : "Je ne vais plus chiffrer ça. Je vais utiliser HTTPS !" Bien qu'il soit utile pour protéger la communication, HTTPS n'est qu'un des éléments de la sécurité d'un système. Traitez toujours la sécurité de votre système avec responsabilité et prenez soin de toutes les couches impliquées.
