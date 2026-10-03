## Structurer le flux de conversation : du schéma à l’arbre décisionnel

Un chatbot efficace repose avant tout sur un **flux conversationnel bien pensé**.  
Il s’agit de déterminer, étape par étape, ce que l’utilisateur peut dire, comment le bot doit réagir, et quelles informations il doit retenir pour personnaliser la suite.  

Dans ce chapitre, nous construisons **un arbre de décision complet** : identification du client, collecte d’informations, réponses aux FAQ et redirection vers un humain. Vous découvrirez comment :

* définir les **intents** (objectifs) et les **entités** (données variables) ;  
* créer des **réponses dynamiques** qui intègrent les variables collectées ;  
* structurer le tout dans un outil no‑code (ex. : Landbot ou Make + OpenAI) sans écrire une ligne de code back‑end.

---

## 1. Cartographier le parcours client

Avant d’ouvrir l’interface de la plateforme, dessinez le parcours sur papier ou sur un outil de mind‑mapping (ex. : Miro, Lucidchart). Voici un exemple de **schéma simplifié** pour une boutique en ligne de produits artisanaux (bijoux, sacs, tissus) basée à Dakar :

1. **Accueil** – Salutation + proposition d’aide.  
2. **Identification** – Demande du prénom (ou du numéro client).  
3. **Objectif** – L’utilisateur choisit parmi :  
   - *Consulter le catalogue*  
   - *Obtenir le prix d’un produit*  
   - *Suivre une commande*  
   - *Poser une question fréquente*  
4. **Collecte d’informations** – Selon l’objectif, le bot demande les données manquantes (ex. : référence du produit, numéro de suivi).  
5. **Réponse dynamique** – Le bot utilise les variables récupérées pour générer une réponse personnalisée.  
6. **Escalade** – Si le bot ne peut répondre, il propose de parler à un agent humain.  

Ce diagramme devient le **plan de votre arbre décisionnel**. Chaque nœud correspond à un **intent** ; chaque branche, à une **transition** conditionnelle.

---

## 2. Définir les intents et les entités

### 2.1. Intents de base

| Intent | Description | Exemple d’utilisateur |
|--------|-------------|-----------------------|
| `greeting` | Saluer et proposer de l’aide | “Bonjour”, “Salut” |
| `identify_user` | Collecter le prénom ou le numéro client | “Je m’appelle Aïssa”, “Mon numéro c’est 12345” |
| `browse_catalog` | Demander le catalogue | “Montre‑moi les sacs” |
| `price_query` | Obtenir le prix d’un produit | “Quel est le prix du sac « Mali » ?” |
| `order_status` | Suivre une commande | “Où en est ma commande 9876 ?” |
| `faq` | Question fréquente (retour, livraison…) | “Comment retourner un article ?” |
| `human_transfer` | Rediriger vers un agent | “Je veux parler à un humain” |

### 2.2. Entités dynamiques

| Entité | Type | Exemple de valeur |
|--------|------|-------------------|
| `user_name` | texte | “Aïssa” |
| `customer_id` | numérique | 12345 |
| `product_ref` | texte | “SAC-MALI” |
| `order_id` | alphanumérique | “ORD-9876” |
| `faq_topic` | catégorie | “retour”, “livraison” |

Ces entités seront **stockées dans des variables** que la plateforme met à disposition pour les réponses suivantes.

---

## 3. Construire l’arbre décisionnel dans un outil no‑code

Nous illustrons ici la mise en place sur **Landbot**, mais les principes sont identiques sur Make, Chatfuel ou Botpress Cloud.

### 3.1. Bloc « Message d’accueil »

1. Créez un **Bloc de texte** contenant :  
   ```
   👋 Bonjour {{user_name | default:"!"}}, je suis *Lina*, votre assistant virtuel.  
   Que puis‑je faire pour vous aujourd’hui ?
   ```  
   La syntaxe `{{variable | default:"!"}}` affiche un point d’exclamation si la variable n’est pas encore définie.

