# Hands-on with Cloud Cost Management and Resource Optimization (Paris 2024)

![Azure Logo](https://learn.microsoft.com/en-us/media/logos/logo_azure.svg)

## Objectif du TP

L’objectif de ce travail pratique est d’**optimiser les coûts et les ressources cloud sur Azure**, en appliquant des stratégies de gestion financière, d’auto-scaling, de surveillance et d’optimisation du stockage tout en traitant des **données simulées des Jeux Olympiques de Paris 2024** (horaires d’événements, sites, ventes de billets, etc.).

---

## Étape 1 : Analyse des coûts avec Azure Cost Management

**Objectif :**  
Utiliser les outils de **Cost Management** d’Azure pour analyser la consommation de ressources et identifier les leviers d’optimisation.

### Procédure

1. Accédez au [Portail Azure](https://portal.azure.com).  
2. Dans la barre de recherche, tapez **“Cost management + billing”**.  
3. Sélectionnez **Cost Management** puis ouvrez l’onglet **Analyse des coûts (Cost analysis)**.  
4. Filtrez vos données :
   - Période : 7 derniers jours ou 30 derniers jours  
   - Regroupement : par **service**, **groupe de ressources**, ou **abonnement**  
5. Identifiez les ressources les plus coûteuses (machines virtuelles, stockage, réseau).

### ✅ Résultat attendu
Une **vision claire de la répartition des coûts** par type de ressource et par projet (ici, l’analyse des données Paris 2024).

### Bonnes pratiques
- Utiliser des **groupes de ressources par projet** (ex. `rg-paris2024`) pour faciliter le suivi.
- Activer les **étiquettes (tags)** sur chaque ressource :
  ```bash
  az tag create --name project --values paris2024
  ```
  
## Étape 2 : Implémentation de l’Auto-Scaling pour les machines virtuelles
Objectif :
Mettre en place un Virtual Machine Scale Set (VMSS) pour ajuster automatiquement la capacité selon la charge CPU.

🔹 Création du Scale Set
```bash
Copier le code
az vmss create \
  --resource-group rg-paris2024 \
  --name vmss-paris2024 \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --instance-count 2 \
  --upgrade-policy-mode automatic \
  --vm-sku Standard_D2s_v5 \
  --tags project=paris2024
```

_Remarque_ :
Sous macOS (zsh), assurez-vous de ne pas laisser d’espace après chaque \.
L’image UbuntuLTS n’est plus valide — utilisez Ubuntu2204 ou Ubuntu2404.

### Définition des règles d’auto-scaling
Dans le portail Azure, ouvrez votre Scale Set.

Allez dans Mise à l’échelle (Scaling).

Créez des règles :

Scale out (ajout d’instances) : si la CPU > 75 % pendant 5 minutes

Scale in (réduction d’instances) : si la CPU < 30 % pendant 5 minutes

Instances minimum / maximum : 1 à 5

✅ Résultat attendu
Les machines virtuelles s’ajustent automatiquement selon la charge de traitement des données Paris 2024, optimisant ainsi les coûts et la performance.

## Étape 3 : Mise en place d’alertes budgétaires
Objectif :
Créer un budget mensuel et des alertes de seuil pour surveiller les dépenses liées au projet.

🔹 Étapes
Dans le portail Azure, ouvrez Cost Management + Billing.

Sélectionnez Budgets → Ajouter un budget.

Paramètres recommandés :

Montant : $500 / mois

Alertes à : 80 %, 90 %, 100 %

Notifications envoyées par e-mail

✅ Résultat attendu
Des notifications automatiques préviennent dès que les dépenses approchent du budget défini, permettant une gestion proactive des coûts.

## Étape 4 : Optimisation du stockage avec le tiering
Objectif :
Réduire les coûts en déplaçant les données peu consultées vers un niveau de stockage moins coûteux.

🔹 Étapes
Accédez à Stockage (Storage Accounts).

Sélectionnez votre compte de stockage, puis Conteneurs (Blobs).

Identifiez les fichiers peu consultés (ex. anciens horaires d’événements).

Modifiez leur niveau d’accès vers Archive :

```bash
Copier le code
az storage blob set-tier \
  --account-name mystorageparis2024 \
  --container-name paris2024data \
  --name historique-2023.json \
  --tier Archive
```

✅ Résultat attendu
Les données historiques ou rarement consultées sont archivées, réduisant les coûts de stockage tout en restant disponibles à long terme.

### Bonnes pratiques
Automatiser le tiering avec Azure Lifecycle Management :

Règle : déplacer les blobs inactifs depuis plus de 30 jours vers Cool ou Archive.

Séparer les conteneurs de données actives et historiques.

📊 Résumé des apprentissages
À la fin de ce TP, vous avez appris à :

Analyser vos coûts à l’aide d’Azure Cost Management.

Configurer l’auto-scaling pour ajuster les ressources selon la charge.

Créer et suivre un budget avec alertes de seuil.

Optimiser les coûts de stockage avec le tiering des données.

Ces actions illustrent comment réduire la facture Azure tout en maintenant la performance et la flexibilité nécessaires à un projet réel (ici, l’analyse des Jeux Olympiques de Paris 2024).

