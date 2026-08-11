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
# Prompts de transfert — avec utilisateurs réellement existants

## Utilisateurs confirmés en base (vérifiés via "Gestion des Utilisateurs")
- Khadija Ba — khadija.ba2@exemple.com — Ecobank UCAD — actuellement au GUICHET 2
- Fatou Sarr — fatou.sarr@exemple.com — Ecobank UCAD — sans guichet actuellement
- Ousmane Ndoye — ousmane.ndoye@exemple.com — Ecobank UCAD — sans guichet actuellement
- Modou Fall — nomseul@exemple.com — Ecobank UCAD — sans guichet actuellement
- Awa Deme — awa.deme@exemple.com — Ecobank UCAD — sans guichet actuellement
- Awa Deme — awa.deme2@exemple.com — Agence Principale — sans guichet actuellement

## État réel des guichets à Ecobank UCAD (vérifié via "Gestion des Assignations")
- GUICHET 2 (GIC_UCAD_1) → occupé par Khadija Ba
- caisse 2 → **vide** (libre)
- fsdgxgx (service Clientèle) → occupé par sokhna samb (agent existant hors périmètre de test)

---

## Prompts vers un guichet libre (transfert simple, aucun conflit attendu)
- Affecte fatou.sarr@exemple.com au guichet caisse 2 de l'agence Ecobank UCAD
- Transfère ousmane.ndoye@exemple.com vers le guichet caisse 2 de l'agence Ecobank UCAD
- Attribue le guichet caisse 2 à nomseul@exemple.com dans l'agence Ecobank UCAD

## Prompts vers un guichet occupé (déclenche le choix Remplacer / Réaffecter)
- Affecte fatou.sarr@exemple.com au guichet 2 de l'agence Ecobank UCAD
- Transfère ousmane.ndoye@exemple.com vers le guichet 2 de l'agence Ecobank UCAD
- Affecte nomseul@exemple.com au guichet Clientele de l'agence Ecobank UCAD

## Prompt déplaçant un utilisateur déjà affecté (Khadija Ba, actuellement au GUICHET 2)
- Transfère khadija.ba2@exemple.com vers le guichet caisse 2 de l'agence Ecobank UCAD
- Déplace khadija.ba2@exemple.com au guichet Clientele de l'agence Ecobank UCAD

## Prompt entre deux agences (Awa Deme, Agence Principale → Ecobank UCAD)
- Transfère awa.deme2@exemple.com vers l'agence Ecobank UCAD, guichet caisse 2

## Prompt avec nom seul (sans e-mail), en s'appuyant sur un nom unique confirmé
- Affecte Fatou Sarr au guichet caisse 2 de l'agence Ecobank UCAD
- Affecte Khadija Ba au guichet caisse 2 de l'agence Ecobank UCAD

## Prompt avec vocabulaire métier + utilisateur réel
- Affecte ousmane.ndoye@exemple.com à la tablette 2 du CSO à l'agence Ecobank UCAD
| Double-clic sur confirmation | Une seule opération est réellement exécutée |
| Utilisateur créé et confirmé (Khadija Ba) | Visible avec la bonne agence (Ecobank UCAD) après création |
