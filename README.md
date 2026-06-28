# Marketplace Safety – Machine Learning

Det här projektet är en individuell version av gruppuppgiften i Machine Learning. Uppgiften handlar om att bygga ett beslutsstöd för en marketplace-app, ungefär som Blocket, Tradera eller Facebook Marketplace.

Målet är att prioritera misstänkta händelser, till exempel bluffannonser, spam eller försök att flytta konversationer utanför plattformen.

Modellen ska inte användas för att automatiskt stänga av användare eller ta bort annonser. Den ska användas som stöd för Trust & Safety-teamet så att de kan granska de mest misstänkta händelserna först.

## Dataset

Projektet använder två dataset:

```text
data/raw/historical_data.csv
data/raw/new_data.csv
```

`historical_data.csv` innehåller historisk data med target-kolumnen:

```text
is_suspicious
```

Där:

* `1` betyder misstänkt händelse
* `0` betyder ej misstänkt händelse

`new_data.csv` innehåller ny data utan target. Den används endast för prediktion och prioritering efter att modellen är vald och låst.

## Kravkort

Eftersom jag gör uppgiften individuellt och inte fick ett gruppspecifikt kravkort valde jag ett rimligt kravkort för att kunna genomföra projektet konsekvent.

**Stakeholder:** Trust & Safety-teamet
**Prioritet:** Minimera false negatives, alltså att undvika att missa misstänkta händelser.

Motiveringen är att det i det här caset är värre att missa en faktisk bluff eller misstänkt aktivitet än att råka skicka några extra händelser till manuell granskning. Modellen används därför som ett prioriteringsstöd, inte som ett automatiskt beslutssystem.

## Strategi

Jag tränade modellerna på `historical_data.csv` och gjorde en egen stratifierad train/test-split. Preprocessing gjordes i en scikit-learn-pipeline för att undvika data leakage.

Jag jämförde en baseline-modell med flera machine learning-modeller:

* DummyClassifier baseline
* Logistic Regression
* Decision Tree
* Random Forest
* Gradient Boosting

Eftersom target-klassen är obalanserad använde jag inte bara accuracy. Jag tittade även på precision, recall, F1, ROC-AUC och PR-AUC.

Final modell blev Logistic Regression eftersom den passade kravkortet bra: den är relativt lätt att förklara, fungerar bra i en pipeline och kan kombineras med threshold-val för att prioritera hög recall.

Efter tuning valde jag en threshold utifrån målet att hitta så många misstänkta händelser som möjligt, samtidigt som mängden false positives fortfarande ska vara rimlig för manuell granskning.

Till sist kördes den låsta pipelinen på `new_data.csv` för att skapa risk-score och en prioriteringslista.

## Projektstruktur

```text
marketplace-safety-ml/
├── data/
│   └── raw/
│       ├── historical_data.csv
│       └── new_data.csv
├── models/
│   └── final_model.joblib
├── notebooks/
│   └── report.ipynb
├── reports/
│   ├── figures/
│   ├── final_model_test_metrics.csv
│   ├── model_comparison.csv
│   ├── new_data_predictions.csv
│   ├── new_data_summary.csv
│   ├── new_data_top_50_priority_list.csv
│   ├── selected_threshold_metrics.csv
│   ├── threshold_results.csv
│   └── tuning_results.csv
├── requirements.txt
└── README.md
```

## Så kör man projektet

### 1. Klona repot

```bash
git clone https://github.com/SoroushGReza/marketplace-safety-ml
cd marketplace-safety-ml
```

### 2. Skapa virtuell miljö

På Windows:

```bash
py -m venv .venv
.venv\Scripts\activate
```

### 3. Installera dependencies

```bash
pip install -r requirements.txt
```

### 4. Starta Jupyter / öppna notebooken

Öppna projektet i Visual Studio Code och öppna:

```text
notebooks/report.ipynb
```

Välj Python-miljön `.venv` som kernel.

Kör sedan:

```text
Restart Kernel and Run All Cells
```

Notebooken ska kunna köras från början till slut utan fel.

## Viktiga filer

| Fil                                         | Beskrivning                                            |
| ------------------------------------------- | ------------------------------------------------------ |
| `notebooks/report.ipynb`                    | Huvudrapport med kod, figurer, resultat och reflektion |
| `models/final_model.joblib`                 | Sparad pipeline med preprocessing och final modell     |
| `reports/model_comparison.csv`              | Resultat från modelljämförelse                         |
| `reports/tuning_results.csv`                | Resultat från hyperparameter-tuning                    |
| `reports/selected_threshold_metrics.csv`    | Resultat för vald threshold                            |
| `reports/new_data_predictions.csv`          | Prediktioner på ny data                                |
| `reports/new_data_top_50_priority_list.csv` | Prioriteringslista med högst risk                      |
| `reports/new_data_summary.csv`              | Sammanfattning av resultat på ny data                  |

## Ansvarsfördelning

Eftersom uppgiften gjordes individuellt ansvarade jag själv för alla delar.

| Område                           | Ansvarig |
| -------------------------------- | -------- |
| Data & EDA                       | Soroush  |
| Pipeline & preprocessing         | Soroush  |
| Modelljämförelse                 | Soroush  |
| Optimering/tuning                | Soroush  |
| Threshold/prioritering           | Soroush  |
| Rapport, risker och presentation | Soroush  |

## Begränsningar

Modellen är tränad på historisk data. Om beteenden på plattformen förändras kan modellen prestera sämre över tid. Därför bör modellen följas upp regelbundet.

`new_data.csv` saknar facit, så det går inte att direkt veta hur många av de flaggade händelserna som faktiskt är misstänkta. Resultatet bör därför ses som en prioriteringslista och följas upp efter manuell granskning.

Modellen bör inte användas för automatiska straff eller borttagningar. Den bör användas som ett beslutsstöd där människor fortfarande tar det slutliga beslutet.

## Nästa steg

Om lösningen skulle användas i produktion skulle nästa steg vara att:

* följa upp manuell granskning och samla nya labels
* mäta precision och recall löpande
* bevaka data drift
* justera threshold efter teamets kapacitet
* träna om modellen regelbundet
* komplettera modellen med fler signaler, till exempel textanalys
