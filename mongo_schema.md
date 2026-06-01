# Analyse de la base de données MongoDB — Marlodjinfra

> Généré le 2026-06-01  
> 31 collections · Express.js + Mongoose

---

## Vue d'ensemble

| # | Collection | Description |
|---|-----------|-------------|
| 1 | `advertisements` | Publicités (médias, texte, bannières) pour TV et kiosques |
| 2 | `agenceguichetagents` | Affectations agents ↔ guichets par agence |
| 3 | `agenceguichets` | Liaisons guichets ↔ services par agence |
| 4 | `agences` | Agences/branches bancaires |
| 5 | `agenceservices` | Services disponibles par agence |
| 6 | `agenceservicetypeoperations` | Types d'opération par agence+service |
| 7 | `agencyconfigs` | Configuration planning/rendez-vous par agence |
| 8 | `appointments` | Rendez-vous (agents → clients) |
| 9 | `appointmentreasons` | Motifs de rendez-vous |
| 10 | `appointmentrequests` | Demandes de rendez-vous (clients → agents) |
| 11 | `blockedslots` | Créneaux bloqués (réunions, pauses, congés) |
| 12 | `clients` | Clients finaux (auth par OTP) |
| 13 | `clientotps` | Codes OTP temporaires (TTL auto-expiry) |
| 14 | `counters` | Compteurs de tickets par agence/service |
| 15 | `customforms` | Formulaires personnalisés dynamiques |
| 16 | `guichets` | Guichets (caisses) |
| 17 | `motifs` | Motifs de mise en attente / rejet |
| 18 | `notificationlogs` | Logs des notifications (email/SMS/WhatsApp/FCM) |
| 19 | `onlinerequests` | Demandes en ligne (type chatbot/formulaire) |
| 20 | `onlinerequesttypes` | Types de demandes en ligne |
| 21 | `priorityqueueitems` | File d'attente prioritaire |
| 22 | `requestmessages` | Messages entre agent et client (demandes en ligne) |
| 23 | `requestratelimits` | Rate limiting des requêtes (TTL) |
| 24 | `reservations` | **Collection principale** — tickets de file d'attente |
| 25 | `satisfactionsurveys` | Enquêtes de satisfaction post-service |
| 26 | `services` | Services offerts |
| 27 | `televisions` | Écrans d'affichage (TV en agence) |
| 28 | `ticketreservationlogs` | Logs d'appels API externe (tickets Marlodj) |
| 29 | `typeoperationconfigs` | Config type d'opération par agence/guichet |
| 30 | `typeoperations` | Types d'opérations bancaires |
| 31 | `users` | Staff (agents, superviseurs, admins…) |

---

## Collections détaillées

### 1. `advertisements`

| Champ | Type | Requis | Défaut | Contraintes |
|-------|------|--------|--------|-------------|
| `firebaseId` | String | Non | — | unique, sparse |
| `adType` | enum | Oui | — | `media` / `text` / `banner_text` |
| `mediaType` | enum | Conditionnel | — | `video` / `image` / `audio` / `text` / `document` — requis si adType=media |
| `mimeType` | String | Non | `''` | trim |
| `title` | String | Oui | — | trim |
| `description` | String | Non | `''` | trim |
| `textContent` | String | Conditionnel | `''` | requis si adType=text ou banner_text |
| `textStyle` | enum | Non | `default` | `default` / `bold` / `italic` / `highlight` |
| `bannerDirection` | enum | Non | `left` | `left` / `right` |
| `bannerSpeed` | Number | Non | `60` | min: 1 |
| `bannerPosition` | enum | Non | `bottom` | `top` / `bottom` |
| `bannerLoop` | Boolean | Non | `true` | — |
| `bannerBackgroundColor` | String | Non | `''` | trim |
| `bannerTextColor` | String | Non | `''` | trim |
| `fileUrl` | String | Non | `''` | trim |
| `firebaseStoragePath` | String | Non | `''` | trim |
| `storageProvider` | enum | Non | `firebase` | `firebase` / `cloudinary` |
| `cloudinaryPublicId` | String | Non | `''` | trim |
| `filename` | String | Non | `''` | trim |
| `originalName` | String | Non | `''` | trim |
| `fileType` | String | Non | `''` | trim |
| `fileSize` | Number | Non | `0` | min: 0, max: 1 073 741 824 |
| `startDate` | Date | Oui | — | — |
| `endDate` | Date | Oui | — | — |
| `isActive` | Boolean | Non | `true` | — |
| `displayOrder` | Number | Non | `0` | min: 0 |
| `targetAudience` | enum | Non | `all` | `all` / `agents` / `clients` |
| `targetPlatforms` | [enum] | Non | `['mobile_app','kiosk','tv']` | `mobile_app` / `kiosk` / `tv` |
| `targetAgencies` | [Number] | Non | `[]` | — |
| `targetTelevisions` | [ObjectId] | Non | `[]` | ref: Television |
| `viewCount` | Number | Non | `0` | min: 0 |
| `clickCount` | Number | Non | `0` | min: 0 |
| `createdBy` | String | Oui | — | — |
| `updatedBy` | String | Non | — | — |
| `createdAt` / `updatedAt` | Date | Auto | — | timestamps |

