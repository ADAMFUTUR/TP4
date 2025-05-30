## TP4 - Web Service SOAP avec JAX-WS

Ce projet illustre la création et la consommation d'un service web basé sur le protocole SOAP en utilisant Java et l'API standard Jakarta XML Web Services (JAX-WS). L'objectif pédagogique principal est de familiariser l'étudiant avec les concepts fondamentaux des services web SOAP, la publication d'un endpoint de service, la génération d'un client à partir d'un WSDL, et l'invocation des opérations du service distant. Le scénario applicatif choisi est celui d'un service bancaire simplifié, offrant des opérations de conversion de devises et de consultation de comptes.

### Structure du Projet

Le projet est organisé sous forme d'un projet Maven multi-modules pour séparer clairement le serveur et le client. Le répertoire racine `ws-soap/ws-soap/` contient la configuration Maven parente (`pom.xml`) ainsi que le code source du serveur de service web.

Le code source du serveur se trouve dans `src/main/java/`. La classe `ServerJWS.java` sert de point d'entrée pour démarrer et publier le service web. Elle utilise `jakarta.xml.ws.Endpoint.publish` pour rendre le service accessible sur une URL spécifiée (par défaut `http://0.0.0.0:9090/`). Voici l'extrait pertinent de `ServerJWS.java` :

```java
import jakarta.xml.ws.Endpoint;
import ws.BanqueService;

public class ServerJWS {
    public static void main(String[] args) {
        String url = "http://0.0.0.0:9090/";
        // Publication du service web à l'URL spécifiée
        Endpoint.publish(url, new BanqueService());
        System.out.println("Web service déployé sur "+url);
    }
}
```

L'implémentation concrète du service est définie dans la classe `ws.BanqueService.java`, qui contient la logique métier des opérations exposées. Le modèle de données utilisé par le service, ici la classe `ws.Compte.java`, représente la structure d'un compte bancaire.

Un sous-module Maven nommé `client-soap-java` est dédié au client qui consommera le service web. Ce module possède son propre `pom.xml` héritant du parent et contient le code source du client dans `src/main/java/`. La classe principale du client est `net.adam.Main.java`. Elle démontre comment interagir avec le service web distant. Pour ce faire, elle s'appuie sur des classes dites "proxy" situées dans le package `proxy/`. Ces classes sont typiquement générées automatiquement à partir du fichier WSDL (Web Services Description Language) du service. Dans ce projet, les classes proxy sont déjà fournies.

### Prérequis

Pour compiler et exécuter ce projet, vous aurez besoin des outils suivants installés sur votre système :

*   Un Kit de Développement Java (JDK), version 22 ou une version ultérieure compatible.
*   Apache Maven, pour la gestion des dépendances et la construction du projet.

Le projet utilise JAX-WS, dont l'implémentation de référence est ajoutée comme dépendance Maven dans le fichier `pom.xml` parent :

```xml
<dependencies>
    <!-- https://mvnrepository.com/artifact/com.sun.xml.ws/jaxws-ri -->
    <dependency>
        <groupId>com.sun.xml.ws</groupId>
        <artifactId>jaxws-ri</artifactId>
        <version>4.0.2</version>
        <type>pom</type>
    </dependency>
</dependencies>
```

### Compilation et Exécution

La compilation du projet s'effectue à l'aide de Maven. Ouvrez un terminal, naviguez jusqu'au répertoire racine du projet (`ws-soap/ws-soap/`) et exécutez la commande :

```bash
mvn clean install
```

Cette commande nettoiera les artefacts de construction précédents, compilera les sources du serveur et du client, et installera les artefacts dans votre dépôt Maven local.

Pour lancer le serveur de service web, vous devez exécuter la classe `ServerJWS`. Après avoir compilé le projet, vous pouvez le faire depuis votre Environnement de Développement Intégré (IDE) ou en ligne de commande depuis le répertoire `ws-soap/ws-soap/` :

```bash
mvn exec:java -Dexec.mainClass="ServerJWS"
```