2. Ajoutez quatre **Boutons** :  
   - “Voir le catalogue” → transition → intent `browse_catalog`  
   - “Quel est le prix ?” → transition → intent `price_query`  
   - “Suivre ma commande” → transition → intent `order_status`  
   - “FAQ” → transition → intent `faq`

### 3.2. Bloc d’identification

Si `user_name` n’est pas encore remplie, le bot doit la demander :

```markdown
Quel est votre prénom ?  
(Enregistrement dans la variable `user_name`)
```

Dans Landbot, utilisez le **Bloc Question – Texte** avec l’option *Enregistrer la réponse dans* → `user_name`.  
Ajoutez une condition de sortie : si `user_name` n’est pas vide, passer au bloc suivant ; sinon répéter la question (max 3 essais).

### 3.3. Gestion du catalogue

1. **Intent** : `browse_catalog`.  
2. **Bloc** : afficher une galerie d’images (ou des cartes) représentant les catégories (bijoux, sacs, tissus).  
3. Chaque carte possède un **payload** qui déclenche l’intent `price_query` avec l’entité `product_ref` pré‑remplie. Exemple de payload :

```json
{
  "intent": "price_query",
  "entities": {
    "product_ref": "SAC-MALI"
  }
}
```

Ainsi, dès que l’utilisateur clique sur la carte “Sac Mali”, le bot sait de quel produit il parle.

### 3.4. Question de prix dynamique

Bloc **Message** :

```
Le sac *Mali* coûte **{{price_of(product_ref)}}** FCFA.  
Souhaitez‑vous l’ajouter au panier ?  
[Oui] [Non]
```

Le placeholder `{{price_of(product_ref)}}` nécessite une **fonction custom** (ou un appel API) qui récupère le prix depuis votre catalogue. Sur Landbot, créez un **Bloc API** qui :

- Envoie une requête GET à `https://api.maboutique.com/products/{{product_ref}}`  
- Récupère le champ `price` et le stocke dans la variable `product_price`  
- Retourne la valeur dans le message précédent grâce à `{{product_price}}`.

### 3.5. Suivi de commande

Intent : `order_status`.  

Bloc **Question – Texte** :  
```
Veuillez saisir votre numéro de commande :
```
Enregistrement dans `order_id`.  

Bloc **API** : appel à `https://api.maboutique.com/orders/{{order_id}}`  
Réponse typique :

```json
{
  "status": "En cours de préparation",
  "estimated_delivery": "2024-10-10"
}
```

Bloc **Message** :

```
Votre commande {{order_id}} est **{{status}}**.  
Livraison estimée le {{estimated_delivery}}.
```

### 3.6. FAQ avec entité de sujet

Créez une **liste de topics** (retour, livraison, paiement).  

Bloc **Question – Choix** :  
```
Quel sujet vous intéresse ?  
- Retour  
- Livraison  
- Paiement
```

Enregistrement dans `faq_topic`.  

Bloc **Switch / Condition** : selon la valeur de `faq_topic`, afficher le texte correspondant. Exemple :

```markdown
{% if faq_topic == "Retour" %}
Vous avez 30 jours pour retourner un produit. Merci de nous envoyer le colis à l’adresse suivante…
{% elif faq_topic == "Livraison" %}
Nos livraisons sont assurées par DHL et arrivent en 3 à 5 jours ouvrés…
{% else %}
Le paiement s’effectue par carte bancaire ou mobile money (MTN Mobile Money, Orange Money)…
{% endif %}
```

Landbot propose un **Bloc Condition** avec plusieurs branches ; chaque branche contient le texte de la réponse.

### 3.7. Escalade vers un agent humain

Dans chaque branche (catalogue, prix, suivi, FAQ), ajoutez un **bouton “Parler à un humain”**.  
Le bouton déclenche l’intent `human_transfer`.  

Bloc **Message** :

```
Je vous transfère immédiatement à un de nos conseillers. Veuillez patienter…
```

