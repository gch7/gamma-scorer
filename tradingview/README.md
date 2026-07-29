# Medie Mobili + Bande di Bollinger + VWAP

Indicatore TradingView (**Pine Script v6**, `overlay = true`) che raccoglie in un solo script
i tre strumenti più usati per leggere trend, volatilità e prezzo medio ponderato.

File: [`GammaCapital_MA_BB_VWAP.pine`](GammaCapital_MA_BB_VWAP.pine)

## Installazione

1. Apri un grafico su TradingView → **Pine Editor** (in basso).
2. Menu **Apri → Nuovo indicatore vuoto**, cancella il contenuto.
3. Incolla tutto il contenuto del file `.pine`.
4. **Salva** (dai un nome), poi **Aggiungi al grafico**.
5. L'ingranaggio ⚙️ sull'indicatore apre tutti i parametri, divisi in cinque gruppi.

## Cosa contiene

### Medie mobili
Sei slot indipendenti, ognuno con tipo, periodo, colore, spessore e timeframe propri.
Di fabbrica sono impostate le cinque più usate nel trading:

| Slot | Default | Uso tipico |
|---|---|---|
| MA 1 | EMA 9 | timing di brevissimo periodo, scalping |
| MA 2 | EMA 21 | supporto dinamico nei trend, pullback intraday/swing |
| MA 3 | SMA 50 | trend intermedio, gamba veloce del Golden/Death Cross |
| MA 4 | SMA 100 | trend di medio periodo |
| MA 5 | SMA 200 | spartiacque di regime, gamba lenta del Golden/Death Cross |
| MA 6 | EMA 20 (spenta) | slot libero |

Tipi disponibili: **SMA, EMA, WMA, HMA, RMA/SMMA, VWMA, DEMA, TEMA, LSMA, ALMA**.
DEMA e TEMA non esistono come funzioni native di Pine e sono implementate a mano
(`2·EMA1 − EMA2` e `3·EMA1 − 3·EMA2 + EMA3`).

Extra: **Ribbon** colorato fra due slot a scelta ed etichette con il nome della media
all'estremità destra del grafico.

**Multi-timeframe:** lasciando vuoto il campo timeframe si usa quello del grafico. Se lo
valorizzi deve essere *superiore* a quello del grafico. La lettura MTF usa
`request.security()` con `lookahead_off` e la tecnica di sfasamento della documentazione
ufficiale: **non ripinge**, al prezzo di aggiornare un gradino dopo (riflette l'ultima barra
HTF chiusa). Con un timeframe inferiore lo script si ferma con un messaggio esplicito.

### Bande di Bollinger
Basis con tipo selezionabile (default SMA 20), moltiplicatore 2.0, seconda coppia di bande
opzionale, offset, riempimento dell'area con colore dinamico su %B.

Il basis è il *centro* delle bande, ma la semi-ampiezza resta sempre `ta.stdev()`, cioè la
dispersione attorno alla media della finestra: è l'unica misura che rappresenta la
**volatilità**. Calcolarla attorno a un basis con ritardo diverso gonfierebbe le bande in
proporzione al lag del basis (effetto massimo con HMA, LSMA, DEMA, TEMA) falsando %B,
BandWidth e Squeeze.

**Squeeze:** BandWidth al minimo delle ultime N barre (default 120; Bollinger usava ~125 barre
daily, cioè 6 mesi). Il rilevamento parte solo dopo aver osservato N barre con BandWidth
valido, così non scatta su ogni grafico appena caricato.

**Segnali di banda:** viene marcata solo la *prima* barra di ogni sequenza fuori banda —
finché il prezzo "cammina sulla banda" è continuazione, non inversione.

### VWAP
Ancoraggio a **Sessione / Settimana / Mese / Trimestre / Anno / Data personalizzata**, fino a
3 bande con moltiplicatori indipendenti (1.0 / 2.0 / 3.0) in modalità **deviazione standard**
o **percentuale**, riempimenti fra bande adiacenti e **MVWAP** opzionale.

Le bande usano la deviazione standard *pesata per il volume* restituita da `ta.vwap()`, la
stessa della VWAP nativa di TradingView. Sui simboli senza volume compare un'etichetta di
avviso **non bloccante**: medie e Bollinger continuano a funzionare.

Su timeframe 1D e superiori la VWAP è nascosta di default: l'ancoraggio giornaliero sarebbe
vero su ogni barra e la linea coinciderebbe con la sorgente.

### Tabella e alert
Tabella informativa opzionale (posizione, colori e dimensione configurabili) con valore e
posizione del prezzo per ogni media attiva, bande, %B, BandWidth e VWAP, più una sintesi di
tendenza.

Sette alert pronti: **Golden Cross**, **Death Cross**, prezzo sopra/sotto la **VWAP**,
chiusura sopra/sotto la **banda di Bollinger**, **inizio Squeeze**. Disponibili sia come
`alertcondition()` (finestra "Crea allarme") sia come `alert()` con messaggio dinamico,
impostati su `freq_once_per_bar_close` per evitare notifiche su barre ancora in formazione.

## Note tecniche

- Consumo dichiarato: ~46 conteggi di plot sui 64 massimi consentiti da Pine.
- Nessun uso di `barmerge.lookahead_on`, `timenow` o `chart.left_visible_bar_time`: nessuna
  fonte di *future leak*. La sezione 15 del file dettaglia il comportamento di repainting di
  ogni componente.
- Con tutti i campi timeframe vuoti (default) lo script non effettua **nessuna** chiamata
  `request.security()`.