---

### 2. `agenceguichetagents`

| Champ | Type | Requis | Défaut | Contraintes |
|-------|------|--------|--------|-------------|
| `agenceId` | Number | Oui | — | ref: Agence |
| `guichetId` | Number | Oui | — | ref: Guichet |
| `userId` | ObjectId | Oui | — | ref: User |
| `actif` | Boolean | Non | `true` | index |
| `isPrincipal` | Boolean | Non | `false` | — |
| `dateDebut` | Date | Oui | `Date.now` | — |
| `dateFin` | Date | Non | `null` | null = affectation permanente |
| `horaires[].jour` | enum | Oui | — | `lundi` … `dimanche` |
| `horaires[].heureDebut` | String | Oui | — | format HH:MM |
| `horaires[].heureFin` | String | Oui | — | format HH:MM |
| `noteInterne` | String | Non | `''` | — |
| `createdAt` / `updatedAt` | Date | Auto | — | timestamps |

**Index unique :** `(agenceId, guichetId, userId)` ; index partiel unique `(agenceId, guichetId)` pour `actif=true`

---

### 3. `agenceguichets`

| Champ | Type | Requis | Défaut | Contraintes |
|-------|------|--------|--------|-------------|
| `agenceId` | Number | Oui | — | ref: Agence |
| `serviceId` | Number | Oui | — | ref: Service |
| `guichetId` | Number | Oui | — | ref: Guichet |
| `actif` | Boolean | Non | `true` | index |
| `configuration.capaciteOverride` | Number | Non | `null` | — |
| `configuration.prioriteOverride` | Number | Non | `null` | — |
| `configuration.noteInterne` | String | Non | `''` | — |
| `createdAt` / `updatedAt` | Date | Auto | — | timestamps |

**Index unique :** `(agenceId, guichetId)`

---

### 4. `agences`

| Champ | Type | Requis | Défaut | Contraintes |
|-------|------|--------|--------|-------------|
| `id` | Number | Oui | — | unique (ID externe Marlodj) |
| `nomAgence` | String | Oui | — | trim |
| `adresse` | String | Oui | — | trim |
| `codeAgence` | String | Oui | — | trim |
| `pays` | String | Oui | — | trim |
| `regionId` | Number | Non | `null` | — |
| `regionName` | String | Non | — | trim |
| `longitude` | Number | Oui | — | — |
| `latitude` | Number | Oui | — | — |
| `structureId` | Number | Oui | — | — |
| `structureName` | String | Oui | — | trim |
| `typeStructureName` | String | Oui | — | trim |
| `nbClientByDay` | Number | Non | `null` | — |
| `status` | String | Non | `null` | — |
| `capacites` | Number | Oui | — | min: 0 |
| `telephone` | Number | Non | `0` | — |
| `heureDemarrage` | String | Oui | — | format HH:MM |
| `heureFermeture` | String | Oui | — | format HH:MM |
| `suspensionActivite` | Boolean | Non | `false` | — |
| `activationReservation` | Boolean | Non | `true` | — |
| `nombreLimitReservation` | Number | Oui | — | min: 0 |
| `distanceEnMetre` | Number | Non | `0` | — |
| `gfaStatus` | Boolean | Non | `false` | — |
| `clientsEnAttente` | Number | Non | `0` | min: 0 |
| `operationModels` | [Mixed] | Non | `[]` | — |
| `serviceIds` | [Number] | Non | `[]` | index |
| `operationIds` | [Number] | Non | `[]` | index |
| `createdAt` / `updatedAt` | Date | Auto | — | timestamps |

---

### 5. `agenceservices`

| Champ | Type | Requis | Défaut | Contraintes |
|-------|------|--------|--------|-------------|
| `agenceId` | Number | Oui | — | ref: Agence |
| `serviceId` | Number | Oui | — | ref: Service |
| `actif` | Boolean | Non | `true` | index |
| `ordre` | Number | Non | `0` | — |
| `isPrioritaire` | Boolean | Non | `false` | index |
| `configuration.dureeEstimeeOverride` | Number | Non | `null` | — |
| `configuration.prioriteOverride` | Number | Non | `null` | — |
| `configuration.noteInterne` | String | Non | `''` | — |
| `dateActivation` | Date | Non | `Date.now` | — |
| `dateDesactivation` | Date | Non | `null` | — |
| `createdAt` / `updatedAt` | Date | Auto | — | timestamps |

