# TP Azure - Optimisation des Coûts Paris 2024

## Correction détaillée

---

## Pré-requis

Avant de débuter ce TP, les éléments suivants sont nécessaires :

- **Compte Azure** avec une subscription disposant au minimum des droits **Cost Management Contributor** ou **Contributor** (requis pour la création de budgets et l'accès aux données de coûts)
- **Azure CLI** installé localement, ou accès à **Azure Cloud Shell** via le portail Azure
- Une **resource group dédiée** (exemple : `rg-paris2024`) 
- Un **Storage Account** pour héberger les jeux de données

### Bonne pratique : Stratégie de tagging systématique

L'application d'un tag `project=paris2024` sur l'ensemble des ressources liées au TP permet un filtrage précis de la consommation budgétaire.

**Commande de tagging :**
```bash
az tag update --resource-id <resource-id> --operation Merge --tags project=paris2024 environment=dev
```

**Recommandation :** Les ressources temporaires peuvent être marquées avec `lifecycle=ephemeral` pour faciliter leur identification et suppression après le TP.

---

## Étape 1 — Analyse des coûts avec Azure Cost Management

**Objectif :** Identifier et analyser les postes de dépenses liés au traitement des datasets Paris 2024.

### 1.1 Configuration de l'analyse des coûts

1. Accéder au portail Azure : **Cost Management + Billing** → **Cost Management** → **Cost analysis**
2. Appliquer les filtres suivants :
   - **Scope :** Subscription ou Resource group (exemple : `rg-paris2024`)
   - **Group by :** Service, Resource group, Resource, Tags (filtrer sur `project=paris2024`)
   - **Granularity :** 
     - **Daily** : pour l'observation d'un job spécifique
     - **Monthly** : pour le suivi du budget mensuel
3. Sauvegarder la vue configurée (fonction **Save as**) pour une réutilisation ultérieure

### 1.2 Actions d'optimisation

**Export des données :**
- Configurer un export CSV automatique via **Cost Management exports** pour des analyses offline
- Planifier des exports récurrents selon les besoins

**Azure Advisor :**
- Activer l'onglet **Cost** pour bénéficier de recommandations automatiques :
  - Redimensionnement des ressources sous-utilisées
  - Suggestions de Reserved Instances (RI) ou Savings Plans

### 1.3 Stratégie de tagging avancée

L'utilisation de tags structurés permet une granularité fine dans l'analyse des coûts :

| Tag | Valeurs possibles | Usage |
|-----|-------------------|-------|
| `project` | `paris2024` | Identification du projet |
| `environment` | `dev`, `prod`, `test` | Séparation des environnements |
| `dataset` | `events`, `tickets`, `venues` | Type de données |
| `lifecycle` | `hot`, `cold`, `ephemeral` | Politique de rétention |

---

## Étape 2 — Mise en place de l'auto-scaling avec VM Scale Sets

**Objectif :** Optimiser la consommation en adaptant automatiquement le nombre d'instances en fonction de la charge réelle.

### 2.1 Création d'un Virtual Machine Scale Set (VMSS)

```bash
# Création du resource group (si nécessaire)
az group create -n rg-paris2024 -l francecentral

# Création du VM Scale Set
az vmss create \
  -g rg-paris2024 \
  -n vmss-paris2024 \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys \
  --instance-count 2 \
  --upgrade-policy-mode automatic \
  --tags project=paris2024
```

**Alternative :** Le VMSS peut également être créé via le portail Azure ou un template ARM selon les préférences de déploiement.

### 2.2 Configuration des règles d'autoscaling

L'autoscaling basé sur les métriques CPU permet une adaptation dynamique de la capacité.

**Création de la configuration autoscale :**
```bash
# Configuration de base (minimum 2, maximum 10 instances, par défaut 2)
az monitor autoscale create \
  -g rg-paris2024 \
  --resource "/subscriptions/<SUB_ID>/resourceGroups/rg-paris2024/providers/Microsoft.Compute/virtualMachineScaleSets/vmss-paris2024" \
  --min-count 2 --max-count 10 --count 2 --name autoscale-paris2024
```

**Règle de scale-out (montée en charge) :**
```bash
# Ajout d'une instance si CPU > 75% pendant 5 minutes
az monitor autoscale rule create \
  -g rg-paris2024 --autoscale-name autoscale-paris2024 \
  --condition "Percentage CPU > 75 avg 5m" --scale out 1
```

**Règle de scale-in (réduction de charge) :**
```bash
# Retrait d'une instance si CPU < 30% pendant 10 minutes
az monitor autoscale rule create \
  -g rg-paris2024 --autoscale-name autoscale-paris2024 \
  --condition "Percentage CPU < 30 avg 10m" --scale in 1
```

### 2.3 Test et validation

**Méthode de test :**
1. Lancer un job de traitement intensif (simulation avec `stress-ng` sur une VM)
2. Observer dans le portail Azure Monitor la montée en charge des instances
3. Vérifier la descente en charge après arrêt du job

### 2.4 Optimisations avancées

**Profiles d'autoscaling :**
- Profil "heures de bureau" : seuils plus agressifs
- Profil "nuit/weekend" : configuration économe

**Spot VMs et Azure Batch :**
- Pour les traitements batch tolérant les interruptions
- Réduction des coûts jusqu'à 90% par rapport aux instances régulières
- Attention : risque d'interruption en cas de manque de capacité Azure

---

## Étape 3 — Configuration des budgets et alertes

**Objectif :** Établir un contrôle budgétaire avec notifications avant dépassement.

### 3.1 Création via le portail Azure

**Procédure :**
1. Naviguer vers **Cost Management + Billing** → **Budgets** → **Add**
2. Configuration du budget :
   - **Scope :** Subscription ou resource group spécifique (`rg-paris2024`)
   - **Type :** Monthly (mensuel)
   - **Amount :** Montant alloué (exemple : $500)
   - **Alerts :** Définir les seuils d'alerte
     - 80% : alerte préventive
     - 90% : alerte critique
     - 100% : dépassement

3. Ajouter les destinataires des notifications par email

### 3.2 Création via Azure CLI

```bash
az consumption budget create \
  --budget-name "paris2024-budget" \
  --amount 500 \
  --time-grain monthly \
  --category cost \
  --start-date 2025-10-01 \
  --end-date 2026-10-01
```

**Note :** Le groupe de commandes `consumption` peut nécessiter une version récente d'Azure CLI. Adapter les dates selon la période du TP.

### 3.3 Stratégies d'alerte avancées

**Intégrations possibles :**
- **Webhooks** vers Slack ou Microsoft Teams pour des notifications en temps réel
- **Action Groups** pour déclencher des runbooks automatiques (arrêt de ressources, notifications escaladées)

**Budgets multiples :**
- Un budget par resource group pour séparer les environnements
- Budgets par département ou équipe pour la répartition des coûts

---

## Étape 4 — Optimisation du stockage (Storage Tiering & Lifecycle Management)

**Objectif :** Réduire les coûts de stockage en déplaçant automatiquement les données peu accédées vers des niveaux de stockage économiques.

### 4.1 Activation du suivi du dernier accès (Last Access Time Tracking)

Cette fonctionnalité permet de baser les politiques de lifecycle sur la dernière date d'accès réelle aux données.

```bash
az storage account blob-service-properties update \
  --resource-group rg-paris2024 \
  --account-name mystorageparis2024 \
  --enable-last-access-tracking true
```

**Impact :** Permet la création de règles basées sur `lastAccessTime` en plus de `lastModifiedTime`.

### 4.2 Création d'une politique de lifecycle management

**Objectif de la règle :** Déplacer automatiquement les blobs non modifiés depuis plus de 30 jours vers le tier Archive.

**Fichier de configuration `policy.json` :**

```json
{
  "policy": {
    "rules": [
      {
        "name": "paris2024-move-to-archive-30d",
        "enabled": true,
        "type": "Lifecycle",
        "definition": {
          "filters": {
            "prefixMatch": ["paris2024/"],
            "blobTypes": ["blockBlob"]
          },
          "actions": {
            "baseBlob": {
              "tierToArchive": {
                "daysAfterModificationGreaterThan": 30
              }
            }
          }
        }
      }
    ]
  }
}
```

**Application de la politique :**

```bash
az storage account management-policy create \
  --resource-group rg-paris2024 \
  --account-name mystorageparis2024 \
  --policy @policy.json
```

### 4.3 Considérations importantes

**Tier Archive - Points d'attention :**
- **Rehydration nécessaire :** L'accès à un blob archivé nécessite une opération de restauration
- **Délai de rehydration :** Plusieurs heures selon la priorité choisie (standard ou haute priorité)
- **Coût de rehydration :** Facturation au Go restauré
- **Recommandation :** Réserver ce tier aux données rarement consultées (archives historiques, conformité réglementaire)

**Stratégies de tiering progressif :**

| Âge des données | Tier recommandé | Cas d'usage |
|-----------------|-----------------|-------------|
| 0-30 jours | Hot | Données actives, accès fréquent |
| 30-90 jours | Cool | Données d'analyse récente |
| 90-180 jours | Cool | Archives récentes |
| > 180 jours | Archive | Conformité, archives historiques |

**Affinage de la politique :**
- Utiliser `prefixMatch` pour cibler des containers ou sous-dossiers spécifiques
- Exemple : `paris2024/schedules/` pour ne cibler que les anciens plannings
- Combiner plusieurs règles pour différents types de données

---

## Stratégies complémentaires d'optimisation

### 5.1 Reserved Instances et Savings Plans

**Quand les utiliser :**
- Machines virtuelles avec utilisation stable et prévisible 24/7
- Charges de travail de production à long terme

**Options disponibles :**

| Option | Durée | Flexibilité | Économies |
|--------|-------|-------------|-----------|
| **Reserved Instances** | 1 ou 3 ans | Limitée (VM type, région fixes) | Jusqu'à 72% |
| **Savings Plans** | 1 ou 3 ans | Élevée (changement VM, région) | Jusqu'à 65% |

**Recommandation :** Analyser l'historique d'utilisation via Cost Analysis sur 3 mois minimum avant tout engagement.

### 5.2 Spot VMs et Azure Batch

**Cas d'usage adaptés au projet Paris 2024 :**
- Traitements batch sur archives (ré-extraction de données historiques)
- Analyses statistiques non urgentes
- Processing de données en parallèle tolérant les interruptions

**Économies potentielles :** Jusqu'à 90% par rapport aux instances on-demand

**Prérequis :** Le code applicatif doit gérer gracieusement les interruptions (checkpointing, reprises).

### 5.3 Auto-shutdown pour VMs de développement

**Configuration recommandée :**
- Auto-shutdown quotidien à 19h00 (hors heures de travail)
- Automation start/stop pour environnements de développement
- Notifications avant shutdown

**Impact :** Économies de ~50% sur les VMs non-production utilisées uniquement en journée.

### 5.4 Azure Advisor - Recommandations continues

**Processus recommandé :**
1. Consulter l'onglet **Cost** d'Azure Advisor hebdomadairement
2. Prioriser les recommandations par impact financier estimé
3. Appliquer les "quick wins" (VMs sous-utilisées, disques non attachés)
4. Documenter les actions prises et leurs résultats

**Types de recommandations typiques :**
- Redimensionnement ou arrêt de VMs sous-utilisées (< 5% CPU)
- Suppression de disques non attachés
- Passage en Reserved Instances pour charges stables
- Optimisation de SKU de bases de données

---

## Checklist d'exécution du TP

**Ordre recommandé pour la réalisation :**

- [ ] **1. Initialisation**
  - Créer le resource group `rg-paris2024`
  - Créer le Storage Account `mystorageparis2024`
  - Uploader les datasets de test

- [ ] **2. Tagging**
  - Appliquer le tag `project=paris2024` sur toutes les ressources créées
  - Ajouter les tags complémentaires (`environment`, `dataset`, `lifecycle`)

- [ ] **3. Analyse des coûts**
  - Configurer Cost Analysis avec filtrage par tag `project=paris2024`
  - Sauvegarder la vue personnalisée
  - Exporter un premier rapport CSV

- [ ] **4. Auto-scaling**
  - Créer le VMSS avec 2 instances initiales
  - Configurer les règles d'autoscale (min=2, max=10)
  - Tester avec un job de charge CPU
  - Valider la montée et descente automatique

- [ ] **5. Budget et alertes**
  - Créer un budget mensuel de $500
  - Configurer les alertes (80%, 90%, 100%)
  - Tester la réception des emails

- [ ] **6. Optimisation du stockage**
  - Activer Last Access Time Tracking
  - Créer et appliquer la policy de lifecycle (archive à 30 jours)
  - Vérifier l'application de la policy après 24h

- [ ] **7. Recommandations Advisor**
  - Consulter Azure Advisor (onglet Cost)
  - Identifier et appliquer au moins 2 recommandations
  - Documenter les économies estimées

---

## Pièges courants et solutions

### Erreur 1 : Absence de tagging

**Problème :** Impossible de filtrer les coûts par projet, analyse budgétaire imprécise.

**Solution :**
- Mettre en place une Azure Policy forçant les tags obligatoires
- Utiliser des templates ARM/Bicep avec tags pré-configurés
- Script d'audit régulier des ressources non taguées

### Erreur 2 : Archivage prématuré de données actives

**Problème :** Des données encore consultées sont déplacées vers Archive, entraînant des coûts de rehydration imprévus.

**Solution :**
1. Activer blob inventory pour analyser les patterns d'accès
2. Surveiller Last Access Time sur une période complète (minimum 30 jours) avant application de la policy
3. Commencer avec un tier Cool avant Archive
4. Monitorer les métriques de rehydration les premières semaines

### Erreur 3 : VM arrêtée mais toujours facturée

**Problème :** Confusion entre "Stopped" et "Deallocated". Une VM arrêtée (stopped) continue d'être facturée.

**Solution :**
- Toujours utiliser l'option **Stop (deallocate)** dans le portail
- Via CLI : `az vm deallocate` plutôt que `az vm stop`
- Vérifier le statut : une VM deallocated affiche "Stopped (deallocated)"

### Erreur 4 : Reserved Instances mal dimensionnées

**Problème :** Achat d'un engagement qui ne correspond pas à l'utilisation réelle, gaspillage budgétaire.

**Solution :**
- Analyser l'historique d'utilisation sur minimum 3 mois via Cost Analysis
- Utiliser Azure Advisor pour les recommandations de RI
- Privilégier les Savings Plans pour plus de flexibilité si l'architecture évolue
- Commencer par un engagement 1 an avant 3 ans

### Erreur 5 : Règles d'autoscaling trop agressives

**Problème :** Oscillations fréquentes (flapping) entre scale-out et scale-in, impact sur la stabilité.

**Solution :**
- Augmenter la fenêtre d'observation (cooldown period)
- Créer une hystérésis : seuils différents pour montée (>75%) et descente (<30%)
- Utiliser plusieurs métriques combinées (CPU + Memory + requests)
- Tester en situation réelle avant production

---

## Ressources officielles Microsoft

### Documentation de référence

- **Cost Management & Billing**
  - [Quickstart : Analyse des coûts](https://learn.microsoft.com/azure/cost-management-billing/)
  - [Tutoriel : Créer et gérer des budgets](https://learn.microsoft.com/azure/cost-management-billing/costs/tutorial-acm-create-budgets)

- **VM Scale Sets & Autoscaling**
  - [Documentation VMSS](https://learn.microsoft.com/azure/virtual-machine-scale-sets/)
  - [Azure Monitor autoscale - Exemples CLI](https://learn.microsoft.com/cli/azure/monitor/autoscale)

- **Storage Optimization**
  - [Lifecycle management et tiers de stockage](https://learn.microsoft.com/azure/storage/blobs/lifecycle-management-overview)
  - [Configuration du Last Access Time Tracking](https://learn.microsoft.com/azure/storage/blobs/lifecycle-management-policy-configure)

- **Optimisation globale**
  - [Azure Advisor - Recommandations de coûts](https://learn.microsoft.com/azure/advisor/advisor-cost-recommendations)
  - [Reserved Instances et Savings Plans](https://learn.microsoft.com/azure/cost-management-billing/reservations/)

---

## Conclusion

Ce TP couvre les principaux leviers d'optimisation des coûts sur Azure dans le contexte d'un projet de traitement de données (Paris 2024). Les techniques présentées sont applicables à tout type de workload cloud :

**Points clés à retenir :**
1. **Visibilité** : Le tagging et Cost Analysis sont essentiels pour comprendre les dépenses
2. **Élasticité** : L'autoscaling évite le sur-provisionnement
3. **Contrôle** : Les budgets et alertes préviennent les dépassements
4. **Optimisation continue** : Les policies de lifecycle et Azure Advisor automatisent l'optimisation

**Pratique professionnelle :**
Dans un contexte professionnel, ces pratiques doivent être :
- Documentées dans un Cloud Financial Management framework
- Auditées régulièrement (revue mensuelle minimum)
- Intégrées dans les processus CI/CD (Infrastructure as Code avec tags, policies)
- Partagées au sein des équipes (FinOps culture)
