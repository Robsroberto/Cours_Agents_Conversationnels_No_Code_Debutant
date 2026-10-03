## Tableau de bord : visualiser les indicateurs clés de performance  

Un chatbot, même sans code, génère des données à chaque interaction. La première étape pour mesurer son impact consiste à configurer un tableau de bord qui agrège ces données en temps réel.  

| KPI                     | Pourquoi c’est important                                               | Source de donnée typique |
|-------------------------|------------------------------------------------------------------------|--------------------------|
| **Taux de résolution** | % de conversations terminées sans intervention humaine.               | Log de fin de session (flag *resolved*) |
| **Temps moyen de réponse** | Rapidité perçue par le client; influence la satisfaction.            | Horodatage du message utilisateur → premier message du bot |
| **Score de satisfaction (CSAT)** | Note donnée par l’utilisateur après la session (1‑5 étoiles).   | Prompt de feedback intégré à la fin du flux |
| **Taux d’abandon**      | % d’utilisateurs qui quittent avant la fin du dialogue.               | Sessions sans flag *resolved* |
| **Nombre d’escalades**  | Volume d’appels transférés à un agent humain.                         | Trigger d’escalade configuré dans le workflow |

### Créer le tableau de bord dans la plateforme no‑code  

1. **Collecte des logs** – La plupart des outils (Landbot, Botpress Cloud, Make) offrent un module « Analytics ». Activez le suivi des événements *session_start*, *message_sent*, *session_end* et *escalation*.  
2. **Export vers un tableur ou un BI** – Connectez le module d’analytics à Google Sheets, Airtable ou Power BI via un webhook. Exemple avec Make :  

```yaml
# Scenario Make
- Trigger: Webhook (receives JSON from chatbot)
- Action: Google Sheets > Add a row
  Mapping:
    Date: {{trigger.payload.timestamp}}
    SessionID: {{trigger.payload.session_id}}
    Event: {{trigger.payload.event}}
    Value: {{trigger.payload.value}}
```

3. **Construire les visualisations** – Dans Google Data Studio ou Power BI, créez des graphiques :  
   * Courbe du temps moyen de réponse par jour.  
   * Camembert du taux de résolution vs escalades.  
   * Histogramme du CSAT.  

4. **Alertes automatiques** – Configurez des notifications (Slack, WhatsApp) lorsqu’un KPI chute sous un seuil (ex. : taux de résolution < 80 %).  

> **Astuce :** pour les PME africaines où la connectivité peut être intermittente, privilégiez des exports locaux (CSV) et un tableau de bord hébergé sur un serveur interne afin d’éviter les pertes de données.  

---

## A/B testing des prompts : optimiser le langage du bot  

Le **prompt** détermine la façon dont le modèle génère ses réponses. Un petit changement de formulation peut augmenter le taux de résolution de façon significative.  

### Méthodologie pas à pas  

| Étape | Action | Outil recommandé |
|------|--------|------------------|
| **1. Définir l’hypothèse** | « Un prompt plus orienté « Comment pouvons‑nous vous aider ? » augmentera le CSAT. » | Aucun |
| **2. Créer deux variantes** | *Version A* : « Que puis‑je faire pour vous ? »<br>*Version B* : « Comment pouvons‑nous vous aider aujourd’hui ? » | Dans le builder de prompts de la plateforme |
| **3. Diviser le trafic** | 50 % des utilisateurs voient A, 50 % voient B. | Fonction *Split traffic* de Landbot ou *Randomizer* de Make |
| **4. Collecter les KPI** | Suivre CSAT et taux de résolution pour chaque groupe. | Tagger chaque session avec `variant=A` ou `variant=B` |
| **5. Analyser** | Utiliser un test de chi‑carré ou un test t pour vérifier la différence statistique. | Google Sheets + fonction `=TTEST()` |

### Exemple de configuration dans Landbot  

