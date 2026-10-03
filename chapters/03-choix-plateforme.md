## Panorama des solutions no‑code du marché

Le paysage des plateformes « no‑code » pour créer un chatbot IA s’est enrichi ces dernières années. Quatre acteurs se démarquent aujourd’hui pour les entrepreneurs africains : **Landbot**, **Chatfuel**, **Make** (couplé à l’API OpenAI) et **Botpress Cloud**. Chacun propose une approche différente du design conversationnel, de la tarification et de la prise en charge des langues locales.  

| Plateforme | Modèle de facturation | Langues supportées | Connexion internet requise | Support local (Afrique) | Points forts pour l’Afrique |
|------------|----------------------|--------------------|----------------------------|--------------------------|-----------------------------|
| **Landbot** | Freemium : plan gratuit (limité à 100 conversations/mois) → Pro à 30 €/mois, Business à 80 €/mois | Français, Anglais, Portugais, Swahili (via API) | Toujours en ligne (hébergement cloud) | Partenaires à Dakar, Nairobi, Abidjan | Interface drag‑and‑drop très visuelle, widgets intégrables sur sites WordPress/Shopify, export CSV des logs |
| **Chatfuel** | Freemium (500 utilisateurs) → Pro à 15 €/mois, Premium à 300 €/mois (volume illimité) | Français, Anglais, plusieurs langues africaines via modules | En ligne (hébergement Facebook) | Communauté francophone active, support en français via tickets | Intégration native avec Messenger et Instagram, plugins WhatsApp via Twilio, automatisation simple des réponses |
| **Make + OpenAI** | Make : 0 €/mois (1000 opérations) → 9 €/mois (10 000 ops) ; OpenAI : paiement à l’usage (0,002 $/token) | Français, Anglais, langues via prompts personnalisés | En ligne, mais les scénarios peuvent être exécutés hors‑ligne via webhook local | Documentation en français, webinars ciblés Afrique | Flexibilité maximale : combiner n n modules (CRM, paiement mobile, SMS) ; facturation à l’usage très adaptée aux petits volumes |
| **Botpress Cloud** | Freemium (1 bot, 1000 messages) → Starter à 49 €/mois, Pro à 199 €/mois | Français, Anglais, langues via formation du modèle | En ligne, mais possibilité d’auto‑héberger la version open‑source (on‑prem) | Présence via partenaires en Afrique du Sud, support en français disponible | Gestion fine des intents/entités, conformité RGPD + possibilité d’héberger les données en Europe ou en Afrique (via serveur dédié) |

> **Remarque** : les tarifs indiqués sont ceux en vigueur au 1er octobre 2026 et peuvent varier selon les promotions ou les forfaits entreprise.  

---

## Critères de sélection adaptés aux PME africaines

### 1. Budget et modèle de facturation

- **Volume de conversations** : Si votre activité ne dépasse pas quelques centaines d’échanges mensuels, un plan gratuit (Landbot ou Chatfuel) suffit. Au-delà, comparez le coût par conversation (Landbot ≈ 0,30 €/100 conv., Chatfuel ≈ 0,10 €/100 conv.) avec le modèle à l’usage de Make + OpenAI (0,002 $/token, soit ~0,0015 €/conv. pour 100 tokens).  
- **Coût de la donnée** : Certaines plateformes facturent les exports de logs (Landbot) ou le stockage de fichiers (Botpress). Anticipez les besoins d’historisation pour la conformité locale (ex. : exigences de la CNIL‑Afrique).

### 2. Langues et capacité de localisation

- **Support natif** : Landbot et Chatfuel offrent le français dès le plan de base. Pour le swahili, le lingala ou le wolof, il faut passer par des prompts personnalisés (OpenAI) ou former des intents (Botpress).  
- **Gestion des caractères spéciaux** : Vérifiez que la plateforme accepte les caractères Unicode (ex. : « é », « ñ », « ߞ ») afin d’éviter les corruptions de texte dans les réponses.

### 3. Connectivité internet et robustesse

- **Infrastructure locale** : Dans les zones où la latence est élevée, privilégiez les solutions qui permettent le **caching** des réponses (ex. : Botpress on‑prem) ou la génération de réponses en batch (Make + OpenAI).  
- **Mode hors‑ligne** : Botpress Cloud propose une version auto‑hébergée qui peut fonctionner sur un serveur local (ex. : VPS à Lagos) pour garantir la continuité du service même en cas de coupure d’internet.