Une fois lancé, le serveur affichera un message indiquant l'URL sur laquelle le service est déployé (par exemple, "Web service déployé sur http://0.0.0.0:9090/").

Le client SOAP (`client-soap-java`) utilise les classes proxy pour communiquer avec le serveur. Si ces classes n'étaient pas fournies, il faudrait les régénérer avec l'outil `wsimport` (inclus dans le JDK) en utilisant l'URL du WSDL du service publié :

```bash
# Commande à exécuter depuis le répertoire client-soap-java
wsimport -s src/main/java -p proxy http://localhost:9090/?wsdl
```

Pour exécuter le client, assurez-vous que le serveur est en cours d'exécution. Lancez la classe `net.adam.Main` depuis votre IDE ou via Maven depuis le répertoire racine (`ws-soap/ws-soap/`) :

```bash
mvn exec:java -pl client-soap-java -Dexec.mainClass="net.adam.Main"
```

Le client effectuera alors des appels au service web et affichera les résultats dans la console.

### Fonctionnalités du Service

Le service `BanqueService` expose plusieurs opérations simples, annotées avec `@WebMethod` pour les rendre accessibles via SOAP. L'annotation `@WebService` marque la classe comme une implémentation de service web. Voici un aperçu de la classe `BanqueService.java` :

```java
package ws;

import jakarta.jws.WebMethod;
import jakarta.jws.WebParam;
import jakarta.jws.WebService;
import java.util.List;
import java.util.Date;

@WebService(serviceName = "BanqueWS")
public class BanqueService {

    @WebMethod(operationName = "ConversionEuroToDH")
    public double conversion(@WebParam(name = "montant") double mt){
        // Logique de conversion simple
        return mt * 11;
    }

    @WebMethod
    public Compte getCompte(@WebParam(name = "code") int code) {
        // Retourne un compte fictif
        return new Compte(code, Math.random() * 50000, new Date());
    }

    @WebMethod
    public List<Compte> getComptes() {
        // Retourne une liste de comptes fictifs
        return List.of(
                new Compte(1, Math.random() * 50000, new Date()),
                new Compte(2, Math.random() * 50000, new Date()),
                new Compte(3, Math.random() * 50000, new Date())
        );
    }
}
```

Le client `Main.java` démontre l'utilisation de ces opérations en interagissant avec le proxy du service. Il instancie d'abord le service et obtient un port proxy, puis appelle les méthodes distantes :

```java
package net.adam;

import proxy.BanqueService;
import proxy.BanqueWS;
import proxy.Compte;

public class Main {
    public static void main(String[] args) {
        // Création du proxy pour interagir avec le service
        BanqueService proxy = new BanqueWS().getBanqueServicePort();

        // Appel de l'opération de conversion
        System.out.println("Conversion 10 EUR -> DH: " + proxy.conversionEuroToDH(10));

        // Appel de l'opération getCompte
        Compte compte = proxy.getCompte(4);
        System.out.println("--- Détails Compte 4 ---");
        System.out.println("Code: " + compte.getCode());
        System.out.println("Solde: " + compte.getSolde());
        System.out.println("Date Création: " + compte.getDateCreation());

        // Appel de l'opération getComptes
        System.out.println("--- Liste des Comptes ---");
        proxy.getComptes().forEach(cp -> {
            System.out.println("  Code: " + cp.getCode());
            System.out.println("  Solde: " + cp.getSolde());
            System.out.println("  Date Création: " + cp.getDateCreation());
            System.out.println("  ----------");
        });
    }
}
```

### Technologies Utilisées

Ce projet met en œuvre les technologies et standards suivants :

*   **Java:** Langage de programmation principal.
*   **JAX-WS (Jakarta XML Web Services):** API Java standard pour la création de services web SOAP.
*   **SOAP (Simple Object Access Protocol):** Protocole d'échange de messages basé sur XML pour les services web.
*   **WSDL (Web Services Description Language):** Langage basé sur XML pour décrire les services web.
*   **Maven:** Outil de gestion de projet et de construction (build).

