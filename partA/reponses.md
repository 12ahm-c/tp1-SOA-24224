# TP1 SOA — Partie A : Code-first

## Étape 1 — Service

1. `@WebService` expose `EmpruntService` comme service web JAX-WS. `@WebFault` déclare `LivreIndisponibleException` comme faute métier dans le contrat.
2. Le constructeur vide permet à JAXB (utilisé par JAX-WS) de créer un objet pendant la désérialisation XML, puis de renseigner ses propriétés.

## Étape 2 — WSDL

1. L'URL du service est `http://localhost:8080/bib/EmpruntService`. Les opérations sont `emprunterLivre` (entrée : `idAdherent`, `isbn`; sortie : `emprunt` avec `numero` et `dateRetour`) et `rendreLivre` (entrée : `numeroEmprunt`; sortie : `penalite`).
2. Le binding généré par Metro est document/literal. La version SOAP utilisée par le binding généré est SOAP 1.1.
3. `isbn` et `dateRetour` sont des `xsd:string`, car ils sont déclarés comme `String` en Java. La validation regex de l'ISBN n'est pas exposée automatiquement dans le schéma généré.
4. Les paramètres de l'opération sont généralement non qualifiés dans la requête wrapped générée. Je reprends les noms et espaces de noms exactement comme indiqués dans le XML SoapUI produit par le WSDL.
5. HTTP transporte la requête et la réponse (méthode, en-têtes et statut). XML représente les données. SOAP définit l'enveloppe, l'en-tête, le corps et les fautes. Le WSDL décrit le contrat : opérations, messages, types, binding et adresse du service.

## Étape 3 — Messages SOAP

1. `POST` transporte le message SOAP complet dans le corps HTTP, même lorsque l'opération consulte ou modifie une information.
2. Le bloc `jeton` n'est pas traité par cette implémentation : aucun handler ne le lit. Sans `mustUnderstand="1"`, son absence de traitement ne bloque pas l'appel.
3. Le service attend le binding SOAP 1.1 généré. Un message SOAP 1.2 n'est donc pas conforme à ce binding et doit être rejeté avec une faute de version/binding.
4. Les corrections du message fourni sont : utiliser `http://supnum.mr/bib` pour `bib`, renommer l'argument `numero` en `numeroEmprunt`, employer la forme d'espace de noms des paramètres montrée dans SoapUI, placer `Header` avant `Body`, et conserver l'opération et le nom de service attendus. Dans l'exemple, le mauvais espace de noms d'opération et le mauvais nom d'argument empêchent la résolution correcte de l'appel; un en-tête inconnu non marqué `mustUnderstand` peut être ignoré. L'ordre Header/Body est requis par SOAP.

## Étape 4 — Faults

1. ISBN invalide : faute SOAP 1.1 de code `Client`, avec le texte `ISBN mal formé : 978-2-10` et un statut HTTP d'erreur (souvent 500 avec JAX-WS). Livre déjà emprunté : faute déclarée `LivreIndisponible`, texte `Livre indisponible`, également transportée comme erreur HTTP.
2. Le `detail` de la faute métier contient les données `LivreIndisponibleInfo`, notamment l'ISBN et la date `retourPrevu`. Sa structure est publiée dans les types XSD du WSDL généré.
3. `LivreIndisponibleException` est une faute déclarée. Son code SOAP concret dépend du runtime JAX-WS/Metro et du mapping de la faute; sémantiquement, le livre indisponible n'est pas une faute de syntaxe du client. Je le traiterais comme une faute métier déclarée plutôt que comme une faute technique.
4. `mustUnderstand="1"` sur un en-tête que le service ne comprend pas déclenche une faute `MustUnderstand`. Avec `"0"`, le bloc peut être ignoré.
5. SOAP 1.1 exprime la faute avec `faultcode`, `faultstring` et éventuellement `detail`. SOAP 1.2 utilise `Code/Value`, `Reason/Text` et `Detail`, avec des codes tels que `Sender` et `Receiver`.

## Résumé des fichiers

- `xml/01-emprunter.xml` : demande SOAP 1.1 valide avec l'en-tête `jeton`.
- `xml/02-emprunter-12.xml` : même demande avec enveloppe SOAP 1.2 pour l'essai de compatibilité.
- `xml/03-rendre-corrige.xml` : exemple de retour corrigé.
- `xml/04-fault-sender-12.xml` : exemple de Fault SOAP 1.2 pour ISBN invalide.