**Index unique :** `(agenceId, serviceId)`

---

### 6. `agenceservicetypeoperations`

| Champ | Type | Requis | Défaut | Contraintes |
|-------|------|--------|--------|-------------|
| `agenceId` | Number | Oui | — | ref: Agence |
| `serviceId` | Number | Oui | — | ref: Service |
| `typeOperationId` | Number | Oui | — | ref: TypeOperation |
| `actif` | Boolean | Non | `true` | index |
| `ordre` | Number | Non | `0` | — |
| `isPrioritaire` | Boolean | Non | `false` | index |
| `configuration.dureeEstimeeOverride` | Number | Non | `null` | — |
| `configuration.requiresAppointment` | Boolean | Non | `false` | — |
| `configuration.noteInterne` | String | Non | `''` | — |
| `dateActivation` | Date | Non | `Date.now` | — |
| `dateDesactivation` | Date | Non | `null` | — |
| `createdAt` / `updatedAt` | Date | Auto | — | timestamps |

**Index unique :** `(agenceId, serviceId, typeOperationId)`

---

### 7. `agencyconfigs`

| Champ | Type | Requis | Défaut | Contraintes |
|-------|------|--------|--------|-------------|
| `agencyId` | Number | Oui | — | unique, ref: Agence |
| `slotDuration` | Number | Non | `30` | min: 5 (minutes) |
| `bufferTimeBetweenAppointments` | Number | Non | `0` | min: 0 |
| `maxAppointmentsPerDay` | Number | Non | `16` | min: 1 |
| `maxConsecutiveAppointments` | Number | Non | `3` | min: 1 |
| `minAdvanceBooking` | Number | Non | `2` | min: 0 (heures) |
| `maxAdvanceBooking` | Number | Non | `14` | min: 1 (jours) |
| `workingDays` | [Number] | Non | `[1,2,3,4,5]` | valeurs 0–6 (dim–sam) |
| `workingHours.start` | String | Oui | — | format HH:MM |
| `workingHours.end` | String | Oui | — | format HH:MM |
| `workingHours.lunchBreak.start` | String | Non | — | format HH:MM |
| `workingHours.lunchBreak.end` | String | Non | — | format HH:MM |
| `holidays` | [String] | Non | `[]` | format YYYY-MM-DD |
| `serviceTypes` | Map | Non | `{}` | configuration par type de service |
| `createdAt` / `updatedAt` | Date | Auto | — | timestamps |

---

### 8. `appointments`

| Champ | Type | Requis | Défaut | Contraintes |
|-------|------|--------|--------|-------------|
| `firebaseId` | String | Non | — | unique, sparse |
| `agenceId` | String | Oui | — | — |
| `agenceName` | String | Non | — | trim |
| `agentId` | String | Oui | — | — |
| `agentName` | String | Non | — | trim |
| `clientName` | String | Oui | — | trim |
| `clientEmail` | String | Oui | — | trim, lowercase |
| `clientPhone` | String | Oui | — | trim |
| `date` | String | Oui | — | format YYYY-MM-DD |
| `timeSlot` | String | Oui | — | trim |
| `duration` | Number | Oui | `30` | min: 1 (minutes) |
| `subject` | String | Oui | — | trim |
| `notes` | String | Non | `''` | trim |
| `status` | enum | Non | `pending` | `pending` / `confirmed` / `called` / `rejected` / `cancelled` / `completed` |
| `rejectionReason` | String | Non | — | trim |
| `createdBy` | String | Oui | — | — |
| `updatedBy` | String | Non | — | — |
| `confirmedAt` | Date | Non | — | — |
| `createdAt` / `updatedAt` | Date | Auto | — | timestamps |

---

### 9. `appointmentreasons`

| Champ | Type | Requis | Défaut | Contraintes |
|-------|------|--------|--------|-------------|
| `firebaseId` | String | Non | — | unique, sparse |
| `name` | String | Oui | — | trim |
| `description` | String | Non | `''` | trim |
| `isActive` | Boolean | Non | `true` | — |
| `createdBy` | String | Oui | — | — |
| `updatedBy` | String | Non | — | — |
| `createdAt` / `updatedAt` | Date | Auto | — | timestamps |

---

### 10. `appointmentrequests`

