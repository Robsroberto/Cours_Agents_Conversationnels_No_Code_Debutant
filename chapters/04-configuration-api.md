## Créer votre compte OpenAI

### 1. S’inscrire sur la plateforme OpenAI  
1. Ouvrez le navigateur et rendez‑vous sur **https://platform.openai.com/**.  
2. Cliquez sur **Sign‑up** puis choisissez l’option *email* ou *Google* selon votre préférence.  
3. Remplissez les champs obligatoires (nom, prénom, adresse e‑mail, mot de passe).  
4. Validez votre adresse e‑mail en cliquant sur le lien reçu dans votre boîte de réception.  

> **Astuce pour les entrepreneurs africains** : utilisez une adresse professionnelle (ex. `contact@votreentreprise.com`) afin de séparer les communications personnelles et commerciales.

### 2. Vérification d’identité et configuration de la facturation  
OpenAI impose une vérification d’identité pour débloquer l’accès aux modèles GPT‑4 et à des quotas plus élevés.  

| Étape | Action | Pourquoi |
|------|--------|----------|
| **Vérification d’identité** | Accédez à *Billing → Verification* et téléversez une pièce d’identité (passeport ou carte nationale). | Permet de respecter les exigences de conformité et d’éviter les abus. |
| **Ajout d’un moyen de paiement** | Dans *Billing → Payment methods*, saisissez les coordonnées d’une carte bancaire internationale ou d’un compte PayPal. | La facturation est à l’usage : chaque appel API consomme des crédits. |
| **Définir un plafond mensuel** | Sous *Billing → Usage limits*, indiquez le montant maximal que vous êtes prêt à dépenser chaque mois. | Évite les surprises sur la facture, surtout en phase de test. |

> Pour les petites PME qui ne souhaitent pas dépasser 20 USD/mois, fixez le plafond à 15 USD et surveillez les alertes d’usage dans le tableau de bord.

## Obtenir et sécuriser votre clé API

### 1. Générer la clé  
1. Dans le tableau de bord, cliquez sur **API keys** dans le menu latéral.  
2. Sélectionnez **Create new secret key**.  
3. Donnez un nom explicite (ex. `chatbot_no_code_prod`).  
4. Copiez immédiatement la clé affichée ; elle ne sera plus visible après la fermeture de la fenêtre.  

> **Important** : La clé est l’équivalent d’un mot de passe. Toute personne qui la possède peut consommer votre quota.

### 2. Stockage sécurisé dans les outils no‑code  

#### a. Make (anciennement Integromat)  
- Ouvrez votre scénario, ajoutez le module **HTTP > Make a request**.  
- Dans l’onglet **Headers**, créez une nouvelle variable d’en‑tête : `Authorization : Bearer {{your_api_key}}`.  
- Au lieu de coller la clé en clair, utilisez la fonction **Secrets** de Make :  
  1. Accédez à **Tools → Secrets**.  
  2. Créez un secret nommé `OPENAI_API_KEY` et collez-y votre clé.  
  3. Dans le module HTTP, choisissez **{{secret.OPENAI_API_KEY}}**.  

#### b. Zapier  
- Dans le tableau de bord Zapier, allez dans **My Apps → Add a New Account**.  
- Recherchez **OpenAI** et cliquez sur **Connect**.  
- Zapier vous demandera d’entrer votre clé : collez‑la dans le champ prévu, Zapier la crypte automatiquement.  

#### c. Landbot  
- Ouvrez votre bot, cliquez sur **Integrations → API**.  
- Dans la section **Headers**, ajoutez `Authorization` avec la valeur `Bearer {{api_key}}`.  
- Landbot propose un stockage **Secure Variables** : créez une variable nommée `OPENAI_KEY` et insérez votre clé. Utilisez `{{secure.OPENAI_KEY}}` dans le header.  

### 3. Rotation régulière de la clé  
Planifiez une rotation tous les 90 jours : créez une nouvelle clé, mettez à jour les secrets dans chaque outil, puis révoquez l’ancienne clé depuis le tableau de bord OpenAI. Cette pratique réduit le risque d’accès non autorisé.

