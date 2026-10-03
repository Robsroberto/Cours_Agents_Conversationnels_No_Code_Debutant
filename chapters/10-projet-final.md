## Brief du projet : définir les objectifs et le persona  

### Choisir le secteur d’activité  

Prenons comme fil conducteur **« Boutique Kora »,** une petite boutique de vêtements située à Dakar qui vend des tenues traditionnelles et modernes en ligne. Le même modèle s’applique à une agence de voyage à Abidjan ou à un centre de formation à Kigali : il suffit d’adapter le catalogue, les prix et les canaux de contact.  

### Objectifs mesurables  

| Objectif | Indicateur | Cible (3 mois) |
|----------|------------|----------------|
| Générer des ventes via le chatbot | % de sessions aboutissant à un paiement | 12 % |
| Réduire le nombre de tickets manuels | Tickets créés par le bot / total tickets | 70 % |
| Améliorer la satisfaction client | Score CSAT post‑conversation | ≥ 4,5/5 |

Ces KPI seront suivis dans le tableau de bord du chapitre 9 (voir **chapitre 9**).  

---

## Architecture du chatbot complet  

### Composants no‑code  

| Composant | Rôle | Exemple d’outil |
|-----------|------|-----------------|
| Interface conversationnelle | Widget visible par le client | Landbot, Chatfuel |
| Moteur IA génératif | Génération de réponses contextuelles | OpenAI GPT‑4 (via API) |
| Base de connaissances métier | FAQ, catalogue, politiques | Google Sheet ou Airtable |
| Orchestrateur d’automatisations | Création de tickets, mise à jour CRM | Make (Integromat) ou Zapier |
| Stockage sécurisé des clés | Gestion des secrets | 1Password, Vault de Make |

Toutes ces briques sont reliées par des **webhooks** ou des **connecteurs natifs** fournis par la plateforme choisie.  

### Flux de données et intégrations  

1. **Produit** : le bot interroge une API REST du catalogue (ex. : `https://api.boutiquekora.com/products`).  
2. **Paiement** : redirection vers un checkout Stripe ou PayDunya via un lien dynamique généré par le bot.  
3. **CRM** : chaque nouvelle commande crée ou met à jour un contact dans HubSpot (ou Odoo).  
4. **Ticketing** : les demandes non résolues sont envoyées à Freshdesk ou à un groupe WhatsApp dédié.  

Le schéma suivant résume le parcours :  

```
User → Widget → Prompt → OpenAI → Réponse
                ↓               ↓
          Webhook API      Webhook CRM
                ↓               ↓
          Mise à jour DB   Création ticket
```  

---

## Construction du prompt maître  

### Structurer le contexte métier  

Le **prompt maître** doit contenir :  

* Une description concise de la boutique et de son ton (ex. : « amical, professionnel, en français et en wolof »).  
* Les règles de priorité (ex. : toujours proposer le paiement en XOF, ne jamais divulguer le prix sans confirmation de la taille).  
* Un petit extrait de la base de connaissances (FAQ).  

```json
{
  "role": "system",
  "content": "Tu es l’assistant virtuel de Boutique Kora, une boutique de vêtements à Dakar. Tu réponds en français et, si le client le demande, en wolof. Tu proposes les produits du catalogue via l’API https://api.boutiquekora.com/products. Tu ne donnes jamais le prix sans demander la taille du vêtement. Tu encourages toujours le client à finaliser l’achat en lui proposant le lien de paiement Stripe en XOF."
}
```

Ce prompt est stocké dans la configuration du bloc **« OpenAI Request »** de Make ou dans le **Custom Prompt** de Landbot.  

### Gestion des variantes (langues, devise)  

Ajoutez un **paramètre de langue** dans le flux :  

```json
{
  "user_language": "{{contact.language}}",   // "fr" ou "wo"
  "currency": "XOF"
}
```

Le bot utilise alors un sous‑prompt conditionnel :  

```
{% if user_language == "wo" %}
  Réponds en wolof.
{% else %}
  Réponds en français.
{% endif %}
```  

Ces structures conditionnelles sont prises en charge par les **templates** de Make (Handlebars) ou par le **Logic Jump** de Landbot.  

---

## Mise en place du flux conversationnel avancé  

