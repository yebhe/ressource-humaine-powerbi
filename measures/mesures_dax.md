# 🧮 Mesures DAX

Le modèle contient **62 mesures** dans la table `_Mesures`. Ce document liste les **mesures de base** de chaque domaine, avec leur formule. Les mesures d'attrition ont des dérivées qui suivent toutes le même schéma :

| Suffixe | Rôle | Exemple |
|---|---|---|
| `LY` | Même période, année précédente | `Nb departs LY` |
| `PT` | Écart vs N-1 (en valeur, ou en points pour les taux) | `Taux de depart PT` |
| `pt Delta` | Texte avec flèche ▲ ▼ ▶ et variation, « Pas de N-1 » si vide | `Nb departs pt Delta` |
| `pt Couleur` | Code couleur selon le sens métier | `Nb departs pt Couleur` |
| `Texte` | Valeur formatée pour les cartes | `Salaire moyen f/h Texte` |

Les codes couleur sont dans le dossier *Technique* : `_Couleur bon` (#2e7d32), `_Couleur mauvais` (#c92828) et `_Couleur neutre` (#6B7280).

> Pour un indicateur « plus c'est bas, mieux c'est » (départs, démissions, licenciements, taux de départ), la logique de couleur est inversée : une baisse est verte (voir `Nb departs pt Couleur`).

---

## 📐 Schéma N-1

Le N-1 compare la période sélectionnée à la même période de l'année précédente. Aucune mesure n'utilise `TODAY()`.

```dax
Nb departs LY =
CALCULATE (
    [Nb departs],
    SAMEPERIODLASTYEAR ( dim_date[Date] )
)
```

```dax
Nb departs PT =
VAR _actuel    = [Nb departs]
VAR _precedent = [Nb departs LY]
RETURN
    IF ( ISBLANK ( _actuel ) || ISBLANK ( _precedent ), BLANK (), _actuel - _precedent )
```

Pour les taux (`Taux de depart`, `Part des départs volontaires`), l'écart `PT` est une différence de taux, affichée en points de pourcentage par `pt Delta`.

---

## 🎯 Population « présente dans la période »

Les mesures de rémunération, de démographie et de performance retiennent les employés **présents au moins un jour** dans la période sélectionnée : embauchés avant la fin de la période et non partis avant son début. Le filtre de date est neutralisé (`REMOVEFILTERS`) et remplacé par ces deux bornes.

```dax
Salaire moyen =
VAR _debut = MIN ( dim_date[Date] )
VAR _fin   = MAX ( dim_date[Date] )
RETURN
    CALCULATE (
        AVERAGE ( fact_employees[Salary] ),
        REMOVEFILTERS ( dim_date ),
        fact_employees[DateofHire] <= _fin,
        fact_employees[DateofTermination] >= _debut
            || ISBLANK ( fact_employees[DateofTermination] )
    )
```

Ce même filtre se retrouve dans `Salaire median`, `Part des femmes`, `Age moyenne`, `Engagement moyen`, `Satisfaction moyenne`, `Absences moyenne`, `Retards moyens (30 j)`, `Projets spéciaux` et `Performance moyenne`.

> Avec un axe `Month name` sans année, les bornes couvrent presque toute la période : la valeur reste plate. Mettre `Year` en axe.

---

## 👥 Effectifs

| Mesure | Formule |
|---|---|
| `Effectif total` | `CALCULATE(DISTINCTCOUNT(fact_employees[EmpID]), REMOVEFILTERS(dim_date))` |
| `Employés actifs` | `Effectif total` pour `Termd = 0` |
| `Effectif debut de periode` | Employés embauchés et pas encore partis la veille du premier jour de la période |
| `Effectif fin de periode` | Employés présents le dernier jour de la période |
| `Effectif moyen` | Moyenne des effectifs de fin de mois sur la période (`AVERAGEX` sur année et mois) |
| `Ancienneté moyenne` | `AVERAGE(fact_employees[Anciennete])`, sans filtre de date |

`Ancienneté` est mesurée à la date de départ pour les partis et à la date de référence (`DateRef`, 28/02/2019) pour les actifs.

## 🚪 Attrition

| Mesure | Formule |
|---|---|
| `Nb departs` | `CALCULATE(DISTINCTCOUNT(EmpID), Termd = 1, USERELATIONSHIP(dim_date[Date], fact_employees[DateofTermination]))` |
| `Taux de depart` | `DIVIDE([Nb departs], [Effectif moyen] * années)` avec `années` = nombre de mois distincts de la période / 12 |
| `Demissions` | `Nb departs` pour `EmploymentStatus = "Voluntarily Terminated"` |
| `Licenciements` | `Nb departs` pour `EmploymentStatus = "Terminated for Cause"` |
| `Part des départs volontaires` | `DIVIDE([Demissions], [Nb departs])` |
| `Anciennete moyenne au depart` | Ancienneté moyenne des employés partis dans la période (date de départ) |

Le taux de départ est **annualisé** : sur une année civile, c'est les départs divisés par l'effectif moyen de l'année.

## 🎯 Recrutement

| Mesure | Formule |
|---|---|
| `Embauches` | `DISTINCTCOUNT(fact_employees[EmpID])` (par date d'embauche, relation active) |
| `Canaux recrutement` | `DISTINCTCOUNT(fact_employees[RercrutmentSourceID])` |
| `Part canal principal` | Embauches du premier canal / `Embauches` (`MAXX` sur les canaux) |
| `Part forum diversité` | Embauches via « Diversity Job Fair » / `Embauches` |
| `Part de cooptation` | Embauches via « Employee Referral » / `Embauches` |

## 💶 Rémunération

| Mesure | Formule |
|---|---|
| `Salaire moyen` | `AVERAGE(fact_employees[Salary])` sur la population présente dans la période |
| `Salaire median` | `MEDIAN(fact_employees[Salary])`, même population |
| `Salaire moyen femmes` | `Salaire moyen` pour `dim_employee[Sex] = "F"` |
| `Salaire moyen hommes` | `Salaire moyen` pour `dim_employee[Sex] = "M"` |
| `Salaire moyen toutes années` | `AVERAGE(fact_employees[Salary])` sans filtre de date |
| `Salaire moyen f/h Texte` | Texte « femmes / hommes » formaté en euros pour les cartes |

`Salaire moyen toutes années` est volontairement plat sur un axe de temps : c'est une valeur de référence.

## 🧑‍🤝‍🧑 Démographie

| Mesure | Formule |
|---|---|
| `Part des femmes` | Femmes distinctes / employés distincts, sur la population présente dans la période |
| `Age moyenne` | `AVERAGEX(fact_employees, RELATED(dim_employee[Age]))`, même population |

L'âge est calculé une fois pour toutes à la date de référence (28/02/2019), quelle que soit la période.

## ⭐ Performance

| Mesure | Formule |
|---|---|
| `Engagement moyen` | `AVERAGE(fact_employees[EngagementSurvey])`, enquête de 1 à 5 |
| `Satisfaction moyenne` | `AVERAGE(fact_employees[EmpSatisfaction])`, enquête de 1 à 5 |
| `Absences moyenne` | `AVERAGE(fact_employees[Absences])` |
| `Retards moyens (30 j)` | `AVERAGE(fact_employees[DaysLateLast30])` |
| `Projets spéciaux` | `AVERAGE(fact_employees[SpecialProjectsCount])` |
| `Performance moyenne` | Moyenne d'une note de 1 à 4 : PIP = 1, Needs Improvement = 2, Fully Meets = 3, Exceeds = 4 |

Toutes ces mesures portent sur la population présente dans la période.

`PerformanceScore` est une catégorie : la note de `Performance moyenne` repose sur une échelle choisie, avec le même écart entre chaque niveau.

```dax
Performance moyenne =
VAR _debut = MIN ( dim_date[Date] )
VAR _fin   = MAX ( dim_date[Date] )
RETURN
    CALCULATE (
        AVERAGEX (
            fact_employees,
            SWITCH (
                RELATED ( dim_performance_score[PerformanceScore] ),
                "PIP", 1,
                "Needs Improvement", 2,
                "Fully Meets", 3,
                "Exceeds", 4,
                BLANK ()
            )
        ),
        REMOVEFILTERS ( dim_date ),
        fact_employees[DateofHire] <= _fin,
        fact_employees[DateofTermination] >= _debut
            || ISBLANK ( fact_employees[DateofTermination] )
    )
```

---

## 🗂️ Dossiers d'affichage

```
_Mesures
├── Dashboard
│   ├── 1 Base
│   ├── 2 Comparaison N-1
│   └── 3 Affichage carte
├── Attrition
│   ├── 1 Base
│   ├── 2 Comparaison N-1
│   └── 3 Affichage carte
├── Recrutement
│   └── 1 Base
├── Rémunération
│   ├── 1 base
│   └── 3 Affichage carte
├── Démographie
│   ├── 1 Base
│   └── 3 Affichage carte
├── Performance
│   ├── 1 Base
│   └── 3 Affichage carte
└── Technique              (codes couleur, libellé « toutes annees »)
```