| Champ | Type | Requis | Défaut | Contraintes |
|-------|------|--------|--------|-------------|
| `clientId` | ObjectId | Oui | — | ref: Client, index |
| `agenceId` | Number | Oui | — | ref: Agence |
| `serviceId` | Number | Non | — | ref: Service |
| `typeOperationId` | Number | Non | — | ref: TypeOperation |
| `agentId` | ObjectId | Non | — | ref: User |
| `date` | String | Oui | — | format YYYY-MM-DD |
| `timeSlot` | String | Oui | — | trim |
| `reasonId` | ObjectId | Non | — | ref: AppointmentReason |
| `notes` | String | Non | `''` | trim |
| `status` | enum | Non | `pending` | `pending` / `approved` / `called` / `rejected`, index |
| `decidedBy` | ObjectId | Non | — | ref: User |
| `decidedAt` | Date | Non | — | — |
| `rejectionReason` | String | Non | — | trim |
| `createdAt` / `updatedAt` | Date | Auto | — | timestamps |

---

### 11. `blockedslots`

| Champ | Type | Requis | Défaut | Contraintes |
|-------|------|--------|--------|-------------|
| `firebaseId` | String | Non | — | unique, sparse |
| `agentId` | String | Oui | — | — |
| `date` | String | Oui | — | format YYYY-MM-DD |
| `startTime` | String | Oui | — | format HH:MM |
| `endTime` | String | Oui | — | format HH:MM |
| `allDay` | Boolean | Non | `false` | — |
| `type` | enum | Oui | — | `meeting` / `lunch` / `training` / `personal` / `other` |
| `reason` | String | Oui | — | trim |
| `notes` | String | Non | `''` | trim |
| `recurring` | Boolean | Non | `false` | — |
| `recurrencePattern` | enum | Conditionnel | — | requis si recurring=true : `daily` / `weekly` / `monthly` / `yearly` |
| `recurrenceEnd` | String | Conditionnel | — | requis si recurring=true, format YYYY-MM-DD |
| `createdBy` | String | Oui | — | — |
| `updatedBy` | String | Non | — | — |
| `createdAt` / `updatedAt` | Date | Auto | — | timestamps |

---

### 12. `clients`

| Champ | Type | Requis | Défaut | Contraintes |
|-------|------|--------|--------|-------------|
| `fullName` | String | Oui | — | trim, minlength: 2 |
| `primaryIdentifier` | String | Oui | — | unique, index |
| `identifierType` | enum | Oui | — | `phone` / `email`, index |
| `phoneNumber` | String | Non | — | format international (+XXX…) |
| `email` | String | Non | — | trim, lowercase |
| `preferences` | Map | Non | `{}` | Map de valeurs mixtes |
| `fcmTokens[].token` | String | Oui | — | — |
| `fcmTokens[].device` | String | Non | `unknown` | — |
| `fcmTokens[].updatedAt` | Date | Non | `Date.now` | — |
| `actif` | Boolean | Non | `true` | — |
| `tokenVersion` | Number | Non | `0` | min: 0 (révocation JWT) |
| `lastLogin` | Date | Non | — | — |
| `createdAt` / `updatedAt` | Date | Auto | — | timestamps |

**Validation :** au moins `phoneNumber` ou `email` requis ; cohérence avec `primaryIdentifier`

---

### 13. `clientotps`

| Champ | Type | Requis | Défaut | Contraintes |
|-------|------|--------|--------|-------------|
| `identifier` | String | Oui | — | trim, index |
| `identifierType` | enum | Oui | — | `phone` / `email`, index |
| `phone` | String | Non | — | trim, sparse (legacy) |
| `code` | String | Oui | — | exactement 5 chiffres |
| `expiresAt` | Date | Oui | — | **TTL index** (auto-suppression) |
| `attempts` | Number | Non | `0` | min: 0 |
| `consumed` | Boolean | Non | `false` | index |
| `createdAt` / `updatedAt` | Date | Auto | — | timestamps |

**Index composé :** `(identifier, identifierType, consumed, expiresAt)`

---

### 14. `counters`

| Champ | Type | Requis | Défaut | Contraintes |
|-------|------|--------|--------|-------------|
| `_id` | String | Oui | — | clé custom, ex: `ticket_simple_agence_35` |
| `sequence_value` | Number | Oui | `0` | — |
| `createdAt` / `updatedAt` | Date | Auto | `Date.now` | timestamps |

---

### 15. `customforms`

