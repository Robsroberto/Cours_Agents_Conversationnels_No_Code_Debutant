## Préparer vos jeux de données métier  

### Identifier les sources de connaissance  

Un chatbot qui répond correctement ne peut se contenter du « knowledge » général du modèle ; il doit être alimenté par les informations spécifiques à votre activité :  

| Type de donnée | Exemple concret (Afrique francophone) | Où la retrouver |
|----------------|----------------------------------------|-----------------|
| FAQ client | « Quel est le délai de livraison à Bamako ? » | Documents support, tickets CRM |
| Catalogue produit | Nom, description, prix, stock, SKU des pagnes de wax | Export CSV/E‑commerce (Shopify, WooCommerce) |
| Politique de retour | Conditions, frais, procédure | Page juridique du site, PDF interne |
| Tarifs & promotions | Codes promo, période de validité | Tableaux Google Sheet partagés |
| Horaires d’ouverture | Boutique physique à Abidjan, service téléphonique | Calendrier interne |

**Astuce** : commencez par un audit rapide : listez chaque point de contact client (site, WhatsApp, Facebook) et notez les questions récurrentes. Cette cartographie vous donne la base du jeu de données.

### Nettoyer et structurer les données  

1. **Uniformiser la langue** – choisissez le français comme langue principale, mais conservez les variantes locales (ex. : « biko » pour « merci » en lingala).  
2. **Supprimer les doublons** – deux FAQ identiques ne font que gonfler le prompt.  
3. **Normaliser les formats** – utilisez CSV ou JSON ; chaque ligne représente une paire *question‑réponse*.  

Exemple de fichier CSV :  

```csv
question,réponse
"Quel est le délai de livraison à Bamako ?","Livraison en 2 à 4 jours ouvrés depuis notre entrepôt de Dakar."
"Comment retourner un article acheté en ligne ?","Vous avez 30 jours pour retourner l’article. Rendez‑vous sur notre page Retour, remplissez le formulaire et choisissez le point de collecte le plus proche."
```

Exemple de JSON :  

```json
[
  {
    "question": "Quel est le délai de livraison à Bamako ?",
    "réponse": "Livraison en 2 à 4 jours ouvrés depuis notre entrepôt de Dakar."
  },
  {
    "question": "Comment retourner un article acheté en ligne ?",
    "réponse": "Vous avez 30 jours pour retourner l’article. Rendez‑vous sur notre page Retour, remplissez le formulaire et choisissez le point de collecte le plus proche."
  }
]
```

### Découper les informations volumineuses  

Les catalogues produits peuvent contenir des centaines d’articles ; il n’est pas judicieux d’injecter tout le catalogue dans chaque prompt.  
- **Regroupez par catégorie** (ex. : « Pagnes », « Chaussures », « Accessoires »).  
- **Limitez la taille du contexte** : ne dépassez pas 2 000 tokens (voir chapitre 4 pour la gestion des quotas).  

## Structurer les données pour le few‑shot learning  

Le *few‑shot* consiste à fournir au modèle quelques exemples d’interaction afin de le guider. La structure typique est :  

```
[Exemple 1]  
Q: <question>  
A: <réponse>

[Exemple 2]  
Q: <question>  
A: <réponse>

...  
Nouvelle question de l’utilisateur
Q: <question utilisateur>
A:
```

### Créer un « prompt template » réutilisable  

```text
Tu es l’assistant virtuel de **[Nom de l’entreprise]**, spécialisé dans la vente de vêtements traditionnels en Afrique de l’Ouest. Réponds de façon concise, polie et en français. Si la question sort du périmètre, indique clairement « Je ne suis pas en mesure de répondre à cette question. ».  
Voici quelques exemples :

Q: Quel est le délai de livraison à Bamako ?
A: Livraison en 2 à 4 jours ouvrés depuis notre entrepôt de Dakar.

Q: Quels sont les matériaux du pagne « Bamako » ?
A: 100 % coton, teinté à la main.

Q: Comment retourner un article acheté en ligne ?
A: Vous avez 30 jours pour retourner l’article. Rendez‑vous sur notre page Retour, remplissez le formulaire et choisissez le point de collecte le plus proche.

Q: <question utilisateur>
A:
```

Ce template garde la même *tone* et le même format pour chaque appel d’API. Vous n’avez qu’à remplacer la dernière question par le texte reçu du visiteur.

