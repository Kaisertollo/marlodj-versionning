# Cas de conversation fonctionnels — Assistant IA Mode Auto

## Création d'utilisateur

| Prompt | Comportement observé |
|---|---|
| `Créer un agent test.partiel@exemple.com Test Partiel à l'agence Ecoban` | Bloque avant écriture et demande de choisir l'agence dans la liste (« Agence demandée : « Ecoban ». Choisis une agence dans la liste. ») |
| `Créer un agent khadija.ba2@exemple.com Khadija Ba à l'agence UCAD` | Agence détectée et présélectionnée automatiquement (Ecobank UCAD), carte de création correctement remplie |
| Création sans numéro de téléphone renseigné | Le placeholder « Téléphone (ex. 77 123 45 67) » reste visible, la création reste possible |
| Création avec prénom ou nom manquant | La carte permet la correction manuelle avant confirmation |
| Modification manuelle de l'e-mail, du rôle, du téléphone ou de l'agence avant de confirmer | Les champs restent éditables jusqu'à la confirmation |
| Annulation d'une création en cours (bouton « Annuler ») | Aucun utilisateur n'est créé, la carte affiche « Création annulée » |
| Double clic sur « Confirmer la création » | Une seule création est effectuée, pas de doublon |
| Après confirmation réussie | Le mot de passe généré n'apparaît qu'à ce moment, jamais avant |

## Transfert / affectation en une phrase

| Prompt | Comportement observé |
|---|---|
| `Attribue la tablette du CSO n°2 à agent@exemple.com` | « tablette » et « CSO » correctement compris comme guichet / service Clientèle |
| `Affecte agent@exemple.com à la tablette 2 du CSO` | Idem, résolution correcte du vocabulaire métier |
| `Transfère fatou.sarr@exemple.com au guichet Clientele 2 de l'agence Ecobank Roume` | Agent, agence et guichet correctement résolus ; guichet occupé détecté avant toute écriture |

## Gestion des cas ambigus et erreurs

| Prompt | Comportement observé |
|---|---|
| `Affecte fatou.sarr@exemple.com au guichet 2` (sans agence précisée) | Demande explicitement de choisir l'agence, car plusieurs agences ont un « guichet 2 » |
| `Affecte Fatou Sarr au guichet 3 de l'agence UCAD` (nom sans e-mail) | Résout le nom unique sans ambiguïté, puis signale correctement l'absence de guichet 3 à cette agence (« Aucun guichet actif ne correspond à cette demande ») |
| `Affecte Awa Deme (fatou.sarr@exemple.com) au guichet 1 de l'agence UCAD` (nom et e-mail contradictoires) | Détecte la contradiction et tranche explicitement : « Le nom indiqué correspond à Awa Deme, mais l'email désigne Fatou Sarr. L'email est retenu. » |
| `Affecte fatou.sarr@exemple.com à l'agence UCAD` (sans guichet précisé) | Demande explicitement le guichet cible avec un exemple concret |
| E-mail totalement inconnu du système | Message d'erreur clair, aucun blocage silencieux |
| `Affecte Diop au guichet 2` (nom ambigu) | Propose une liste de tous les agents nommés Diop pour sélection explicite |

## Guichet occupé

| Prompt / action | Comportement observé |
|---|---|
| Transfert vers un guichet déjà occupé | Le système affiche systématiquement « Guichet occupé par [email]. Que faire ? » avec deux choix : « Remplacer sans réaffecter » / « Réaffecter la personne » |
| Choix « Remplacer sans réaffecter » | Dry-run puis confirmation requise avant toute écriture réelle |
| Annulation après affichage de l'aperçu | Aucune affectation n'est modifiée |

## Affectation après création (pronoms)

| Prompt | Comportement observé |
|---|---|
| Créer un utilisateur, puis : `Maintenant affecte-le au guichet 2 de l'agence UCAD` | Le pronom « le » a été correctement résolu vers le dernier utilisateur créé (Khadija Ba), sans redemander l'agent |

> Remarque : ce cas a échoué dans une autre tentative (voir rapport de bugs) — le comportement n'est pas garanti à 100 %, mais fonctionne dans les conditions ci-dessus.

## Sécurité et fiabilité

| Action | Comportement observé |
|---|---|
| Annuler une création | Aucun compte n'est créé en base |
| Annuler un aperçu de transfert | Aucune affectation n'est modifiée |
| Double-clic sur confirmation | Une seule opération est réellement exécutée |
| Utilisateur créé et confirmé (Khadija Ba) | Visible avec la bonne agence (Ecobank UCAD) après création |