### Scénario d’achat complet  

1. **Accueil** : « Bonjour ! Que puis‑je faire pour vous aujourd’hui ? »  
2. **Identification du besoin** : le bot propose des catégories (T-shirts, Boubous, Accessoires).  
3. **Recherche produit** : appel à l’API catalogue avec les filtres choisis.  
4. **Présentation** : le bot envoie une carte contenant image, prix, tailles disponibles.  
5. **Confirmation de la taille** : le client choisit, le bot valide la disponibilité.  
6. **Création du lien de paiement** : appel à Stripe Checkout via webhook → URL retournée au client.  
7. **Confirmation** : le bot envoie le récapitulatif et propose de suivre la commande.  

Chaque étape est implémentée comme un **bloc** dans la plateforme no‑code, avec des **conditions** qui redirigent le flux en cas de réponse inattendue (ex. : « Je ne trouve pas ce produit », → proposer d’envoyer le besoin à un agent).  

### Gestion des retours et FAQ dynamique  

* **FAQ dynamique** : le bot interroge la feuille Google `FAQ_Kora` contenant les questions fréquentes.  
* **Retours produit** : le bot demande le numéro de commande, vérifie la date d’achat via l’API, puis propose les options (échange, remboursement).  

Exemple de requête Google Sheets via Make :  

```json
{
  "method": "GET",
  "url": "https://sheets.googleapis.com/v4/spreadsheets/{{sheetId}}/values/FAQ!A:B",
  "headers": {
    "Authorization": "Bearer {{access_token}}"
  }
}
```

Le résultat est mis en cache pendant 5 minutes pour éviter les appels excessifs.  

---

## Automatisation des tickets et escalade  

### Création de ticket avec Zapier/Make  

```json
{
  "method": "POST",
  "url": "https://api.freshdesk.com/v2/tickets",
  "headers": {
    "Authorization": "Basic {{base64_api_key}}",
    "Content-Type": "application/json"
  },
  "body": {
    "description": "{{conversation.history}}",
    "subject": "Ticket généré par le chatbot – {{contact.email}}",
    "email": "{{contact.email}}",
    "priority": 2,
    "status": 2
  }
}
```

Le ticket est automatiquement assigné à l’équipe « Support » et le client reçoit un mail de confirmation contenant le numéro du ticket.  

### Règles d’escalade vers un agent humain  

1. **Détection d’impasse** : si le bot ne trouve pas de réponse après 2 tentatives, il déclenche le webhook d’escalade.  
2. **Notification WhatsApp** : via l’API Business de WhatsApp, le bot envoie le résumé de la conversation à un numéro d’agent.  
3. **Transfert du contexte** : le texte complet de la session est stocké dans un champ `conversation_context` du CRM, accessible par l’agent.  

Ces règles sont configurées dans le **Scenario** de Make : `Router → Filter (no answer) → HTTP > WhatsApp > Update CRM`.  

---

## Déploiement sur les canaux  

### Intégration site web  

* **WordPress** : insérer le script fourni par Landbot dans le footer (`Appearance > Theme Editor > footer.php`).  
```html
<script src="https://cdn.landbot.io/landbot-3/landbot-3.0.0.js"></script>
<script>
  var myLandbot = new Landbot.Livechat({
    configUrl: 'https://chats.landbot.io/v3/H-1234567/index.json',
  });
</script>
```  
* **Shopify** : ajouter le même script dans `Online Store > Themes > Edit code > theme.liquid` avant `</body>`.  

### Messenger, WhatsApp Business, Instagram  

1. **Facebook Messenger** : connecter le bot à la page via le **Facebook Connector** de Landbot, activer le **Webhook** `messages` et mapper les réponses.  
2. **WhatsApp Business** : créer une **Business Account** via Meta, obtenir le **Phone Number ID**, puis configurer le webhook `messages` dans Make.  
3. **Instagram Direct** : activer la **Instagram Messaging API** dans le même tableau de bord Facebook et réutiliser le même scénario de réponse.  

Chaque canal conserve le même **prompt maître**, garantissant une expérience uniforme.  

---

## Checklist de lancement  

