- Usa monte carlo per generare intervalli di valori:
    Prendi un soggetto e misuri la vita. Lo strumento/operatore ha un errore: diciamo che la "vera" vita potrebbe stare a 90 cm ± 1 cm. La tua predict_bf dà un solo numero. Cosa fa Monte Carlo, riga per riga:


    vita_misurata = 90.0
    sigma = 1.0          # incertezza nota della misura

    risultati = []
    for _ in range(10000):                      # 1) ripeti 10.000 volte
        vita_sim = np.random.normal(90.0, 1.0)  # 2) "tira un dado": vita plausibile
        bf = predict_bf({..., "waist": vita_sim})# 3) calcola col valore perturbato
        risultati.append(bf)                     # 4) salva il risultato

    media   = np.mean(risultati)                 # ~23.4%
    intervallo = np.percentile(risultati, [2.5, 97.5])  # es. 

- Sostituisci chi quadro con ANOVA, il chi quadro verrà forse usato per qualcos altro, es. tabelle di contingenza oppure utilizzo sia di uno che dell'altro.