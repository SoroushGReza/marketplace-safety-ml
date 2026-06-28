# Presentation – Marketplace Safety ML

## Slide 1 – Företaget, problemet och kravkortet

Marketplace-plattformen har annonser och meddelanden där en mindre andel kan vara bluffannonser, spam eller misstänkt beteende.

Trust & Safety-teamet hinner inte granska allt manuellt, så målet är att prioritera vilka händelser som bör granskas först.

**Kravkort:**  
Stakeholder: Trust & Safety-teamet  
Prioritet: Minimera false negatives.

Modellen ska användas som beslutsstöd, inte för automatiska avstängningar.

---

## Slide 2 – Data-insikter och risker

`historical_data.csv` innehåller 12 000 historiska händelser.

Ungefär 10% av händelserna är misstänkta, vilket betyder att datan är obalanserad.

Det finns missing values i bland annat:

- `region`
- `price`
- `time_to_first_response_min`

Accuracy är därför inte tillräckligt. En modell kan få hög accuracy genom att nästan alltid gissa “ej misstänkt”.

---

## Slide 3 – Pipeline och utvärdering

Jag gjorde en egen train/test-split:

- 80% train
- 20% test
- stratifierad split

Preprocessing gjordes i en scikit-learn-pipeline för att undvika data leakage.

Numeriska features:

- median-imputation
- standardisering

Kategoriska features:

- missing values ersattes med `"missing"`
- one-hot encoding
- `handle_unknown="ignore"`

---

## Slide 4 – Modelljämförelse

| Modell | Precision | Recall | F1 | PR-AUC |
|---|---:|---:|---:|---:|
| Baseline | 0.000 | 0.000 | 0.000 | 0.102 |
| Logistic Regression | 0.192 | 0.649 | 0.297 | 0.281 |
| Decision Tree | 0.183 | 0.638 | 0.283 | 0.216 |
| Random Forest | 0.213 | 0.540 | 0.306 | 0.258 |
| Gradient Boosting | 0.478 | 0.069 | 0.121 | 0.287 |

Gradient Boosting hade hög precision men väldigt låg recall. Därför passade den sämre för vårt kravkort.

Logistic Regression hade bättre balans och mycket högre recall.

---

## Slide 5 – Vald modell och tuning

Final modell: **Logistic Regression**

Jag valde den eftersom:

- den har rimligt hög recall
- den är enklare att förklara
- den fungerar bra i en pipeline
- threshold kan justeras efter Trust & Safety-teamets behov

Tuning gjordes på:

- `C`
- `class_weight`

Bästa parametrar:

```text
C = 10.0
class_weight = {0: 1, 1: 3}