```json
{
  "type": "randomizer",
  "probabilityA": 0.5,
  "probabilityB": 0.5,
  "blocks": {
    "A": {"type":"send_message","text":"Que puis‑je faire pour vous ?"},
    "B": {"type":"send_message","text":"Comment pouvons‑nous vous aider aujourd’hui ?"}
  }
}
```

Après une semaine d’expérimentation, exportez les logs, calculez le CSAT moyen par variante et choisissez la version gagnante.  

> **Cas d’usage africain** : un service de micro‑finance en Côte d’Ivoire a constaté que la formulation « Quel service cherchez‑vous ? » (Version A) obtenait un CSAT de 3,8/5, tandis que « Comment pouvons‑nous vous accompagner dans votre projet ? » (Version B) a atteint 4,2/5, justifiant le basculement vers la version B.  

---

## Réglage de la température : trouver le bon équilibre entre créativité et précision  

Dans les modèles de génération de texte, la **température** contrôle la probabilité de choisir le token suivant :  

| Température | Comportement du modèle |
|-------------|------------------------|
| **0.0 – 0.3** | Réponses très déterministes, peu de variation. Idéal pour des FAQ strictes. |
| **0.4 – 0.7** | Compromis : réponses cohérentes avec une petite marge de créativité. |
| **0.8 – 1.0** | Réponses très variées, risque d’incohérence. Utilisé pour brainstorming ou storytelling. |

### Procédure de réglage  

1. **Définir le scénario** – Si le bot répond à des questions juridiques (ex. : conditions de remboursement), gardez la température ≤ 0.3. Pour une suggestion de recettes locales, testez 0.6–0.8.  
2. **Créer une variable de température** – La plupart des plateformes exposent ce paramètre dans le bloc d’appel API. Exemple avec Make :  

```json
{
  "model": "gpt-3.5-turbo",
  "messages": [{"role":"user","content":"{{payload.user_message}}"}],
  "temperature": {{temperature}}
}
```

3. **Lancer un test en série** – Variez la température par incréments de 0.1 sur 100 sessions, puis comparez les KPI (taux de résolution, CSAT).  
4. **Choisir la valeur optimale** – Sélectionnez la température qui maximise le CSAT tout en maintenant un taux de résolution ≥ 85 %.  

> **Illustration** : un e‑commerce basé à Lagos a testé la température 0.2, 0.5 et 0.8 pour le bot « Quel produit vous conseille‑t‑je ? ». Le taux de résolution était respectivement 92 %, 88 % et 71 %. Le CSAT était meilleur à 0.5 (4,5/5) qu’à 0.2 (4,2/5). Le compromis retenu a donc été 0.5.  

---

## Enrichir le bot avec de nouvelles intents : boucle d’amélioration continue  

Les **intents** représentent les intentions que l’utilisateur peut exprimer. Au fil du temps, de nouveaux besoins apparaissent (ex. : demande de suivi de livraison, question sur les frais de douane).  

### Détection des lacunes  

1. **Analyser les logs « non compris »** – Recherchez les messages qui n’ont pas déclenché d’intent (souvent marqués `intent=none`).  
2. **Classer les requêtes fréquentes** – Utilisez un tableur pour compter les occurrences. Par exemple :  

| Requête non reconnue | Occurrences (semaine) |
|----------------------|-----------------------|
| « Où est mon colis ? » | 42 |
| « Comment payer en mobile money ? » | 27 |
| « Quel est le taux de change aujourd’hui ? » | 15 |

3. **Prioriser** – Sélectionnez les 2‑3 intents les plus fréquents qui apportent le plus de valeur métier.  

### Créer et entraîner une nouvelle intent sans code  

1. **Définir des exemples d’utterances** – Au moins 5 variantes pour chaque nouvelle intention.  
2. **Ajouter l’intent dans le builder** – Dans Botpress Cloud, cliquez *Add Intent* → saisissez le nom (ex. : `track_order`).  
3. **Associer les réponses** – Vous pouvez soit créer un **prompt** statique (FAQ) soit un **prompt dynamique** qui interroge une API interne (ex. : suivi de colis).  