## Gérer les quotas et la facturation

### 1. Comprendre le système de crédits  
OpenAI facture à la *token* : chaque token représente environ 4 octets de texte (un mot moyen). Les tarifs (au 1 octobre 2026) sont :

| Modèle | Prix (USD) par 1 000 tokens (prompt) | Prix (USD) par 1 000 tokens (completion) |
|--------|--------------------------------------|------------------------------------------|
| **GPT‑3.5‑Turbo** | 0,0005 | 0,0015 |
| **GPT‑4 (8 k)** | 0,0030 | 0,0060 |
| **GPT‑4 (32 k)** | 0,0600 | 0,1200 |

> **Exemple concret** : une réponse de 150 tokens (environ 100 mots) avec GPT‑3.5‑Turbo coûte ≈ 0,0002 USD. Un même échange avec GPT‑4 (8 k) coûtera ≈ 0,0012 USD.

### 2. Configurer les limites d’usage  
Dans *Billing → Usage limits* :  

- **Hard limit** : stoppe immédiatement les appels dès dépassement.  
- **Soft limit** : envoie une alerte par e‑mail mais continue l’exécution.  

Pour un projet de test, activez le *hard limit* à 10 USD afin de ne pas dépasser le budget initial.

### 3. Suivi en temps réel  
- **Dashboard** : le graphique *Usage* montre le nombre de tokens consommés par jour.  
- **Webhooks d’événement** : sous *Settings → Webhooks*, configurez un webhook qui vous notifie lorsqu’un appel dépasse un certain nombre de tokens (ex. > 500 tokens).  

## Choisir le modèle adapté à votre audience francophone

### 1. GPT‑3.5‑Turbo vs GPT‑4  
| Critère | GPT‑3.5‑Turbo | GPT‑4 |
|---------|---------------|-------|
| **Coût** | Très bas | Élevé |
| **Qualité du français** | Bon, parfois des incohérences | Excellent, meilleure compréhension des nuances culturelles |
| **Temps de réponse** | ~ 200 ms | ~ 500 ms |
| **Capacité contextuelle** | 4 k tokens | 8 k ou 32 k tokens (selon le plan) |

Pour un chatbot de service client basique (FAQ, prise de rendez‑vous), **GPT‑3.5‑Turbo** suffit.  
Pour des scénarios complexes (négociation, rédaction de devis personnalisés), privilégiez **GPT‑4 (8 k)**.

### 2. Adapter le modèle à la langue française  
OpenAI a entraîné les modèles sur de larges corpus multilingues, mais le français bénéficie d’un meilleur traitement avec GPT‑4.  

- **Prompt en français** : toujours commencer par une instruction claire, par ex. `Réponds en français, de manière concise, en évitant le jargon technique.`  
- **Utiliser des exemples** : dans le prompt, fournissez des exemples de réponses typiques pour guider le modèle (`Exemple : Question : « Quel est le délai de livraison ? » Réponse : « Nous livrons sous 48 h en ville, 72 h hors agglomération. »`).  

## Configurer les paramètres de génération

### 1. Température  
- **Valeur** : 0 → 1.  
- **Effet** : 0 rend les réponses déterministes (toujours la même réponse); 1 augmente la créativité et la diversité.  

**Recommandation** : pour un chatbot de support, fixez `temperature = 0.3`. Cela garantit des réponses cohérentes tout en laissant une petite marge de variation pour éviter les répétitions.

### 2. Max tokens (limite de sortie)  
Déterminez le nombre maximal de tokens que le modèle peut générer.  

- **FAQ simples** : `max_tokens = 150`.  
- **Rédaction de texte marketing** : `max_tokens = 400`.  

> **Astuce** : si le texte dépasse la limite, le modèle le tronque, ce qui peut rendre une réponse incomplète. Testez différents seuils dans le Playground avant de les fixer dans votre scénario no‑code.