| Étape | Vérification | Responsable |
|-------|--------------|-------------|
| **Clé API OpenAI** sécurisée (env vars) | ✅ | Développeur |
| **Prompt maître** testé avec 5 requêtes | ✅ | Rédacteur IA |
| **Flux conversationnel** complet (scenario complet) | ✅ | Chef de projet |
| **Intégration paiement** (sandbox) | ✅ | Responsable e‑commerce |
| **Création ticket** fonctionnelle | ✅ | Support |
| **Test multicanal** (Web, Messenger, WhatsApp) | ✅ | QA |
| **Conformité RGPD** (consentement cookie, suppression données) | ✅ | Juriste |
| **Backup de la base de connaissances** (Google Sheet) | ✅ | Ops |
| **Monitoring** (alertes quota OpenAI) | ✅ | DevOps |

---

## Stratégies de promotion du chatbot  

### Campagne email & SMS  

* **Segmentation** : extraire les contacts qui n’ont jamais acheté et les cibler avec « Découvrez notre nouveau conseiller virtuel ».  
* **Message** : inclure un bouton « Chattez maintenant » qui ouvre le widget en plein écran.  
* **Suivi** : mesurer le taux d’ouverture et le taux de conversion via le lien UTM `?utm_source=email_chatbot`.  

### Partenariats locaux et influenceurs  

* **Influenceurs mode** à Dakar ou Abidjan : leur offrir un code promo à partager via le chatbot (`PROMO10`).  
* **Co‑branding** : créer une mini‑page « Chat avec l’expert » sur le site du partenaire, redirigée vers le même bot.  
* **Événement en ligne** : organiser un live Instagram où le présentateur montre comment le bot aide à choisir une tenue, incitant les spectateurs à tester en temps réel.  

Ces actions génèrent du trafic qualifié et augmentent le taux d’engagement du bot dès son lancement.  

---

## Plan d’évolution sur 6 mois  

### Analyse des KPI (voir chapitre 9)  

| Mois | KPI à surveiller | Action corrective |
|------|------------------|--------------------|
| 1‑2 | Taux de résolution | Enrichir la base FAQ, ajouter des exemples de prompts |
| 3‑4 | Temps moyen de réponse | Optimiser le nombre de tokens dans le prompt, passer à GPT‑4‑Turbo |
| 5‑6 | Conversion ventes | Implémenter des recommandations produit basées sur l’historique du client (via le CRM) |

### Ajout de nouvelles fonctionnalités  

| Fonctionnalité | Description | Priorité |
|----------------|-------------|----------|
| **Recommandations personnalisées** | Utiliser le modèle `gpt-4` pour suggérer des tenues complémentaires à partir du panier | Haute |
| **Paiement intégré** | Intégrer directement le SDK Stripe Checkout dans le widget (no‑code via Webhooks) | Moyenne |
| **Analyse sentiment** | Ajouter une étape de classification sentiment (positif/negatif) pour prioriser les escalades | Basse |
| **Multilingue avancé** | Ajouter le créole haïtien ou le swahili grâce à des prompts de traduction | Optionnel |

Le suivi mensuel de ces évolutions se fait via le tableau de bord de Make ou de Landbot, où chaque nouvelle version du scénario est versionnée.  

---

## Points clés  

- Un projet complet se construit en **définissant d’abord les objectifs business**, puis en traduisant ces objectifs en **flux conversationnels** et **intégrations**.  
- Le **prompt maître** centralise le ton, les règles métier et les données dynamiques ; il doit être stocké de façon sécurisée et être facilement modifiable.  
- Les **webhooks** et les **scénarios d’automatisation** (Make/Zapier) permettent de transformer chaque interaction en action concrète : création de ticket, mise à jour CRM, génération de lien de paiement.  
- Le **déploiement multicanal** garantit que le même assistant est accessible sur le site, les réseaux sociaux et les messageries mobiles, tout en respectant les spécificités de chaque plateforme.  
- Une **checklist de lancement** rigoureuse évite les oublis critiques (sécurité, conformité, tests).  
- La **promotion** du chatbot doit être intégrée à la stratégie marketing globale : email, SMS, influenceurs locaux et événements en ligne.  
- Le **plan d’évolution** repose sur les KPI mesurés (voir chapitre 9) et introduit progressivement des fonctionnalités à forte valeur ajout