Puis utilisez le **Bloc Zapier/Make** pour créer un ticket dans votre CRM (ex. : HubSpot, Zoho) avec les variables suivantes :

- `user_name`  
- `customer_id` (s’il est connu)  
- `last_intent` (pour que l’agent sache où le client était bloqué)  

Enfin, envoyez le lien de la session de chat (ou le numéro de téléphone) à l’agent via Slack ou WhatsApp.

---

## 4. Exercices pratiques

### Exercice 1 – Dessiner son propre arbre

1. Choisissez votre secteur (ex. : vente de produits agricoles, cours de langues en ligne).  
2. Identifiez **au moins 5 intents** pertinents et **3 entités** à collecter.  
3. Dessinez le flux sur papier, en indiquant les points de décision (oui/non, sélection de catégorie).  

### Exercice 2 – Implémenter le bloc de prix

1. Dans votre plateforme no‑code, créez un **Bloc API** qui interroge un endpoint fictif : `https://myapi.com/price/{{product_ref}}`.  
2. Simulez la réponse JSON `{ "price": 12500 }`.  
3. Utilisez la variable `price` dans un message de réponse dynamique.  

### Exercice 3 – Gestion d’erreurs

Ajoutez une **condition** qui détecte une réponse vide ou un ID invalide :  
- Si `order_id` ne correspond à aucun enregistrement, le bot répond :  
  ```
  Désolé, je n’ai pas trouvé votre commande. Vérifiez le numéro et réessayez.
  ```  
- Limitez le nombre d’essais à 3, puis proposez l’escalade automatique.  

### Exercice 4 – Personnalisation avancée

Utilisez la variable `user_name` pour créer un **message de bienvenue personnalisé** chaque fois que le client revient :  

```markdown
Re‑bonjour {{user_name}} ! Vous étiez en train de consulter le sac « {{product_ref}} ». Souhaitez‑vous finaliser l’achat ?
```

Implémentez une **mémoire de session** (Landbot conserve les variables tant que le chat reste ouvert). Testez la persistance en rafraîchissant la page.

---

## 5. Bonnes pratiques pour un flux robuste

| Aspect | Recommandation | Pourquoi |
|--------|----------------|----------|
| **Clarté des intents** | Nommez les intents de façon explicite (`price_query`, `order_status`). | Facilite le debug et la maintenance. |
| **Gestion des cas d’erreur** | Prévoir toujours une branche “fallback” (ex. : “Je n’ai pas compris”). | Évite que le bot reste bloqué. |
| **Limite de boucles** | Restreindre les répétitions (max 3) avant de proposer l’escalade. | Améliore l’expérience utilisateur. |
| **Variables obligatoires** | Marquez les variables critiques (`order_id`, `product_ref`) comme *required*. | Garantit que les appels API reçoivent les bons paramètres. |
| **Sécurité** | Ne jamais exposer la clé API dans le front‑end ; utilisez le module de stockage sécurisé de la plateforme. | Voir chapitre 4. |
| **Localisation** | Utilisez des formulations et des devises locales (FCFA, XOF). | Renforce la proximité avec le client africain. |
| **Test continu** | Simulez chaque chemin du flux avec des jeux de données réelles. | Détecte les incohérences avant le lancement. |

---

## Points clés

- **Cartographier** le parcours client avant de coder : chaque étape devient un intent ou une condition.  
- **Séparer** les intents (objectifs) et les entités (données) pour garder le flux modulaire et réutilisable.  
- **Utiliser** des réponses dynamiques (`{{variable}}`) pour personnaliser chaque échange et augmenter le taux de conversion.  
- **Intégrer** les appels API (catalogue, suivi, CRM) via des blocs dédiés, en stockant les réponses dans des variables temporaires.  
- **Prévoir** systématiquement un chemin d’escalade vers un humain, avec transmission du contexte (intents, variables) au support.  
- **Tester** chaque branche, gérer les erreurs et limiter les boucles afin d’assurer une expérience fluide pour les utilisateurs africains.