### 3. Top‑p (nucleus sampling) et fréquence / présence penalty  
- **top_p** : 0,8 → 0,95 (contrôle la probabilité cumulative).  
- **frequency_penalty** : 0 → 2 (réduit la répétition de mots).  
- **presence_penalty** : 0 → 2 (encourage l’introduction de nouveaux concepts).  

Pour un chatbot qui doit rester factuel, utilisez `top_p = 0.9`, `frequency_penalty = 0.5`, `presence_penalty = 0.0`.

### 4. Exemple de configuration dans le Playground  

```json
{
  "model": "gpt-4",
  "temperature": 0.3,
  "max_tokens": 200,
  "top_p": 0.9,
  "frequency_penalty": 0.5,
  "presence_penalty": 0.0,
  "messages": [
    {"role": "system", "content": "Tu es un assistant virtuel pour une boutique de tissus africains. Réponds toujours en français, de façon polie et concise."},
    {"role": "user", "content": "Quel est le prix du tissu Ankara en 2 mètres ?"}
  ]
}
```

Copiez‑collez ce JSON dans le **Playground** pour vérifier le rendu avant d’intégrer les paramètres dans votre outil no‑code.

## Tester rapidement l’API sans code

### 1. Appel via `curl`  
```bash
curl https://api.openai.com/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer VOTRE_CLÉ_API" \
  -d '{
    "model": "gpt-3.5-turbo",
    "messages": [{"role":"user","content":"Quel est le délai de livraison pour un client à Dakar ?"}],
    "temperature": 0.3,
    "max_tokens": 120
  }'
```
Analysez la réponse JSON : le champ `choices[0].message.content` contient le texte généré.

### 2. Vérifier le coût d’un appel  
Dans la réponse, le champ `usage` indique le nombre de tokens consommés (prompt + completion). Multipliez par le tarif du modèle pour connaître le coût réel de cet appel.

### 3. Intégrer le test dans Make  
1. Créez un scénario **HTTP > Make a request**.  
2. Copiez le corps JSON ci‑dessus dans le champ *Request body*.  
3. Ajoutez un module **Tools > JSON > Parse JSON** pour extraire le texte.  
4. Ajoutez un module **Notifier > Email** pour recevoir la réponse directement dans votre boîte.  

Ce workflow vous permet de valider le bon fonctionnement de la clé, du modèle et des paramètres avant de le connecter à votre chatbot no‑code.

## Bonnes pratiques spécifiques au marché francophone africain

- **Gestion des accents** : assurez‑vous que votre outil transmet les caractères Unicode correctement. Dans Make, activez l’option *Encode URL* pour éviter les corruptions.  
- **Fuseaux horaires** : les utilisateurs de Lagos, Nairobi ou Abidjan attendent des réponses horodatées dans leur zone locale. Ajoutez un prompt du type `Inclure l’heure locale de {{city}} dans la réponse.`  
- **Références culturelles** : enrichissez le prompt système avec des exemples de salutations locales (`« Bonjour ! Comment puis‑je vous aider aujourd’hui ? »`). Cela améliore la pertinence perçue.  

## Points clés

- Créez votre compte OpenAI, validez votre identité et définissez un plafond de dépenses pour garder le contrôle budgétaire.  
- La clé API doit être stockée comme **secret** dans chaque plateforme no‑code (Make, Zapier, Landbot) ; ne jamais la coller en clair.  
- Surveillez les quotas via le tableau de bord et les webhooks d’événement afin d’éviter les dépassements inattendus.  
- Choisissez **GPT‑3.5‑Turbo** pour les cas simples et **GPT‑4** (8 k) pour des interactions plus riches, notamment en français.  
- Réglez `temperature`, `max_tokens`, `top_p` et les pénalités de fréquence/presence pour obtenir le ton et la concision adaptés à votre audience.  
- Testez chaque configuration d’abord dans le Playground, puis avec un appel `curl` ou un petit scénario Make avant de l’intégrer à votre chatbot.  
- Adaptez le prompt système aux spécificités culturelles et linguistiques de vos clients africains pour renforcer la confiance et l’engagement.