| Champ | Type | Requis | Défaut | Contraintes |
|-------|------|--------|--------|-------------|
| `firebaseId` | String | Non | — | unique, sparse |
| `name` | String | Oui | — | trim |
| `description` | String | Non | `''` | trim |
| `fields[].id` | String | Oui | — | — |
| `fields[].type` | enum | Oui | — | `text` / `email` / `tel` / `number` / `date` / `textarea` / `select` / `checkbox` / `radio` / `file` |
| `fields[].label` | String | Oui | — | trim |
| `fields[].placeholder` | String | Non | — | trim |
| `fields[].helpText` | String | Non | — | trim |
| `fields[].required` | Boolean | Non | `false` | — |
| `fields[].order` | Number | Oui | — | min: 0 |
| `fields[].options` | [{label, value}] | Non | `[]` | pour select/radio/checkbox |
| `fields[].validation` | Object | Non | `{}` | min, max, pattern, message |
| `associatedReasons` | [String] | Non | `[]` | ref: AppointmentReason |
| `settings.requireAuthentication` | Boolean | Non | `true` | — |
| `settings.allowMultipleSubmissions` | Boolean | Non | `false` | — |
| `settings.sendConfirmationEmail` | Boolean | Non | `false` | — |
| `settings.autoSave` | Boolean | Non | `true` | — |
| `isActive` | Boolean | Non | `true` | — |
| `submissionCount` | Number | Non | `0` | min: 0 |
| `createdBy` | String | Oui | — | — |
| `updatedBy` | String | Non | — | — |
| `createdAt` / `updatedAt` | Date | Auto | — | timestamps |

---

### 16. `guichets`

| Champ | Type | Requis | Défaut | Contraintes |
|-------|------|--------|--------|-------------|
| `id` | Number | Oui | — | unique (ID externe Marlodj) |
| `label` | String | Oui | — | trim |
| `code` | String | Oui | — | unique, trim |
| `agenceId` | Number | Oui | — | ref: Agence, index |
| `serviceId` | Number | Oui | — | ref: Service |
| `structureId` | Number | Oui | `2` | — |
| `actif` | Boolean | Non | `true` | — |
| `capacite` | Number | Non | `1` | min: 1 |
| `priorite` | Number | Non | `0` | — |
| `createdAt` / `updatedAt` | Date | Auto | — | timestamps |

---

### 17. `motifs`

| Champ | Type | Requis | Défaut | Contraintes |
|-------|------|--------|--------|-------------|
| `code` | String | Oui | — | unique, uppercase, trim |
| `libelle` | String | Oui | — | trim |
| `type` | enum | Oui | `both` | `hold` / `reject` / `both` |
| `description` | String | Non | — | trim |
| `actif` | Boolean | Non | `true` | — |
| `createdBy` | ObjectId | Non | — | ref: User |
| `updatedBy` | ObjectId | Non | — | ref: User |
| `createdAt` / `updatedAt` | Date | Auto | — | timestamps |

---

### 18. `notificationlogs`

| Champ | Type | Requis | Défaut | Contraintes |
|-------|------|--------|--------|-------------|
| `appointmentId` | ObjectId | Oui | — | ref: Appointment, index |
| `clientPhone` | String | Oui | — | trim |
| `clientEmail` | String | Non | — | trim, lowercase |
| `clientName` | String | Non | — | trim |
| `notificationType` | enum | Oui | — | `created` / `confirmed` / `rejected` / `cancelled` / `updated` / `completed` / `reminder` / `client_called` |
| `channels.email` | Object | Non | `{}` | attempted, success, error, sentAt, messageId |
| `channels.sms` | Object | Non | `{}` | attempted, success, error, sentAt, messageId |
| `channels.whatsapp` | Object | Non | `{}` | attempted, success, error, sentAt, messageId, **limitReached** |
| `channels.fcm` | Object | Non | `{}` | attempted, success, error, sentAt, messageId |
| `overallStatus` | enum | Oui | — | `success` / `partial` / `failed` |
| `successfulChannels` | Number | Non | `0` | min: 0, max: 4 |
| `metadata` | Map | Non | `{}` | Map de valeurs mixtes |
| `createdAt` / `updatedAt` | Date | Auto | — | timestamps |

---

### 19. `onlinerequests`

| Champ | Type | Requis | Défaut | Contraintes |
|-------|------|--------|--------|-------------|
| `onlineRequestType` | ObjectId | Oui | — | ref: OnlineRequestType |
| `client` | ObjectId | Oui | — | ref: Client |
| `agent` | ObjectId | Non | `null` | ref: User |
| `status` | enum | Non | `PENDING` | `PENDING` / `ASSIGNED` / `IN_PROGRESS` / `COMPLETED` / `REJECTED` |
| `formData` | Mixed | Oui | — | données du formulaire soumis |
| `agency` | ObjectId | Non | — | ref: Agence |
| `lastMessage.text` | String | Non | `null` | — |
| `lastMessage.sender` | ObjectId | Non | — | refPath: senderModel |
| `lastMessage.senderModel` | enum | Non | — | `User` / `Client` |
| `lastMessage.timestamp` | Date | Non | — | — |
| `lastActivityAt` | Date | Non | `Date.now` | — |
| `createdAt` / `updatedAt` | Date | Auto | — | timestamps |

