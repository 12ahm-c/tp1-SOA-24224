# TP1 SOA — Partie B : Contract-first

## Étape 5 — Génération depuis le WSDL

1. L'opération est `emprunterLivre`. Elle déclare la faute `LivreIndisponible`, portée par le message `LivreIndisponibleFault`. L'action SOAP à envoyer est `http://supnum.mr/bib/emprunterLivre`.
2. La signature produite par `wsimport` suit le contrat : `EmpruntType emprunterLivre(String idAdherent, String isbn) throws LivreIndisponibleFault` (les annotations et noms qualifiés sont générés par l'outil).
3. `dateRetour` devient `XMLGregorianCalendar`, car le WSDL le déclare `xs:date`. En partie A, le champ Java `String` était exposé comme `xsd:string`.
4. La faute nommée dans le contrat est `LivreIndisponible`; `wsimport` génère une exception Java correspondante, généralement `LivreIndisponibleFault`. Le nom vient du `wsdl:fault`/message de faute.

## Étape 6 — Implémentation et comparaison

1. L'implémentation pointe sur le WSDL fourni via `wsdlLocation`. Avec `endpointInterface`, Metro publie le contrat associé à l'interface générée. Le schéma contient le motif ISBN et déclare `dateRetour` en `xs:date`.
2. Oui, la requête doit respecter le schéma du contrat et l'action SOAP indiquée. Ici, les paramètres sont qualifiés dans l'espace `http://supnum.mr/bib` (`elementFormDefault="qualified"`); `xml/01-emprunter.xml` est préparé ainsi.
3. Sans validation de schéma, un ISBN mal formé peut atteindre la méthode Java. Avec `@SchemaValidation`, la validation à l'entrée rejette l'ISBN avant l'appel métier et renvoie une faute de validation SOAP.
4. Sans `wsdlLocation`, Metro construit un WSDL à partir de l'interface et des annotations disponibles. Le contrat publié peut alors différer du fichier fourni, notamment pour les types et les restrictions de schéma.

## Conclusion

1. Le WSDL de la partie A est généré depuis le Java et expose `dateRetour` comme chaîne; celui de B est fourni d'abord, contient explicitement la restriction ISBN et déclare `dateRetour` comme date XML.
2. En code-first, renommer le champ peut modifier le contrat généré et casser les clients qui utilisent l'ancien nom. En contract-first, le contrat demeure la référence : le code doit continuer à respecter le WSDL.
3. Pour Paiement, je choisirais contract-first, car plusieurs équipes et langages doivent partager un contrat stable. Pour un prototype interne de deux jours, code-first est plus rapide.

