## Modèle de langage : le « cerveau » du chatbot génératif  

Un modèle de langage est une intelligence artificielle entraînée à prédire le mot suivant d’une phrase, à partir d’énormes quantités de texte. Imaginez‑le comme un **bibliothécaire hyper‑polyglotte** qui a lu des millions de livres, d’articles de blog, de conversations WhatsApp et de FAQ. Quand on lui pose une question, il fouille instantanément dans cette bibliothèque interne pour composer une réponse cohérente.  

- **Paramètres** : ce sont les « neurones » du modèle. Un petit modèle comme *GPT‑2‑small* possède 124 M de paramètres, alors que *GPT‑4* en compte plusieurs milliards. Plus il y en a, plus le modèle capte de nuances, mais plus il coûte en calcul.  
- **Entraînement** : les modèles grands publics (OpenAI, Anthropic) sont pré‑entraînés sur des corpus généraux (Internet, Wikipédia). Vous n’avez pas besoin d’entraîner le modèle vous‑même ; vous le « fine‑tunez » ou le « conditionnez » avec vos propres données via des prompts (voir plus bas).  

> **Exemple africain** : un vendeur de manioc à Abidjan utilise le modèle pour répondre automatiquement aux questions sur les prix, les points de livraison et les horaires d’ouverture, sans coder de règles spécifiques.

### Prompt : la façon de parler au modèle  

Le *prompt* est le texte que vous envoyez au modèle pour déclencher une réponse. C’est l’équivalent d’une **question posée à notre bibliothécaire**. La qualité du prompt détermine la pertinence de la réponse.  

#### Structure d’un bon prompt  

| Élément | Rôle | Exemple |
|---------|------|---------|
| Contexte | Informe le modèle du domaine (ex. boutique de tissus). | *« Tu es un assistant virtuel d’une boutique de tissus à Dakar. »* |
| Instruction | Indique ce que le modèle doit faire. | *« Réponds en français, de façon concise, et propose trois modèles de tissus disponibles. »* |
| Entrée utilisateur | La question réelle du client. | *« Quels sont les prix des pagnes en coton ? »* |

#### Prompt simple vs prompt avancé  

```text
Prompt simple :
Quel est le prix du pagne en coton ?

Prompt avancé :
Tu es un assistant commercial d’une boutique de tissus à Dakar. Réponds en français, en moins de 50 mots, en indiquant le prix du pagne en coton et les tailles disponibles. Le client vient de demander :
« Quels sont les prix des pagnes en coton ? »
```

Le second prompt guide le modèle, réduit les risques de réponses hors sujet et assure une tonalité adaptée à votre marque.

### Intents (intention) : ce que veut le client  

Un *intent* représente **l’objectif sous‑jacent** d’une requête utilisateur. Dans un chatbot scripté, chaque intent est relié à une règle fixe. Dans un chatbot génératif, les intents servent surtout à **classer** les messages afin de choisir le bon contexte ou le bon prompt.  

| Intent | Exemple d’expression utilisateur |
|--------|-----------------------------------|
| `demande_prix` | « Combien coûte le pagne en soie ? » |
| `suivi_commande` | « Où en est ma livraison du 12/09 ? » |
| `horaire_ouverture` | « Vous êtes ouverts aujourd’hui ? » |

Identifier les intents permet de :

1. **Routage** : diriger la conversation vers le bon flux (FAQ, support humain, paiement).  
2. **Personnalisation du prompt** : chaque intent peut déclencher un prompt dédié.  

> **Astuce Africaine** : dans les marchés où le français est souvent mélangé avec le wolof ou le swahili, créez des intents qui capturent les expressions locales (« Combien ça coûte ? », « Ndeke ? »).  

### Entités : les informations clés à extraire  

Les *entités* sont les **données spécifiques** que le client fournit ou que le bot doit récupérer. Elles enrichissent l’intent et permettent de personnaliser la réponse.  

| Entité | Exemple |
|--------|---------|
| `produit` | *pagne en coton*, *bouteille d’huile* |
| `quantité` | *2*, *une douzaine* |
| `date` | *12/09/2024* |
| `localité` | *Abidjan*, *Kigali* |

Dans un chatbot génératif, vous pouvez **incorporer les entités directement dans le prompt** :

```text
Prompt :
Tu es un assistant de la boutique X à Abidjan. Le client a demandé le prix du produit {produit} en quantité {quantité}. Réponds en français, en indiquant le prix total.
```