---

### 20. `onlinerequesttypes`

| Champ | Type | Requis | Défaut | Contraintes |
|-------|------|--------|--------|-------------|
| `name` | String | Oui | — | trim |
| `description` | String | Non | — | trim |
| `form` | ObjectId | Oui | — | ref: CustomForm |
| `isActive` | Boolean | Non | `true` | — |
| `createdBy` | ObjectId | Oui | — | ref: User |
| `createdAt` / `updatedAt` | Date | Auto | — | timestamps |

---

### 21. `priorityqueueitems`

| Champ | Type | Requis | Défaut | Contraintes |
|-------|------|--------|--------|-------------|
| `agenceId` | Number | Oui | — | ref: Agence, index |
| `serviceId` | Number | Non | — | ref: Service |
| `typeOperationId` | Number | Non | — | ref: TypeOperation |
| `clientRef.clientId` | ObjectId | Non | — | ref: Client |
| `clientRef.name` | String | Oui | — | trim |
| `clientRef.phone` | String | Oui | — | trim |
| `status` | enum | Non | `waiting` | `waiting` / `served` / `cancelled`, index |
| `servedAt` | Date | Non | — | — |
| `cancelledAt` | Date | Non | — | — |
| `servedBy` | ObjectId | Non | — | ref: User |
| `cancelledBy` | ObjectId | Non | — | ref: User |
| `notes` | String | Non | `''` | trim |
| `createdAt` / `updatedAt` | Date | Auto | — | timestamps |

---

### 22. `requestmessages`

| Champ | Type | Requis | Défaut | Contraintes |
|-------|------|--------|--------|-------------|
| `onlineRequest` | ObjectId | Oui | — | ref: OnlineRequest |
| `sender` | ObjectId | Oui | — | refPath: senderModel |
| `senderModel` | enum | Oui | — | `User` / `Client` |
| `message` | String | Non | — | trim |
| `attachmentUrl` | String | Non | — | — |
| `status` | enum | Non | `sent` | `sending` / `sent` / `delivered` / `read` |
| `readBy[].userId` | ObjectId | Non | — | refPath: senderModel |
| `readBy[].readAt` | Date | Non | `Date.now` | — |
| `createdAt` / `updatedAt` | Date | Auto | — | timestamps |

---

### 23. `requestratelimits`

| Champ | Type | Requis | Défaut | Contraintes |
|-------|------|--------|--------|-------------|
| `key` | String | Oui | — | unique, trim |
| `scope` | String | Oui | — | trim, index |
| `count` | Number | Oui | `0` | min: 0 |
| `windowStart` | Date | Oui | — | — |
| `expiresAt` | Date | Oui | — | **TTL index** (expires: 0) |

---

### 24. `reservations` — Collection principale

| Champ | Type | Requis | Défaut | Contraintes |
|-------|------|--------|--------|-------------|
| `externalReservationId` | String | Non | — | sparse, index |
| `agenceId` | Number | Oui | — | ref: Agence, index |
| `serviceId` | Number | Oui | — | ref: Service, index |
| `typeOperationId` | Number | Non | — | ref: TypeOperation |
| `clientInfo.clientId` | Number | Non | — | ID externe |
| `clientInfo.name` | String | Non | — | trim |
| `clientInfo.primaryContact` | String | Oui | — | trim, index |
| `clientInfo.contactType` | enum | Oui | — | `phone` / `email`, index |
| `clientInfo.phone` | String | Non | — | trim, index |
| `clientInfo.email` | String | Non | — | trim, lowercase |
| `clientInfo.internalClientId` | ObjectId | Non | — | ref: Client |
| `status` | enum | Non | `initiated` | `initiated` / `waiting` / `delayed` / `called` / `treated` / `rejected` / `passed`, index |
| `isPriority` | Boolean | Non | `false` | index |
| `ticketNumber` | String | Non | — | sparse, index |
| `appointmentTime` | String | Non | — | — |
| `queuePosition` | Number | Non | — | — |
| `estimatedWaitTime` | Number | Non | — | — |
| `externalResponse` | Object | Non | `{}` | réponse brute API Marlodj |
| `errorDetails.hasError` | Boolean | Non | — | — |
| `errorDetails.retryCount` | Number | Non | — | — |
| `statusHistory` | Array | Non | `[]` | historique des transitions de statut |
| `assignedGuichetId` | Number | Non | — | ref: Guichet |
| `assignedAgentId` | ObjectId | Non | — | ref: User |
| `calledAt` | Date | Non | — | — |
| `treatedAt` | Date | Non | — | — |
| `rejectedAt` | Date | Non | — | — |
| `passedAt` | Date | Non | — | — |
| `priorityQueueItemId` | ObjectId | Non | — | ref: PriorityQueueItem |
| `syncStatus` | enum | Non | `pending` | `pending` / `synced` / `synced_with_error` / `failed`, index |
| `externalTicketNumber` | String | Non | — | sparse |
| `externalSyncAttempts` | Number | Non | `0` | — |
| `lastSyncError` | Object | Non | — | message, code, timestamp |
| `ticketSource` | enum | Non | `web` | `kiosk` / `mobile` / `web`, index |
| `arrivalStatus` | enum | Non | `not_arrived` | `not_arrived` / `checked_in` / `no_show`, index |
| `checkedInAt` | Date | Non | — | — |
| `gfa.ticketId` | String | Non | — | données sync GFA |
| `gfa.tempsAttente` | Number | Non | — | — |
| `gfa.heureAppel` | String | Non | — | — |
| `gfa.caisse` | String | Non | — | — |
| `gfa.lastSyncAt` | Date | Non | — | — |
| `createdAt` / `updatedAt` | Date | Auto | — | timestamps |

