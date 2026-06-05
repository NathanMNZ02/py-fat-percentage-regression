# Muscle Gain Analyzer — Stima della percentuale di massa grassa

Progetto di analisi statistica e machine learning per stimare la **percentuale di massa
grassa (Body Fat %)** di un individuo a partire da dati antropometrici, demografici e
alimentari, utilizzando come gold standard le misurazioni **DEXA** del dataset pubblico
**NHANES**.

L'obiettivo è costruire un modello di regressione che superi le formule cliniche
tradizionali (Deurenberg, U.S. Navy, CUN-BAE) includendo informazioni che queste
trascurano, come la distribuzione del grasso (circonferenze vita/fianchi), l'etnia e i
macronutrienti assunti.

## Contesto

Il solo **BMI** non è un indicatore affidabile dello stato di salute: esistono soggetti
normopeso ma con percentuale di massa grassa elevata (fenomeno *skinny fat* / *obesità
normopeso*). Misurare direttamente la massa grassa richiede strumenti costosi (DEXA,
pesata idrostatica, plicometria), per cui in letteratura si usano formule di stima a
partire da parametri facilmente misurabili. Questo progetto parte da quelle formule e ne
propone una alternativa basata sui dati.

## Dataset

I dati provengono da **NHANES (National Health and Nutrition Examination Survey)** in
formato `.XPT` (SAS):

| File | Contenuto |
|------|-----------|
| `DXX_J.xpt` | Misurazione DEXA della massa grassa totale (`DXDTOPF`) — la variabile target |
| `BMX_J.XPT` | Dati antropometrici (peso, altezza, BMI, circonferenze braccio/vita/fianchi, lunghezze arti) |
| `DEMO_J.XPT` | Dati demografici (genere, età, etnia, stato di gravidanza) |
| `DR1TOT_J.XPT` | Recall alimentare 24h (kcal, proteine, carboidrati, grassi) |

I dataset vengono uniti sul codice soggetto `SEQN` e filtrati per mantenere solo le
misurazioni valide e complete (DEXA valida, antropometria completa, esame fisico
effettuato, esclusione delle donne in gravidanza, recall alimentare affidabile).

> I file `.XPT` attesi dal notebook si trovano in `src/db/`.

## Pipeline di analisi

L'intero studio è contenuto nel notebook [src/fat_percentage_regression.ipynb](src/fat_percentage_regression.ipynb)
ed è strutturato come segue:

1. **Formule classiche** — implementazione di Deurenberg (1991), U.S. Navy e CUN-BAE come baseline di confronto.
2. **Costruzione del dataset** — merge dei file NHANES, filtraggio dei soggetti, gestione dei valori nulli.
3. **Outlier e valori anomali** — cross-check fra misurazioni indipendenti (DXA vs CUN-BAE) e filtraggio per percentili.
4. **Test statistici** — Chi-quadro per verificare l'associazione fra fascia di massa grassa ed etnia/sesso, a giustificazione dell'inclusione di queste variabili.
5. **Selezione delle variabili** — analisi della multicollinearità (matrice di correlazione) e selezione automatica tramite **ElasticNet**.
6. **Modello** — regressione lineare multipla (OLS), poi estensione **polinomiale** con termini di interazione.
7. **Confronto** — valutazione dell'$R^2$ sul test set contro le formule classiche, con miglioramento netto.
8. **Diagnostica** — Predetto vs Reale, Residui vs Predetto e Q-Q plot dei residui per validare le assunzioni di OLS.
9. **Inferenza** — funzione `predict_bf` che incapsula l'intera pipeline e stima la massa grassa di nuovi soggetti.

Il modello addestrato viene serializzato in [src/fat_percentage_model.joblib](src/fat_percentage_model.joblib).

## Note teoriche

La cartella [note/](note/) contiene i notebook didattici sui metodi statistici usati nel progetto:

- Design of Experiments
- Regressione lineare, multipla, polinomiale e pesata
- ANOVA
- Test del Chi-quadro

## Requisiti e avvio

Il progetto usa Python con Jupyter. Le dipendenze principali sono `pandas`, `numpy`,
`scikit-learn`, `statsmodels`, `scipy`, `matplotlib`, `seaborn` e `joblib`.

```bash
# Crea un ambiente virtuale (consigliato)
python3 -m venv .venv
source .venv/bin/activate

# Installa le dipendenze
pip install -r requirements.txt

# Avvia Jupyter e apri il notebook
jupyter notebook src/fat_percentage_regression.ipynb
```

## Struttura del progetto

```
.
├── db/
│   └── health_fitness_dataset.csv       # dataset fitness ausiliario
├── note/                                # notebook teorici sui metodi statistici
├── src/
│   ├── db/                              # file NHANES (.XPT)
│   ├── fat_percentage_regression.ipynb  # analisi e modello principale
│   └── fat_percentage_model.joblib      # modello serializzato
├── requirements.txt
└── README.md
```