### 4. Intégrations avec les outils du quotidien

| Besoin | Landbot | Chatfuel | Make + OpenAI | Botpress Cloud |
|--------|---------|----------|---------------|----------------|
| CRM (HubSpot, Zoho) | ✔ (via Zapier) | ✔ (via intégration native) | ✔ (Make possède des connecteurs) | ✔ (API REST) |
| Paiement mobile (M‑Pay, Orange Money) | ❌ (requiert webhook) | ❌ (via API) | ✔ (Make possède des modules) | ✔ (via plugins) |
| WhatsApp Business API | Via Twilio (Make) | Via Twilio (Chatfuel) | ✔ (Make + Twilio) | ✔ (via webhook) |
| SMS (Africastalking) | ❌ (via webhook) | ❌ | ✔ (Make possède le module) | ✔ (via API) |

### 5. Support et communauté locale

- **Langue du support** : Un service d’assistance en français réduit le temps de résolution. Landbot et Chatfuel offrent un support ticket en français, tandis que Botpress propose un forum francophone actif.  
- **Formations et webinars** : Empire du Web collabore régulièrement avec les partenaires de Landbot (Dakar) et Botpress (Johannesburg) pour organiser des sessions pratiques.  

### 6. Sécurité et conformité des données

- **Résidence des données** : Certaines entreprises préfèrent que les logs restent en Europe (RGPD) ou en Afrique (ex. : serveur dédié en Afrique du Sud via Botpress).  
- **Chiffrement** : Toutes les plateformes listées utilisent TLS 1.3 pour les échanges API. Botpress Cloud propose le chiffrement au repos (AES‑256).  

---

## Méthodologie de comparaison : le tableau de décision

Pour faciliter le choix, créez un tableau à deux colonnes : **Critère** et **Score** (de 1 à 5). Exemple :

| Critère | Pondération | Landbot | Chatfuel | Make + OpenAI | Botpress Cloud |
|---------|-------------|---------|----------|---------------|----------------|
| Coût mensuel (≤ 50 €) | 3 | 4 | 5 | 3 | 2 |
| Français natif | 4 | 5 | 5 | 4 | 5 |
| Intégration WhatsApp | 5 | 3 | 5 | 5 | 4 |
| Fonctionnement hors‑ligne | 2 | 1 | 2 | 2 | 4 |
| Support francophone | 3 | 4 | 4 | 3 | 4 |
| **Total pondéré** | – | **4,2** | **4,0** | **3,9** | **3,8** |

- **Pondération** : attribuez un poids à chaque critère selon son importance pour votre business (ex. : si le WhatsApp est votre canal principal, donnez‑lui 5).  
- **Score** : notez chaque plateforme de 1 (faible) à 5 (excellence).  
- **Total pondéré** : multipliez le score par la pondération, puis faites la somme. La plateforme avec le score le plus élevé correspond le mieux à vos besoins.

Cette méthode, très utilisée par les start‑ups de Nairobi, permet de prendre une décision objective sans se laisser influencer uniquement par le marketing.

---

## Études de cas concrètes

### Cas 1 : Boutique de vêtements à Abidjan – budget limité

- **Objectif** : répondre aux questions de disponibilité et de tailles via Facebook Messenger.  
- **Solution choisie** : **Chatfuel** (plan Pro à 15 €/mois).  
- **Raison** : intégration native avec la page Facebook, support en français, tarif prévisible.  
- **Mise en œuvre** : création d’un bloc « Vérifier stock » qui interroge un Google Sheet via le plugin intégré.  

```json
{
  "type": "text",
  "text": "Quel modèle vous intéresse ?",
  "quick_replies": [
    {"title": "T‑Shirt", "payload": "TSHIRT"},
    {"title": "Jean", "payload": "JEAN"}
  ]
}
```

Le bot renvoie le stock en temps réel grâce à un webhook Make qui lit le Google Sheet.

### Cas 2 : Service de micro‑finance à Kigali – besoin de conformité