Le modèle remplira `{produit}` et `{quantité}` avec les valeurs extraites du message.

### Flux conversationnel : le fil conducteur  

Le *flux conversationnel* (ou *dialogue flow*) décrit la séquence d’étapes que le bot suit pour atteindre un objectif. Même si le modèle génératif ne suit pas un arbre strict, il est utile de **définir des points de décision** :

1. **Accueil** – présentation et collecte d’informations de base.  
2. **Détection d’intent** – classification du message.  
3. **Extraction d’entités** – récupération des paramètres (produit, quantité…).  
4. **Appel au prompt** – génération de la réponse.  
5. **Boucle de clarification** – si le modèle ne comprend pas, demander des précisions.  
6. **Clôture ou escalade** – terminer la conversation ou passer à un agent humain.  

#### Exemple de flux pour un commerce de fruits  

```
[Début] → Salut ! Que puis‑je faire pour vous ?  
   └─> Client : « Je veux des mangues, 5 kg »  
        → Intent = `commande_produit`  
        → Entités = {produit: mangues, quantité: 5 kg}  
        → Prompt généré → Réponse : « 5 kg de mangues coûtent 2 500 FCFA. Voulez‑vous confirmer ? »  
            └─> Si oui → Crée la commande via l’API de paiement.  
            └─> Si non → Propose d’autres fruits ou termine.  
[Fin]
```

Ce schéma montre que même avec un modèle génératif, **le contrôle du parcours reste entre vos mains** grâce aux étapes de classification et d’extraction.

## Chatbot scripté vs chatbot génératif  

| Caractéristique | Chatbot scripté | Chatbot génératif |
|-----------------|----------------|-------------------|
| **Logique** | Arbre de décision fixe (if/else). | Prompt dynamique, réponse libre. |
| **Maintenance** | Ajout de nouvelles règles = code ou interface. | Modification du prompt ou ajout de nouvelles données d’entraînement. |
| **Couverture** | Limité aux scénarios pré‑vus. | Capable de gérer des variantes inattendues. |
| **Coût d’infrastructure** | Faible (exécution locale). | Dépend de l’API (coût par token). |
| **Langues & dialectes** | Nécessite chaque variante explicitement. | Le modèle comprend souvent les variantes grâce à son entraînement massif. |
| **Risques** | Risque d’impasse si le flux n’est pas prévu. | Risque de « hallucination » (réponse plausible mais fausse). |

### Quand choisir l’un ou l’autre ?  

- **Scripté** : processus très réglementés (ex. procédures bancaires, conformité).  
- **Génératif** : interactions riches, FAQ évolutives, support multilingue, besoin de flexibilité.  

Dans la plupart des PME africaines, le **génératif** apporte un gain de temps considérable, surtout lorsqu’on doit répondre à des questions variées sur les produits, les livraisons ou les promotions locales.

## Comment l’IA génère une réponse en temps réel  

1. **Réception du message** – le bot capte le texte via le canal (WhatsApp, site web, Facebook).  
2. **Pré‑traitement** – nettoyage (suppression des emojis inutiles, normalisation des caractères accentués).  
3. **Classification** – appel à un petit modèle (ou à OpenAI `text‑classification`) pour identifier l’intent.  
4. **Extraction d’entités** – utilisation d’une fonction de reconnaissance d’entités (NER) ou de regex adaptées aux langues locales.  
5. **Construction du prompt** – le système assemble le contexte, l’instruction et les valeurs d’entités.  
6. **Appel à l’API du modèle** – envoi du prompt à OpenAI (ou à un autre LLM) avec les paramètres `temperature`, `max_tokens`, etc.  
7. **Réception de la réponse** – le modèle renvoie un texte généré.  
8. **Post‑traitement** – vérification de la longueur, filtrage de contenus sensibles (via `moderation endpoint`).  
9. **Envoi au client** – le texte final est transmis sur le canal d’origine.  

### Paramètres clés de génération  

| Paramètre | Description | Valeur typique pour un chatbot |
|-----------|-------------|--------------------------------|
| `temperature` | Niveau de créativité (0 = déterministe, 1 = créatif). | 0.3 – 0.6 pour éviter les réponses hors sujet. |
| `max_tokens` | Longueur maximale de la réponse. | 150 tokens (environ 80‑100 mots). |
| `top_p` | Nucleus sampling, contrôle de la diversité. | 0.9 (souvent laissé par défaut). |
| `presence_penalty` | Décourage la répétition de mots déjà utilisés. | 0.2 – 0.5. |