### Limiter le nombre d’exemples  

- **2‑3 exemples** suffisent généralement pour un domaine bien circonscrit.  
- Au‑delà de 5 exemples, le coût en tokens augmente et le modèle peut « dévier » du style attendu.  

### Stocker les exemples dans un tableau no‑code  

Dans Make, créez une **Collection** contenant les champs : `question`, `réponse`, `catégorie`. Lors du déclenchement du scénario, récupérez les 3 premiers exemples pertinents (par catégorie) et assemblez le prompt avec le module *“Compose Text”*.

## Rédaction efficace des prompts  

### Règles d’or  

| Règle | Pourquoi | Exemple |
|-------|----------|---------|
| **Soyez explicite sur le rôle** | Le modèle adapte son ton en fonction du contexte donné. | « Tu es l’assistant de la boutique X » |
| **Définissez le format de sortie** | Évite les réponses trop longues ou non structurées. | « Réponds en une phrase de moins de 30 mots. » |
| **Indiquez la politique de refus** | Réduit les hallucinations hors périmètre. | « Si tu ne connais pas la réponse, réponds « Je ne sais pas ». » |
| **Utilisez des balises simples** | Facilite le parsing côté no‑code. | `Q:` et `A:` comme séparateurs. |
| **Gardez les exemples pertinents** | Le modèle se base sur les exemples les plus proches du nouveau cas. | FAQ sur la livraison → nouvelle question sur les frais de port. |

### Gestion des variables dynamiques  

Dans un scénario Make, vous pouvez injecter les valeurs utilisateur :

```text
Q: {{question_user}}
A:
```

Le double moustache `{{ }}` est la syntaxe native de Make pour insérer une variable. Cela vous permet de réutiliser le même template pour chaque requête.

### Exemple complet d’appel API (cURL)  

```bash
curl https://api.openai.com/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -d '{
    "model": "gpt-3.5-turbo",
    "messages": [
      {"role":"system","content":"Tu es l’assistant virtuel de BoutiqueWax, spécialisé dans les pagnes de wax. Réponds en français, en moins de 30 mots. Si tu ne sais pas, dis \"Je ne sais pas\"."},
      {"role":"user","content":"Quel est le délai de livraison à Bamako ?"},
      {"role":"assistant","content":"Livraison en 2 à 4 jours ouvrés depuis notre entrepôt de Dakar."},
      {"role":"user","content":"Comment retourner un article acheté en ligne ?"},
      {"role":"assistant","content":"Vous avez 30 jours pour retourner l’article. Rendez‑vous sur notre page Retour, remplissez le formulaire et choisissez le point de collecte le plus proche."},
      {"role":"user","content":"Quel est le prix du pagne \"Bamako\" taille L ?"}
    ],
    "temperature":0.2,
    "max_tokens":80
  }'
```

Remarquez : le rôle *system* fixe le ton, les paires *user/assistant* illustrent le few‑shot, et la dernière question représente l’interaction réelle.

## Gestion du multilinguisme et des langues locales  

### Prioriser le français, mais accepter les variantes  

- **Détecter la langue** : utilisez le module *“Detect Language”* de Make (ou l’API d’OpenAI `language` si disponible).  
- **Routage** : si la langue détectée n’est pas le français, vous pouvez soit :  
  1. Répondre en français avec une note : « Je réponds en français, désolé ».  
  2. Rediriger vers un agent humain francophone.  

### Créer des variantes d’exemples  

Pour chaque FAQ, ajoutez une version locale :  

| Question (FR) | Question (local) |
|---------------|------------------|
| Quels sont les frais de port ? | Kí ni owo lónìí? (Yoruba) |
| Comment suivre ma commande ? | Ndààrà wòlé ètò mi? (Bambara) |

Dans le prompt, vous pouvez placer les deux variantes :

```text
Q: Quels sont les frais de port ?
A: Les frais sont de 2 000 FCFA pour le Mali, 3 000 FCFA pour le Sénégal.

Q: Kí ni owo lónìí?
A: Àwọn owó ìkópa jẹ́ 2 000 FCFA fún Mali, 3 000 FCFA fún Senegal.
```

Le modèle apprend à répondre dans les deux langues, tant que vous gardez la même structure `Q:` / `A:`.

### Limiter la taille du contexte multilingue  