- **Objectif** : automatiser la collecte d’informations clients et créer un ticket dans le CRM local.  
- **Solution choisie** : **Botpress Cloud** (Starter à 49 €/mois) hébergé sur un serveur dédié en Afrique du Sud pour respecter les exigences de localisation des données.  
- **Raison** : contrôle total sur les intents (ex. : « demande de crédit », « solde ») et capacité à exporter les logs en conformité RGPD‑Afrique.  
- **Mise en œuvre** : utilisation du **NLP** intégré pour reconnaître les entités « montant », « durée ».  

```yaml
intents:
  - name: demande_credit
    utterances:
      - "Je veux un crédit de {{montant}} euros pour {{durée}} mois"
      - "Prêt de {{montant}} pendant {{durée}}"
entities:
  - name: montant
    type: number
  - name: durée
    type: number
```

Le bot crée ensuite un ticket via l’API du CRM (ex. : **Mifos**) en appelant un webhook sécurisé.

### Cas 3 : Marketplace de produits agricoles à Lagos – besoin de scalabilité et d’intégrations multiples

- **Objectif** : offrir un assistant multicanal (WhatsApp, SMS, web) capable de proposer des prix en temps réel et d’envoyer des factures via **Paystack**.  
- **Solution choisie** : **Make + OpenAI** (plan Standard à 9 €/mois + facturation OpenAI).  
- **Raison** : flexibilité des scénarios, capacité à appeler l’API OpenAI pour générer des réponses personnalisées (ex. : recommandations de cultures selon la saison).  
- **Mise en œuvre** : scénario Make qui reçoit le message du client (WhatsApp via Twilio), interroge l’API OpenAI avec le prompt :

```json
{
  "model": "gpt-4o-mini",
  "messages": [
    {"role": "system", "content": "Tu es un assistant agronomique francophone."},
    {"role": "user", "content": "Quel maïs devrais‑je planter en juin à Lagos ?"}
  ],
  "temperature": 0.7
}
```

La réponse est ensuite formatée et renvoyée au client, puis un webhook crée une facture Paystack.

---

## Guide pas à pas pour tester rapidement chaque plateforme

1. **Créer un compte gratuit**  
   - Landbot : <https://app.landbot.io/>  
   - Chatfuel : <https://chatfuel.com/>  
   - Make : <https://www.make.com/> (connectez‑vous à OpenAI via la section « API Keys »)  
   - Botpress Cloud : <https://cloud.botpress.com/>  

2. **Construire un mini‑flow de 3 étapes**  
   - **Accueil** : « Bonjour, comment puis‑je vous aider ? »  
   - **Question** : « Quel produit cherchez‑vous ? » (menu déroulant)  
   - **Réponse** : affichage d’une donnée statique (ex. : prix).  

3. **Tester sur le canal cible** (Messenger, WhatsApp, site web).  

4. **Mesurer le temps de réponse** (latence) à l’aide du tableau de bord intégré.  

5. **Exporter les logs** et vérifier la conformité du format (CSV, JSON).  

Cette démarche rapide permet de sentir l’ergonomie de chaque interface et d’évaluer la courbe d’apprentissage.

---

## Points clés

- **Aligner le choix sur le budget** : les plans gratuits suffisent pour les premiers tests, mais prévoyez une montée en gamme dès que le volume de conversations dépasse les limites du freemium.  
- **Prioriser la langue et la localisation** : le français natif est disponible sur Landbot, Chatfuel et Botpress ; pour les langues africaines moins répandues, exploitez les prompts OpenAI ou formez vos propres intents.  
- **Tenir compte de la connectivité** : dans les zones à bande passante limitée, privilégiez les solutions qui offrent du caching ou la possibilité d’auto‑hébergement (Botpress).  
- **Vérifier les intégrations essentielles** : paiement mobile, WhatsApp Business, SMS via Africastalking – Make se distingue par son catalogue de modules prêts à l’emploi.  
- **Évaluer le support francophone** : un service client réactif en français accélère le déploiement et réduit les risques de blocage technique.  
- **Utiliser un tableau de décision pondéré** pour objectiver le choix, en adaptant les pondérations aux priorités de votre activité (coût, langue, canaux, conformité).  

En suivant ces repères, vous pourrez sélectionner la plateforme no‑code qui maximise le retour sur investissement de votre premier chatbot IA, tout en respectant les spécificités du marché africain.