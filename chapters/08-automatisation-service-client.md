## Créer un ticket automatiquement depuis le chatbot  

### Connexion du chatbot à Zapier / Make via webhook  

Le point d’entrée le plus simple pour déclencher une automatisation est le **webhook**.  
Dans votre outil no‑code (Landbot, Botpress Cloud, etc.) ajoutez un bloc « Appeler un webhook » dès que le bot détecte une intention « création de ticket ».  

```json
{
  "event": "create_ticket",
  "user_id": "{{user.id}}",
  "email": "{{user.email}}",
  "subject": "{{intent.subject}}",
  "message": "{{intent.message}}",
  "order_id": "{{session.order_id}}"
}
```

- `{{…}}` sont les variables dynamiques du bot.  
- Le webhook doit être configuré en **POST** et renvoyer un code 200 OK pour que le bot sache que le ticket a bien été enregistré.  

Dans Zapier, créez un Zap : **Webhook → Create Record** (Google Sheets, Airtable, ou votre base de tickets).  
Dans Make, ajoutez le module **Webhooks > Custom webhook** puis enchaînez avec **Google Sheets > Add a row** ou **Airtable > Create a record**.  

> **Astuce** : ajoutez un champ « statut » initialisé à `ouvert`. Cela vous permettra de le mettre à jour automatiquement lors de l’escalade.

### Exemple de scénario Zapier : ticket dans Google Sheets  

1. **Trigger** – *Catch Hook* : réception du JSON ci‑dessus.  
2. **Action** – *Google Sheets > Create Spreadsheet Row* : mappez chaque clé du JSON à une colonne (date, user_id, email, sujet, message, statut).  
3. **Action** – *Email by Zapier* (optionnel) : envoyez une confirmation au client (`Merci, votre ticket #{{row.id}} a été créé`).  

Ce flux ne nécessite aucune ligne de code ; tout se configure via l’interface drag‑and‑drop.  

---

## Gérer l’escalade vers un agent humain  

### Définir les critères d’escalade  

L’escalade doit être déclenchée dès que le bot ne parvient pas à résoudre le problème ou que le client attend trop longtemps. Trois critères courants :  

| Critère | Exemple de règle | Action déclenchée |
|---------|------------------|-------------------|
| **Sentiment négatif** | Analyse du texte avec l’API Sentiment d’OpenAI : score < ‑0.5 | Créer un ticket d’escalade |
| **Temps de réponse** | Aucun message du bot pendant > 2 min après la première requête | Créer un ticket d’escalade |
| **Intention “parler à un humain”** | L’utilisateur saisit « parler à un humain » | Créer un ticket d’escalade |

Dans Make, utilisez le module **OpenAI > Create Completion** avec le prompt :  

```
Analyse le sentiment du texte suivant et renvoie un score entre -1 (très négatif) et 1 (très positif) :
{{message}}
```  

Ensuite, ajoutez un filtre **Score < -0.5** pour poursuivre le scénario.

### Implémentation de l’escalade avec Zapier / Make  

1. **Trigger** – *Webhook* (ticket créé).  
2. **Action** – *OpenAI* (sentiment).  
3. **Filter** – *Only continue if* (score < ‑0.5 **ou** temps > 2 min **ou** intention = “human”).  
4. **Action** – *Slack > Send Channel Message* (alerte à l’équipe support).  
5. **Action** – *Update Row* (Google Sheets) : change le statut du ticket en `escalade`.  

Dans Make, le même flux se construit avec les modules suivants :  

- **Webhooks > Custom webhook** (recevoir le ticket).  
- **Tools > Router** (brancher selon les critères).  
- **OpenAI > Create Completion** (sentiment).  
- **Slack > Send a Message** (alerte).  
- **Google Sheets > Update a Row** (statut).  

Le routage vous permet d’ajouter d’autres branches (SMS, WhatsApp) sans dupliquer le scénario.

---

## Synchroniser les tickets avec un CRM (HubSpot, Zoho, Odoo)  

### Mapping des champs essentiels  

| Champ du ticket | Champ CRM correspondant | Remarque |
|-----------------|------------------------|----------|
| `user_id` | `Contact ID` | Créez le contact s’il n’existe pas. |
| `email` | `Email` | Champ obligatoire dans tous les CRM. |
| `subject` | `Subject` | Titre du ticket. |
| `message` | `Description` | Corps du ticket. |
| `order_id` | `Custom field – Order ID` | Permet de relier le ticket à la commande. |
| `statut` | `Ticket Status` | Valeurs : `Open`, `In Progress`, `Escalated`, `Closed`. |

### Scénario Make : création d’un ticket HubSpot  

1. **Webhook** – réception du ticket du bot.  
2. **HubSpot > Find Contact** – recherche par email.  
   - Si le contact n’existe pas → **HubSpot > Create Contact**.  
3. **HubSpot > Create Ticket** – mappez les champs du JSON aux propriétés HubSpot.  
4. **HubSpot > Add Note to Ticket** – ajoutez le texte complet du message comme note.  
5. **Slack > Send Message** – notifiez l’équipe support avec le lien HubSpot du ticket (`{{ticket.id}}`).  

#### Exemple de mapping JSON → HubSpot (dans le module “Create Ticket”)  