Si vous avez plus de 3 langues, créez **un prompt par langue** et choisissez celui qui correspond à la langue détectée. Cela évite de diluer le contexte et garde le coût token raisonnable.

## Validation et contrôle des réponses  

### Mettre en place un filtre post‑traitement  

Après chaque appel API, appliquez une série de vérifications :  

1. **Longueur** – la réponse dépasse‑elle le nombre de mots attendu ?  
2. **Présence d’un refus** – le texte contient‑il « Je ne sais pas » ou « Je ne suis pas en mesure » ?  
3. **Mots interdits** – assurez‑vous qu’aucun terme sensible (ex. : prix non autorisé) ne figure.  

Dans Make, le module *“Router”* peut diriger la réponse vers un scénario de **re‑ask** (re‑question) si l’une des règles échoue.

### Utiliser des tests automatisés (A/B)  

- **Créer un jeu de test** : 20 questions réelles tirées du support client.  
- **Comparer** la réponse du chatbot avec la réponse attendue (manuelle).  
- **Calculer le taux de précision** : `nb_correct / nb_total`.  

Un objectif réaliste pour un premier déploiement : **≥ 80 %** de réponses correctes.

### Détecter les hallucinations  

Les hallucinations sont souvent caractérisées par :  

- **Informations trop précises** (ex. : un numéro de suivi qui n’existe pas).  
- **Contradictions** avec les données du catalogue (ex. : prix différent).  

Pour les limiter :  

- **Rappel de contexte** – incluez toujours les champs clés (SKU, prix) dans le prompt lorsqu’une question porte sur un produit.  
- **Cross‑check** – après la réponse, interrogez une petite base de données (Google Sheet, Airtable) pour vérifier la cohérence.  

Exemple de vérification dans Make :  

```text
Si (réponse contient "SKU") alors
   chercher le SKU dans la table Produits
   si prix différent → envoyer alerte à l’opérateur
```

## Mise en pratique avec une plateforme no‑code (Make + OpenAI)  

### Étape 1 : Importer vos données  

1. **Google Sheet** : créez trois feuilles : `FAQ`, `Catalogue`, `PolitiqueRetour`.  
2. Dans Make, ajoutez le module *“Watch Rows”* pour chaque feuille.  

### Étape 2 : Construire le prompt dynamique  

- **Router** : selon le mot‑clé de la question (ex. : « livraison », « retour », « prix »), choisissez la source de données.  
- **Aggregator** : récupérez les 2‑3 exemples les plus pertinents (module *“Search Rows”* avec filtre sur `catégorie`).  
- **Compose Text** : assemblez le template présenté plus haut, en injectant les exemples et la question de l’utilisateur.  

### Étape 3 : Appeler l’API OpenAI  

Utilisez le module *“HTTP – Make a request”* :  

| Paramètre | Valeur |
|-----------|--------|
| Method    | POST |
| URL       | `https://api.openai.com/v1/chat/completions` |
| Headers   | `Authorization: Bearer {{openai_key}}`<br>`Content-Type: application/json` |
| Body (raw) | Le JSON généré par le module *Compose Text* (voir exemple cURL) |

### Étape 4 : Post‑traitement et réponse à l’utilisateur  

- **Parse JSON** : extraire le champ `content` du tableau `choices`.  
- **Router** : appliquer les filtres de validation décrits précédemment.  
- **Send Message** : selon le canal (WhatsApp via Twilio, Facebook Messenger, widget web), renvoyer la réponse.  

### Astuce de performance  

- **Cache** les réponses fréquentes : si la même question apparaît plusieurs fois, stockez la réponse dans une table `Cache` (clé = hash de la question). Avant d’appeler l’API, vérifiez le cache. Cela réduit les coûts et le temps de latence.  

## Points clés  

- **Collecte ciblée** : ne gardez que les informations réellement utiles (FAQ, catalogue, politique).  
- **Structure minimaliste** : chaque exemple doit suivre le même format `Q:` / `A:` et rester sous 2 000 tokens.  
- **Prompt template** : définissez le rôle, le ton, le format de sortie et la règle de refus dès le départ.  
- **Few‑shot efficace** : 2‑3 exemples pertinents suffisent ; choisissez-les en fonction de la catégorie de la question.  
- **Multilinguisme** : créez des variantes locales, mais limitez chaque appel à une seule langue détectée.  
- **Contrôle post‑traitement** : filtrez la