#### Exemple de prompt dynamique (Make + API de suivi)  

```yaml
# Scenario Make
- Trigger: Webhook (intent = track_order)
- Action: HTTP > GET https://api.maboutique.com/track?order_id={{trigger.payload.order_id}}
- Action: OpenAI > Chat Completion
  Prompt: |
    Vous êtes un assistant client. Répondez à la question suivante en français, en vous basant sur les informations suivantes :
    {{http.response.body}}
  Temperature: 0.3
- Action: Respond to user (via chatbot)
```

4. **Tester** – Simulez les nouvelles utterances dans le mode *Test* de la plateforme et vérifiez que l’intent est bien détecté.  

### Boucle de rétro‑action utilisateur  

Intégrez un petit questionnaire après chaque interaction :  

```json
{
  "type":"send_message",
  "text":"Cette réponse vous a‑t‑elle été utile ? 👍👎"
}
```

En fonction du vote, vous pouvez :

- **Automatiquement créer un ticket d’amélioration** (Zapier → Trello) si le score est négatif.  
- **Mettre à jour la base d’exemples** en ajoutant la requête négative à l’intent correspondant (voir chapitre 6 pour les bonnes pratiques de gestion de données).  

---

## Automatiser le reporting hebdomadaire  

Pour ne pas perdre de vue les performances, mettez en place un **rapport automatisé** qui arrive chaque lundi matin dans la boîte mail du responsable.  

1. **Créer un scénario Make** qui s’exécute chaque semaine.  
2. **Récupérer les KPI** depuis le tableur ou la base de données.  
3. **Générer un PDF** avec les graphiques (utilisez le module *PDF Generator*).  
4. **Envoyer le mail** via Gmail ou Outlook.  

```yaml
# Scenario Make – Rapport hebdo
- Trigger: Scheduler (Every Monday 08:00)
- Action: Google Sheets > Get range (KPI sheet)
- Action: PDF Generator > Create PDF (template.html)
- Action: Gmail > Send email
  To: manager@entreprise.com
  Subject: Rapport chatbot – Semaine {{date}}
  Attachment: {{pdf.file}}
```

Ce processus garantit que les décisions d’optimisation sont basées sur des données fraîches, même si vous n’avez pas le temps de consulter le tableau de bord quotidiennement.  

---

## Points clés à retenir  

- **Visualisation continue** : un tableau de bord centralisé (Google Data Studio, Power BI) permet de surveiller le taux de résolution, le temps moyen de réponse, le CSAT, le taux d’abandon et les escalades.  
- **A/B testing structuré** : créez deux variantes de prompts, partagez le trafic, puis analysez les KPI avec des tests statistiques pour choisir la version la plus performante.  
- **Température calibrée** : utilisez une température basse (≤ 0.3) pour les réponses factuelles, une température moyenne (0.4‑0.7) pour les suggestions et une élevée (≥ 0.8) uniquement pour des scénarios créatifs.  
- **Intents évolutifs** : exploitez les logs « non compris », priorisez les requêtes fréquentes, ajoutez de nouvelles intents avec des exemples pertinents et testez-les immédiatement.  
- **Boucle de feedback** : le petit questionnaire post‑conversation alimente automatiquement un backlog d’amélioration, assurant que le chatbot s’ajuste aux besoins réels des utilisateurs.  
- **Reporting automatisé** : un scénario hebdomadaire qui compile les KPI, génère un PDF et le transmet aux décideurs libère du temps et maintient la visibilité sur la santé du bot.  

En appliquant ces pratiques d’analyse et d’optimisation, votre chatbot IA génératif deviendra non seulement un canal de service client fiable, mais aussi un levier de croissance mesurable pour votre activité.