```json
{
  "subject": "{{subject}}",
  "content": "{{message}}",
  "hs_pipeline": "0",               // pipeline support
  "hs_pipeline_stage": "1",         // stade “New”
  "hs_ticket_status": "{{statut}}",
  "custom_order_id": "{{order_id}}"
}
```

### Zoho Desk et Odoo : différences à connaître  

| Plateforme | Méthode d’insertion | Particularité |
|------------|---------------------|----------------|
| **Zoho Desk** | API REST `POST /tickets` | Nécessite un **authtoken** généré dans le portail Zoho. |
| **Odoo** | RPC XML‑RPC ou API JSON `/api/ticket` | Les modèles Odoo sont très flexibles ; pensez à créer un champ `x_order_id` dans le modèle `helpdesk.ticket`. |

Dans Zapier, vous trouverez des actions natives **Zoho Desk – Create Ticket** et **Odoo – Create Record**.  
Dans Make, utilisez le module **HTTP > Make a request** avec les en‑têtes d’authentification appropriés.

---

## Scénarios typiques d’automatisation  

### Réclamation produit  

1. Le client indique « Mon produit est défectueux ».  
2. Le bot capture le numéro de commande (`order_id`) et le motif (`subject = "Réclamation"`).  
3. Le webhook crée un ticket dans le CRM avec le statut `ouvert`.  
4. Si le sentiment du message est très négatif, le ticket passe immédiatement à `escalade` et une notification Slack est envoyée à l’équipe qualité.  

**Valeur ajoutée** : le client reçoit immédiatement un numéro de suivi (`#{{ticket.id}}`) et sait que son problème est en cours de traitement, même avant qu’un humain ne prenne le relais.  

### Suivi de commande  

1. L’utilisateur demande « Où en est ma livraison ? ».  
2. Le bot interroge l’API du transporteur (ex. : **Shippo**, **Kobo**).  
3. Si le statut est « en cours », le bot répond directement.  
4. Si le statut est « non trouvé » ou si le client indique « Je n’ai toujours rien reçu », le bot crée un ticket d’escalade : `subject = "Suivi de commande – problème"`.  
5. Le ticket est lié au contact et au numéro de commande dans le CRM, ce qui permet au service logistique de prendre le relais sans perte d’information.  

---

## Bonnes pratiques et maintenance continue  

### Monitoring des flux et logs  

- **Zapier** : activez le *Task History* et créez une alerte par email lorsqu’un zap échoue ; cela vous évite les tickets “fantômes”.  
- **Make** : utilisez le tableau de bord « Scénario Run » pour visualiser chaque étape, et ajoutez un module **Tools > Log** à chaque branche critique.  

Conservez les logs pendant **30 jours** minimum ; ils seront utiles pour diagnostiquer les problèmes de latence ou de dépassement de quota API.  

### Gestion des limites d’API  

- OpenAI impose un quota quotidien ; prévoyez un **fallback** : si le quota est dépassé, le bot envoie une réponse générique et crée un ticket manuel.  
- Les CRM (HubSpot, Zoho) ont des limites de **requests / secondes**. Utilisez le module **Delay** de Make (ex. : 200 ms) pour lisser les pics d’envoi.  

### Sécurité et conformité des données  

- Masquez les champs sensibles (numéro de carte bancaire, données personnelles) avant de les transmettre à Zapier/Make.  
- Activez le chiffrement TLS sur le webhook et stockez la clé API dans les *Secrets* de votre plateforme no‑code.  
- En Afrique francophone, respectez les exigences du **RGPD** et du **Loi sur la protection des données personnelles** (ex. : conservation limitée à 12 mois).  

### Test automatisé avant mise en production  

1. Créez un **environnement de test** (ex. : feuille Google Sheets “Tickets‑Test”).  
2. Simulez les trois scénarios (réclamation, suivi, escalade) avec des messages factices.  
3. Vérifiez que chaque ticket apparaît dans le CRM, que le statut change correctement, et que les notifications Slack sont reçues.  

Automatisez ces tests avec **Make > Schedule > Every day** : le scénario lit les lignes “test” du sheet, déclenche le webhook, puis compare le résultat attendu avec le statut réel.  

---

## À retenir  

- **Webhooks** sont le point d’entrée le plus simple pour créer des tickets : ils permettent à n’importe quel bot no‑code d’alimenter un flux d’automatisation.  
- L’**escalade** repose sur des critères mesurables : sentiment négatif, temps d’attente, ou demande explicite d’un humain. Un router ou un filtre dans Zapier/Make orchestre ces règles sans écrire de code.  
- La **synchronisation CRM** doit être pensée comme un mapping de champs ; chaque plateforme (HubSpot, Zoho, Odoo) possède ses spécificités d’authentification et de structure de ticket.  
- Les **scénarios typiques** (réclamation produit, suivi de commande) illustrent comment le chatbot, le système de tickets et le CRM forment une chaîne de valeur où chaque maillon est automatisé mais reste contrôlable.  
- **Monitoring**, **gestion des quotas** et **sécurité** sont indispensables pour garantir la continuité du service client et la conformité légale.  
- Un **processus de test automatisé** permet de valider chaque nouveau flux avant le déploiement en production, évitant ainsi les ruptures de service qui nuisent à la confiance des clients.