---

### 25. `satisfactionsurveys`

| Champ | Type | Requis | Défaut | Contraintes |
|-------|------|--------|--------|-------------|
| `reservationId` | ObjectId | Oui | — | ref: Reservation, **unique** |
| `clientId` | ObjectId | Oui | — | ref: Client |
| `agenceId` | Number | Oui | — | ref: Agence, index |
| `agentId` | ObjectId | Non | — | ref: User, index |
| `serviceId` | Number | Non | — | ref: Service, index |
| `typeOperationId` | Number | Non | — | ref: TypeOperation, index |
| `reservationStatus` | enum | Oui | — | `treated` / `rejected` |
| `status` | enum | Non | `pending` | `pending` / `completed` / `expired`, index |
| `rating` | Number | Non | `null` | min: 1, max: 5 |
| `comment` | String | Non | — | trim, maxlength: 500 |
| `expiresAt` | Date | Oui | — | — |
| `respondedAt` | Date | Non | — | — |
| `createdAt` / `updatedAt` | Date | Auto | — | timestamps |

---

### 26. `services`

| Champ | Type | Requis | Défaut | Contraintes |
|-------|------|--------|--------|-------------|
| `id` | Number | Oui | — | unique (ID externe Marlodj) |
| `nomService` | String | Oui | — | trim |
| `description` | String | Non | — | trim |
| `codeService` | String | Non | — | trim |
| `actif` | Boolean | Non | `true` | — |
| `dureeEstimee` | Number | Non | `0` | min: 0 (minutes) |
| `priorite` | Number | Non | `0` | — |
| `typeStructureId` | Number | Non | `1` | — |
| `parentId` | Number | Non | `null` | hiérarchie de services |
| `isToday` | Boolean | Non | `true` | — |
| `nombreDossierTraiter` | Number | Non | `0` | — |
| `daughterServiceId` | [Number] | Non | `[]` | sous-services |
| `createdAt` / `updatedAt` | Date | Auto | — | timestamps |

---

### 27. `televisions`

| Champ | Type | Requis | Défaut | Contraintes |
|-------|------|--------|--------|-------------|
| `firebaseId` | String | Non | — | unique, sparse |
| `agenceId` | Number | Oui | — | ref: Agence, index |
| `code` | String | Oui | — | trim |
| `label` | String | Oui | — | trim |
| `location` | String | Non | `''` | trim |
| `isActive` | Boolean | Non | `true` | index |
| `resolution` | enum | Non | `1920x1080` | `1280x720` / `1920x1080` / `2560x1440` / `3840x2160` |
| `fcmTokens[].token` | String | Oui | — | — |
| `fcmTokens[].device` | String | Non | `android_tv` | — |
| `fcmTokens[].updatedAt` | Date | Non | `Date.now` | — |
| `createdBy` | String | Oui | — | — |
| `updatedBy` | String | Non | — | — |
| `createdAt` / `updatedAt` | Date | Auto | — | timestamps |

**Index unique composé :** `(agenceId, code)`

---

### 28. `ticketreservationlogs`

| Champ | Type | Requis | Défaut | Contraintes |
|-------|------|--------|--------|-------------|
| `externalReservationId` | String | Oui | — | unique, trim, index |
| `agenceId` | Number | Oui | — | ref: Agence, index |
| `serviceId` | Number | Non | — | ref: Service |
| `typeOperationId` | Number | Non | — | ref: TypeOperation |
| `clientName` | String | Non | — | trim |
| `clientPhone` | String | Non | — | trim |
| `rawPayload` | Mixed | Non | `{}` | payload brut de l'API |
| `priorityDetected` | Boolean | Non | `false` | index |
| `createdAt` / `updatedAt` | Date | Auto | — | timestamps |