En ajustant ces paramètres, vous pouvez **équilibrer rapidité, pertinence et coût**. Un réglage trop créatif (`temperature` = 0.9) peut générer des réponses marketing exagérées, alors qu’un réglage trop bas (`temperature` = 0) rend les réponses monotones.

## Analogie du chef cuisinier  

Pensez au modèle comme à un **chef cuisinier** :  

- Le *prompt* est la **recette** que vous lui donnez.  
- Les *intents* sont les **plats** que le client souhaite (soupe, ragoût, salade).  
- Les *entités* sont les **ingrédients** (piment, riz, poisson).  
- Le *flux conversationnel* est le **service** : prise de commande, cuisson, dressage, service.  

Si vous donnez une recette claire et précise, le chef prépare le plat exactement comme attendu. Si la recette est vague, le résultat peut être surprenant — tout comme avec un prompt mal formulé.

## Cas pratique : un chatbot pour une petite entreprise de distribution d’énergie solaire  

### 1. Définir les intents et entités  

| Intent | Exemples d’utterances | Entités clés |
|--------|----------------------|--------------|
| `demande_prix` | « Quel est le prix du kit solaire ? », « Combien coûte un panneau ? » | `produit` |
| `disponibilite` | « Vous avez des batteries en stock ? » | `produit`, `localité` |
| `support_technique` | « Mon panneau ne charge plus ! » | `produit`, `symptome` |
| `paiement` | « Comment payer par mobile money ? » | `moyen_paiement` |

### 2. Prompt de base (en français)  

```text
Tu es l’assistant virtuel d’une entreprise de distribution d’énergie solaire au Kenya. Réponds en français, en restant courtois et en incluant le prix en KES. Utilise les informations suivantes :

Produit : {produit}
Quantité : {quantité}
Localité : {localité}
```

### 3. Implémentation no‑code (exemple avec Make + OpenAI)  

```json
{
  "model": "gpt-4o-mini",
  "messages": [
    {"role": "system", "content": "Tu es un assistant commercial pour une boutique d’énergie solaire au Kenya."},
    {"role": "user", "content": "Quel est le prix du kit solaire de 200 W pour Nairobi ?"}
  ],
  "temperature": 0.4,
  "max_tokens": 120
}
```

Le résultat pourrait être :

> « Le kit solaire de 200 W coûte 24 500 KES pour Nairobi. Il comprend le panneau, la batterie et le contrôleur. Voulez‑vous passer commande ? »

### 4. Gestion des réponses inattendues  

Si le modèle répond hors sujet (ex. « Les panneaux sont verts »), le post‑traitement doit :

- Vérifier la présence du mot‑clé `prix` ou `coût`.  
- Si absent, renvoyer le message à un **fallback** : « Désolé, je n’ai pas compris. Pouvez‑vous reformuler votre question ? »  

Cette logique de **fallback** fait partie du flux conversationnel et assure une expérience fluide.

## Sécuriser les échanges et respecter les données locales  

- **Conformité RGPD & loi locale** : anonymisez les données personnelles avant de les envoyer à l’API.  
- **Chiffrement** : utilisez HTTPS et stockez la clé API dans un coffre (ex. Azure Key Vault, AWS Secrets Manager).  
- **Limitation des tokens** : définissez un plafond quotidien dans votre plateforme no‑code pour éviter les dépassements de budget.  

> **Voir chapitre 04** pour les détails de création et sécurisation de la clé API OpenAI.

## À retenir  

- Le **modèle de langage** est le moteur qui génère les réponses ; le **prompt** agit comme la recette qui le guide.  
- **Intents** et **entités** permettent de structurer la conversation, même lorsqu’on utilise un modèle génératif.  
- Un **flux conversationnel** bien pensé garde le contrôle sur le parcours utilisateur tout en profitant de la souplesse du LLM.  
- Les chatbots scriptés offrent prévisibilité et faible coût, alors que les chatbots génératifs offrent adaptabilité et richesse linguistique, cruciales pour les marchés africains multilingues.  
- La génération en temps réel repose sur une chaîne de traitement : réception → classification → extraction → prompt → appel LLM → post‑traitement → réponse.  
- Ajuster les paramètres (`temperature`, `max_tokens`, etc.) permet de maîtriser le ton, la longueur et le coût.  
- Toujours prévoir un **fallback** et sécuriser les données pour garantir une expérience fiable et conforme.