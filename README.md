# Supermercati sostenibili: rating, qualità e utili

Progetto del corso Advanced Analytics (Master in AI e Agenti AI per il Business, Unimarconi e start2impact). Valutazione 492/500, chiuso dopo cinque giri di feedback con il docente.

Tre domande per una catena di supermercati:
1. **Vendite** (1.000 transazioni, 3 città): dove si incassa di più e se il rating del cliente si può prevedere.
2. **Mele** (4.000 frutti): classificare buone e cattive dalle caratteristiche fisiche.
3. **Utili nel tempo** (89 giorni): prevedere il gross income giornaliero.

## Cosa ho trovato

- **Il rating non si può prevedere con questi dati.** Nessuna colonna ha correlazione sopra 0,04. La regressione lineare fa MSE 3,11 contro 3,08 di una baseline che risponde sempre la media, la polinomiale fa peggio (3,45). L'ho dimostrato con i numeri invece di forzare un modello.
- **Mele: la logistic con feature polinomiali di grado 3 arriva a F1 0,88** (AUC test 0,955). Il decision tree overfitta: AUC 1,0 sul train e 0,80 sul test. K-Means non ritrova la qualità: i gruppi sono mele fisicamente simili, non buone o cattive.
- **Gli utili non sono prevedibili, il giorno della settimana sì.** Ho provato regressione lineare, XGBoost, Random Forest con grid search e ARIMA, sempre con split cronologico e `TimeSeriesSplit`. Arrivano tutti vicino alla media (il migliore, Random Forest, fa -2,4% di MAE sulla baseline). ARIMA sceglie da solo (0,0,0) e il test di Ljung-Box (p = 0,78) dice che la serie è compatibile con rumore bianco. Il segnale utile è il sabato: +37% sul lunedì.

## Scelte

- Baseline prima di ogni metrica: un MSE da solo non dice niente.
- Split cronologico sulla serie storica, con le date di train e test stampate: allenare sul futuro dà risultati falsi.
- `OneHotEncoder` di scikit-learn invece di `get_dummies`, così train e test hanno sempre le stesse colonne.
- Il dataset delle mele "già pulito" non lo era: l'ultima riga contiene la firma dell'autore e trasformava `Acidity` in testo.

## Limiti

- I dati di vendita sono il dataset pubblico "Supermarket sales" (Myanmar, 2019), non dati veri di una catena sostenibile.
- Lo scaler nella parte sulle mele è fittato prima dello split, come chiedeva l'ordine del notebook del corso. L'effetto qui è trascurabile, ma andrebbe fittato solo sul train.
- 89 giorni sono pochi per vedere una stagionalità.

## Come eseguirlo

I CSV non sono nel repository. Scaricali e mettili in `data/`:
- `supermarket_sales.csv`: dataset "Supermarket sales" di Kaggle (Myanmar, 2019)
- `apple_quality.csv`: [Apple Quality](https://www.kaggle.com/datasets/nelgiriyewithana/apple-quality) di Kaggle

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
jupyter notebook notebook/DavideAloisi_AdvancedAnalytics.ipynb
```

Il notebook resta leggibile anche senza eseguirlo: tutti gli output e i grafici sono salvati.
