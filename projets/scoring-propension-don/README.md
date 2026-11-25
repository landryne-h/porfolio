# Score de propension au don : qui va donner cette année ?

> Projet de scoring prédictif réalisé en Python
> Objectif : aider une grande association à cibler ses sollicitations en prédisant, pour chacun de ses 73 124 contacts, la probabilité de faire au moins un don sur l'année.

![Python](https://img.shields.io/badge/Python-3-blue) ![pandas](https://img.shields.io/badge/pandas-preparation-150458) ![scikit--learn](https://img.shields.io/badge/scikit--learn-modelisation-F7931E) ![statsmodels](https://img.shields.io/badge/statsmodels-regression%20logistique-4051B5) ![Claude](https://img.shields.io/badge/Claude-assistance%20IA-D97757)

## En bref

| | |
|---|---|
| **Question métier** | Quelle est la probabilité qu'un contact fasse au moins un don l'année N ? |
| **Données** | 73 124 contacts anonymisés, 42 colonnes (12 ans d'historique de dons, canaux, moyens de paiement, profil) |
| **Méthode** | Discrétisation optimale (WoE / IV) puis régression logistique |
| **Outils** | Python (pandas, optbinning, statsmodels, scikit-learn), Claude en assistance IA |
| **Performance (test)** | **AUC 0,925**, Gini 0,851, KS 0,73 |
| **Impact** | Contacter 30 % de la base suffit pour toucher **84 % des donateurs** |

## Le contexte

Une association sollicite sa base de contacts plusieurs fois par an (courriers, appels, e-mails). Chaque sollicitation a un coût, et seulement **25,6 %** des contacts donnent dans l'année : contacter tout le monde au hasard, c'est 3 envois sur 4 sans retour.

Le score répond à une question précise : pour un **contact**, la probabilité de faire **au moins un don** sur l'**année N**, prédite uniquement à partir des années N-11 à N-1.

## La démarche

Le [notebook](scoring_propension_don.ipynb) déroule les 6 étapes, avec le code, les tableaux et les graphiques :

0. **Qualité des données** : contrôles, capping P1/P99 des montants, « âge non renseigné » (63 % des contacts) gardé comme modalité, exclusion des variables de l'année N pour éviter toute fuite d'information.
1. **Discrétisation** de chaque variable en tranches avec `optbinning`, apprise sur l'échantillon d'apprentissage seulement.
2. **Sélection des variables** par Information Value, khi-carré et V de Cramér.
3. **Échantillons** apprentissage / test 70 / 30 stratifiés.
4. **Corrélations** : suppression des variables redondantes (corrélation > 0,7).
5. **Régression logistique** sur les WoE, en ne gardant que des coefficients positifs et significatifs : 11 variables retenues.
6. **Évaluation** sur l'échantillon de test : AUC, Gini, KS, matrice de confusion, analyse par déciles.

## Les résultats

**Un modèle stable** : AUC de 0,921 en apprentissage et 0,925 en test, le modèle n'a pas appris « par cœur ». Au seuil de 0,5, il classe correctement 89 % des contacts.

**Un ciblage qui change la rentabilité des campagnes** :

| Déciles de score | Taux de don réel | Donateurs captés (cumul) |
|---|---|---|
| 1 | 94 % | 37 % |
| 1 et 2 | 78 % à 94 % | 67 % |
| 1 à 3 | 42 % à 94 % | 84 % |
| 7 à 10 | moins de 4 % | seulement 3,5 % des donateurs |

**Les variables qui comptent le plus** : l'abonnement à la newsletter générale (52 % de donateurs, contre 5 % sans abonnement), l'origine du contact, le fait que l'âge soit renseigné (9 % de donateurs quand il ne l'est pas), la récence du dernier don et le montant cumulé.

## Recommandations métier

| Segment | Déciles | Action proposée |
|---|---|---|
| Ciblage prioritaire | 1 et 2 | Sollicitation en premier, sur les canaux les plus rentables |
| Ciblage secondaire | 3 | Sollicitation ensuite, avec un message adapté |
| Pression réduite | 4 à 6 | Canaux peu coûteux (e-mail), pas d'envois répétés |
| Ne pas solliciter | 7 à 10 | Retrait des campagnes coûteuses |

Pistes complémentaires : encourager l'inscription à la newsletter générale et mieux collecter l'âge des contacts.

## Limites identifiées

- **Contacts sans historique** : 7 % des contacts n'ont aucun don avant N et ont tous donné en N. Ils ont probablement été ajoutés à la base *parce qu'*ils ont donné, ce qui flatte le modèle. Sur les seuls contacts avec historique, l'AUC reste de 0,91 après ré-entraînement.
- **Risque de fuite d'information** : l'abonnement et l'origine ont une IV très élevée (> 1,7). Il faudrait vérifier avec l'association que ces informations ne sont pas enregistrées *après* le don.
- **Suites possibles** : validation temporelle (entraîner sur N, tester sur N+1), comparaison avec un arbre de décision ou un gradient boosting, grille de points avec `scorecardpy`.

## Fichiers

| Fichier | Contenu |
|---|---|
| [`scoring_propension_don.ipynb`](scoring_propension_don.ipynb) | Le notebook complet (étapes 0 à 6) |
| [`donnees_score.csv`](donnees_score.csv) | Les données anonymisées (séparateur `;`, encodage latin-1) |
| [`requirements.txt`](requirements.txt) | Les bibliothèques Python : pandas, numpy, optbinning, scipy, statsmodels, scikit-learn, matplotlib |

Pour relancer l'analyse : `pip install -r requirements.txt`, puis exécuter le notebook dans ce dossier.

## Compétences mobilisées

- **Data science** : feature engineering, discrétisation supervisée, sélection de variables, régression logistique, évaluation de modèle (AUC, Gini, KS, lift)
- **Statistiques** : WoE / IV, khi-carré, V de Cramér, analyse de corrélations
- **Marketing et CRM** : segmentation, ciblage de campagnes, optimisation de la pression marketing
- **Communication** : traduction de résultats techniques en recommandations actionnables pour un public métier
