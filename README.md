# ah249ki

# Detekcia podvodov v transakčných dátach - PaySim experimenty

## Základné informácie

Tento repozitár/notebook obsahuje experimentálnu časť bakalárskej práce zameranej na detekciu podvodných transakcií v transakčných dátach. Primárne sa pracuje s datasetom **PaySim**, ktorý simuluje mobilné finančné transakcie počas 30 dní. Cieľom je porovnať vybrané metódy strojového učenia pri klasifikácii transakcií na legitímne a podvodné.

## Cieľ práce

Hlavným cieľom je navrhnúť a experimentálne overiť modely schopné detegovať podvodné transakcie v silne nevyváženom datasete. Práca sa zameriava na:

- analýzu transakčných dát,
- tvorbu doménovo odvodených príznakov,
- porovnanie modelov strojového učenia,
- vyhodnotenie pomocou metrík vhodných pre nevyvážené dáta,
- interpretáciu správania modelov.

## Dataset

Použitý dataset: **PaySim**

Základné atribúty datasetu:

- `step` - časový krok simulácie, kde 1 krok predstavuje 1 hodinu,
- `type` - typ transakcie (`CASH-IN`, `CASH-OUT`, `DEBIT`, `PAYMENT`, `TRANSFER`),
- `amount` - suma transakcie,
- `oldbalanceOrg`, `newbalanceOrig` - zostatok odosielateľa pred a po transakcii,
- `oldbalanceDest`, `newbalanceDest` - zostatok príjemcu pred a po transakcii,
- `isFraud` - cieľová premenná, kde 1 označuje podvodnú transakciu,
- `isFlaggedFraud` - pravidlový flag, ktorý bol v hlavných experimentoch vynechaný.

Identifikátory `nameOrig` a `nameDest` neboli použité priamo ako vstupné premenné, pretože ide o vysoko kardinalitné textové polia. Z `nameDest` bol odvodený iba indikátor `isMerchantDest`.

## Metodika

Experimenty boli štruktúrované podľa logiky CRISP-DM:

1. **Business Understanding** - definícia problému detekcie podvodných transakcií.
2. **Data Understanding** - analýza štruktúry PaySim datasetu a nerovnováhy tried.
3. **Data Preparation** - čistenie dát, odstránenie nevhodných polí a tvorba nových príznakov.
4. **Modeling** - tréning modelov Decision Tree, Random Forest, XGBoost, LightGBM a doplnkového hybridu Autoencoder -> LightGBM.
5. **Evaluation** - vyhodnotenie pomocou AUPRC, ROC-AUC, Precision, Recall, F1 a confusion matrix.
6. **Deployment/Interpretation** - interpretácia výsledkov a príprava grafov použiteľných v bakalárskej práci.

## Feature engineering

V experimentoch boli vytvorené najmä tieto odvodené príznaky:

```python
errorBalanceOrig = oldbalanceOrg - amount - newbalanceOrig
errorBalanceDest = oldbalanceDest + amount - newbalanceDest
```

Ďalej boli vytvorené binárne indikátory:

- `isMerchantDest`,
- `zeroDestBalancesNonZeroAmount`,
- `zeroOrigBalancesNonZeroAmount`.

Tieto príznaky sú deterministické funkcie jednej transakcie, preto ich výpočet nepredstavuje data leakage. Operácie závislé od distribúcie dát, ako škálovanie, resampling alebo ladenie prahu, sa vykonávali iba na trénovacej alebo validačnej množine.

## Modely

V notebooku sú testované najmä tieto modely:

- Decision Tree,
- Random Forest,
- XGBoost,
- LightGBM,
- Logistic Regression pre zjednodušený baseline,
- Autoencoder -> LightGBM ako doplnkový hybridný prístup.

## Vyhodnocovacie metriky

Keďže ide o silne nevyváženú klasifikačnú úlohu, hlavnou metrikou je **AUPRC**. Doplnkovo sa používa:

- ROC-AUC,
- Precision,
- Recall,
- F1-score,
- confusion matrix,
- počet FP/FN pri zvolenom prahu.

Rozhodovací prah sa odporúča ladiť na validačnej množine, nie priamo na testovacej množine.

## Spustenie v Google Colab

1. Nahraj dataset `Fraud.csv` do Google Drive.
2. Otvor notebook `Experiment_anton.ipynb` v Google Colab.
3. Pripoj Google Drive.
4. Skontroluj cestu k datasetu v premennej `PATH`.
5. Spúšťaj bunky postupne od načítania dát po vyhodnotenie modelov.

## Výstupy

Výsledkom experimentov sú:

- tabuľky metrík modelov,
- PR/ROC grafy,
- porovnanie prístupov class_weight vs. SMOTENC,
- porovnanie full-feature modelov s hybridom AE->LGBM,
- interpretácia vhodná do kapitoly experimentálnej časti bakalárskej práce.

## Poznámka k dátovej bezpečnosti

Dataset PaySim je syntetický, preto neobsahuje reálne osobné údaje klientov. Napriek tomu sa metodika práce snaží rešpektovať princípy anti-fraud modelovania, najmä časové delenie dát, oddelenie validačnej a testovacej množiny a prevenciu data leakage.
