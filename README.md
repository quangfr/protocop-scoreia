# Assistant Contrat SCORE – Documentation fonctionnelle

Ce dépôt contient deux prototypes d'assistants conversationnels destinés à la création de contrats SCORE. Ce document décrit, pour chaque version, le contexte d'usage, les données manipulées, l'interface utilisateur, les règles métier implémentées et les aspects techniques clés.

## index.html – Assistant latéral guidé

### Contexte
- Panneau latéral flottant activable depuis un bouton flottant « 🤖 Assistant contrat » intégré à une page SCORE existante.
- Vise une expérience guidée mixant saisie libre, sélection par chips et conversation pas-à-pas.
- Destiné aux utilisateurs opérationnels qui préparent un contrat en renseignant rapidement les informations principales.

### Données manipulées
- Type de contrat (`contractType`) : valeurs préconfigurées (`PBH`, `Time & Material`, `Full Support`, `On Condition`, `Exchange`, `Inspection`).
- Type de moteur (`engine`) : liste restreinte (`LEAP-1A`, `LEAP-1B`, `CFM56-5B`, `CFM56-7B`, `CFM56-3`, `CF6-80E1`).
- Client (`client`) : suggestions (`Air France`, `Transavia France`, `easyJet`, `Ryanair`, `KLM`, `Lufthansa`, `Safran Test Fleet`) mais saisie libre autorisée.
- Identifiant Contratèque (`contratequeId`) : format `CFM-2025-XXXXX` (lettres/chiffres, X ∈ [A-Z0-9] hors O/I/0/1 via alphabet personnalisé).
- Référence contrat (`contractRef`) : format `Z-ZZZZ` (Z chiffre 1–9, ZZZZ quatre chiffres).
- Aperçu JSON (`preview`) comprenant également la date ISO du jour et la source `assistant-ia-sidepanel`.

### Interface
- Bouton flottant (FAB) pour ouvrir/fermer le panneau.
- Panneau vertical avec sections : conversation guidée, parsing de commande naturelle, chips de sélection, formulaires d’identifiants, aperçu JSON.
- Conversation affichée en bulles (assistant/utilisateur), champs de saisie et boutons « Envoyer » / « Recommencer ».
- Zone de texte pour coller une phrase en langage naturel et déclencher l’analyse automatique.
- Composants chips pour type de contrat, moteur et client (avec sélection visuelle).
- Inputs standards pour client, identifiant Contratèque, référence contrat, et boutons de génération automatique.
- Carte d’aperçu JSON avec actions « Créer le contrat » et « Copier JSON ».

### Règles métier / validation
- Parsing NL : expression régulière pour détecter type, moteur, client, identifiant et référence à partir d’un texte libre.
- Conversation guidée en plusieurs étapes (type, moteur, client, identifiant, référence) avec possibilité de répondre « auto » pour générer un identifiant/référence valide.
- Validation à la volée :
  - `contratequeId` doit respecter `^CFM-2025-[A-Z0-9]{5}$` ; message d’erreur si invalide.
  - `contractRef` doit respecter `^[1-9]-\d{4}$` ; message d’erreur si invalide.
- Bouton « Créer le contrat » affiche une alerte si des champs manquent ou sont invalides, sinon trace l’événement dans la console et confirme la création (simulation).
- Copie JSON via API `navigator.clipboard`, avec feedback visuel.

### Technique
- Application HTML/CSS/JS autonome sans dépendances externes.
- Style en CSS inline dans `<style>` ; palette définie via variables CSS.
- Gestion d’état par objet `state` synchronisé avec les inputs et la conversation.
- Générateurs d’identifiant et référence utilisant alphabet restreint et padding numérique.
- Interaction DOM pure (pas de framework) : construction dynamique des chips, messages et prévisualisation JSON.
- Conversation orchestrée via tableau `steps` et fonction `handleConversation` déclenchée par saisie utilisateur ou bouton.

## index2.html – Assistant pas-à-pas intégral

### Contexte
- Variante centrée sur un flux pas-à-pas « full screen panel », pensée pour guider l’utilisateur via une machine à états.
- L’expérience commence vide : ouverture du panneau déclenche immédiatement la première étape.
- Convient à un onboarding contrôlé où chaque champ est validé successivement.

### Données manipulées
- Même jeux de données principaux que `index.html` : type de contrat, moteur, client, identifiant Contratèque, référence contrat.
- Etat conversationnel `state` stocke les cinq champs ; `steps` décrit la progression (intro → type → moteur → client → identifiant → référence → revue → confirmation).
- Payload final envoyé/affiché contient date ISO, champs saisis et source `assistant-ia-stepper`.

### Interface
- Bouton flottant « 🤖 Assistant » ouvrant un panneau coulissant.
- En-tête avec barre de progression (élément `.bar`) reflétant l’étape courante.
- Zone de chat verticale avec messages horodatés (`now()`), bulles assistant/utilisateur et séparateurs `hr` entre étapes.
- Suggestions rapides (`chips`) pour chaque étape, y compris auto-génération et saisie libre.
- Champ de saisie principal + bouton « Envoyer » ; touches Entrée prises en compte.
- Carte récapitulative (`.card`) en étape revue avec possibilité de cliquer chaque valeur pour revenir à l’étape correspondante.
- Étape finale affichant le payload JSON dans `<pre>` avec actions « Créer maintenant », « Copier le JSON », « Terminer ».

### Règles métier / validation
- Pré-remplissage automatique via analyse d’un texte libre (`parseFromFree`) dès que l’utilisateur colle ou envoie un message global.
- Génération d’identifiants/références via `genId` et `genRef`, partagés avec `index.html` (mêmes règles de format).
- Validation stricte :
  - Identifiant Contratèque : `isId` vérifie `^CFM-2025-[A-Z0-9]{5}$`.
  - Référence contrat : `isRef` vérifie `^[1-9]-\d{4}$`.
- Système `awaiting` pour exiger une saisie conforme lorsqu’un champ est en édition ; message d’erreur si format incorrect.
- Étape « Review » bloque la validation tant que tous les champs ne sont pas valides.
- Actions finales : journalisation console `CREATE_CONTRACT` et confirmation textuelle ; copie JSON via `navigator.clipboard`.

### Technique
- Application autonome (HTML/CSS/JS vanilla) ; aucun bundle requis.
- Machine à états pilotée par tableau `steps` et index `i`; fonction `next()` avance ou rejoue une étape.
- Barre de progression mise à jour par pourcentage prédéfini par étape.
- Gestion des chips avec boutons dynamiques et fonction `pickChip` pour marquage visuel.
- Pré-remplissage : `prefillFromFreeText` inspecte l’input actuel pour détecter des valeurs reconnues et informer l’utilisateur.
- Historique chat enrichi avec horodatage et messages de feedback pour guider l’utilisateur.
- Réinitialisation complète via `reset()` (efface état, remise à zéro progression, relance l’intro).

---

Ces prototypes peuvent être adaptés pour intégrer un backend (API SCORE ou Firestore) en remplaçant les simulations `console.log`/`alert` par des appels réseau et en sécurisant l’accès au presse-papiers selon les politiques du navigateur.
