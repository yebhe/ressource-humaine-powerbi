# 📁 Données

Le fichier CSV est **inclus** dans ce dépôt, dans ce dossier : [`HRDataset_v14.csv`](HRDataset_v14.csv). Ce sont des données de démonstration, fictives, qui ne représentent aucune entreprise réelle.

## Origine

Jeu de données public à vocation pédagogique, **HRDataset_v14** (Human Resources Data Set), utilisé ici comme support d'apprentissage Power BI.

## Fichier

Un seul fichier plat, une ligne par employé. Power Query l'éclate ensuite en 1 table de faits et 7 dimensions, auxquelles s'ajoute le calendrier `dim_date`.

| Fichier | Contenu | Lignes | Colonnes |
|---|---|---:|---:|
| `HRDataset_v14.csv` | Employés actifs et partis (salaire, dates, poste, département, enquêtes, absences) | 311 | 36 |

Parmi les 36 colonnes, 9 identifiants d'origine (`MarriedID`, `MaritalStatusID`, `GenderID`, `EmpStatusID`, `DeptID`, `PerfScoreID`, `FromDiversityJobFairID`, `PositionID`, `ManagerID`) sont supprimés à l'import. Les clés des dimensions sont recréées dans Power Query. 27 colonnes sont conservées.

### Périodes couvertes

| Date | Première | Dernière |
|---|---|---|
| Embauche | 09/01/2006 | 09/07/2018 |
| Départ | 30/08/2010 | 10/11/2018 |
| Dernière évaluation | 14/07/2010 | 28/02/2019 |

## Brancher les données au rapport

1. Ouvrir `rh_p4_last.pbix` dans **Power BI Desktop**.
2. **Accueil → Transformer les données → Gérer les paramètres**.
3. Renseigner `chemin_fichier` avec le chemin **complet du fichier** `HRDataset_v14.csv` (nom et extension compris, pas seulement le dossier).
   Exemple : `C:\projets\rh-effectifs-attrition\data\HRDataset_v14.csv`
4. Vérifier `DateRef` (28/02/2019, date de référence du calcul de l'âge et de l'ancienneté).
5. Fermer l'éditeur et cliquer sur **Actualiser**.

## Points d'attention

- Le fichier est lu en UTF-8, séparateur virgule. Les salaires, l'enquête d'engagement et toutes les dates sont convertis avec la locale **en-US** (point décimal, dates au format américain). Une conversion en fr-FR décalerait les dates et casserait les décimaux.
- Le code postal (`Zip`) est converti en texte et complété à 5 chiffres pour garder les zéros initiaux.
- `DateofTermination` est vide pour les 207 employés actifs. Les 104 partis ont tous une date de départ.
- Le jeu d'origine est américain (États, codes postaux). Les montants sont affichés en euros dans le rapport (voir le [README principal](../README.md#-limites-connues)).