---

### 29. `typeoperationconfigs`

| Champ | Type | Requis | Défaut | Contraintes |
|-------|------|--------|--------|-------------|
| `typeOperationId` | Number | Oui | — | ref: TypeOperation |
| `agenceId` | Number | Oui | — | ref: Agence |
| `serviceId` | Number | Oui | — | ref: Service |
| `guichetId` | Number | Non | — | ref: Guichet |
| `actif` | Boolean | Non | `true` | index |
| `createdAt` / `updatedAt` | Date | Auto | — | timestamps |

**Index composé :** `(typeOperationId, agenceId, serviceId)`

---

### 30. `typeoperations`

| Champ | Type | Requis | Défaut | Contraintes |
|-------|------|--------|--------|-------------|
| `id` | Number | Oui | — | unique (ID externe Marlodj) |
| `nomTypeOperation` | String | Oui | — | trim |
| `description` | String | Non | — | trim |
| `codeTypeOperation` | String | Oui | — | unique, trim |
| `actif` | Boolean | Non | `true` | — |
| `serviceId` | Number | Oui | — | ref: Service, index |
| `dureeEstimee` | Number | Non | `0` | min: 0 (minutes) |
| `ordre` | Number | Non | `0` | — |
| `createdAt` / `updatedAt` | Date | Auto | — | timestamps |

---

### 31. `users`

| Champ | Type | Requis | Défaut | Contraintes |
|-------|------|--------|--------|-------------|
| `username` | String | Oui | — | unique, trim, minlength: 3 |
| `email` | String | Oui | — | unique, trim, lowercase |
| `password` | String | Oui | — | minlength: 8, **hashé bcrypt** |
| `role` | enum | Oui | `agent` | `admin` / `superviseur` / `gestionnaire` / `agent` / `commercial` / `tresorier` / `analyste` |
| `actif` | Boolean | Non | `true` | — |
| `tokenVersion` | Number | Non | `0` | min: 0 (révocation JWT) |
| `lastLogin` | Date | Non | — | — |
| `agenceIds` | [Number] | Non | `[]` | ref: Agence (agences assignées) |
| `guichetIds` | [Number] | Non | `[]` | ref: Guichet (guichets assignés) |
| `agenceId` | Number | Non | — | ref: Agence (agence principale) |
| `firstName` | String | Non | — | trim |
| `lastName` | String | Non | — | trim |
| `displayName` | String | Non | — | trim |
| `phoneNumber` | String | Non | — | trim |
| `telephone` | String | Non | — | trim |
| `adresse` | String | Non | — | trim |
| `firstConnection` | Boolean | Non | `true` | — |
| `emailConfirmed` | Boolean | Non | `false` | — |
| `phoneNumberConfirmed` | Boolean | Non | `false` | — |
| `resetPasswordToken` | String | Non | — | — |
| `resetPasswordExpires` | Date | Non | — | — |
| `oldBackendId` | String | Non | — | unique, sparse (migration legacy) |
| `createdAt` / `updatedAt` | Date | Auto | — | timestamps |

**Hook pre-save :** hachage bcrypt du mot de passe · **Virtual :** relation `agence` via `agenceId`

---

## Statistiques globales

| Métrique | Valeur |
|----------|--------|
| Nombre total de collections | **31** |
| Collections avec timestamps automatiques | 28 |
| Collections avec TTL (auto-suppression) | 2 (`clientotps`, `requestratelimits`) |
| Collections avec `firebaseId` (sync Firebase) | 7 |
| Collections utilisant des IDs numériques externes (Marlodj API) | 16 |
| Collection la plus complexe | `reservations` (~35 champs, multi-index) |
| Rôles utilisateur | 7 (`admin`, `superviseur`, `gestionnaire`, `agent`, `commercial`, `tresorier`, `analyste`) |
| Canaux de notification | 4 (email, SMS, WhatsApp, FCM) |

## Relations principales

```
Agence (1) ──── (N) AgenceService ──── (N) Service
Agence (1) ──── (N) AgenceGuichet ──── (N) Guichet
Agence (1) ──── (N) AgenceGuichetAgent ──── (N) User
Service (1) ──── (N) TypeOperation
Reservation (N) ──── (1) Agence
Reservation (N) ──── (1) Service
Reservation (N) ──── (1) Client
Reservation (1) ──── (1) SatisfactionSurvey
OnlineRequest (1) ──── (N) RequestMessage
OnlineRequest (N) ──── (1) OnlineRequestType ──── (1) CustomForm
Appointment (1) ──── (N) NotificationLog
```
