# Network Intrusion Traffic Analysis Lab

Questo README contiene il riassunto di tutti i passaggi operativi e delle domande di analisi richieste dal laboratorio di **Network Intrusion Traffic Analysis**, basati sui file `network_intrusion_traffic_analysis_lab.pdf` e `SUMMARY.pdf`. 

Puoi utilizzare questo file come to-do list per tenere traccia dei tuoi progressi.

## Informazioni sul Dataset (`SUMMARY.pdf`)
- **Records**: 133,000
- **File**: `data.csv` (47 MB)
- **Sorgenti**: CICIDS2017 + UNSW-NB15 + NSL-KDD
- **Target**: Classificazione binaria (BENIGN vs Attack) o multiclasse (16 classi).

---

## Part 1 -- Dataset Exploration

- [x] **1. Caricamento e analisi preliminare**
  - [x] Carica il dataset `data.csv` (es. tramite pandas).
  - [x] Documenta il numero di righe e colonne.
  - [x] Documenta i tipi di dato di ciascuna colonna.
  - [x] Documenta il numero di valori mancanti per ogni colonna.
  - [x] Analizza la distribuzione del campo `source` (quanti flussi provengono dalle 3 diverse fonti).
  - [x] Analizza la distribuzione del campo `label`: conta i flussi BENIGN vs ogni classe di attacco.
  - [x] Produci un grafico a barre orizzontali (horizontal bar chart) ordinato per frequenza delle label.
  - [x] *Domanda*: Quante classi di attacco distinte sono presenti? Il dataset è bilanciato?

- [x] **2. Profilazione dei campi numerici**
  - [x] Analizza `duration`, `src_bytes`, `dst_bytes`, `packet_count` confrontandoli tra le varie sorgenti.
  - [x] Calcola le statistiche descrittive (media, mediana, min, max, deviazione standard) per ciascun campo numerico.
  - [x] Produci degli istogrammi su scala logaritmica (es. trasformazione `log1p` o `plt.yscale("log")`) per ciascun campo.
  - [x] *Domanda*: Commenta brevemente la forma "heavy-tailed" delle distribuzioni.
  - [x] Produci dei boxplot affiancati (un pannello per feature, tre box per pannello, uno per sorgente) per le quattro feature numeriche.
  - [x] *Domande*: Le tre sorgenti coprono gli stessi range numerici? Ci sono differenze sistematiche (es. durate in NSL-KDD vs CICIDS)? Cosa implica questo per un modello addestrato su una sorgente e testato su un'altra?

## Part 2 -- Protocol, Ports, and Source/Label Structure

- [x] **3. Analisi dei campi `protocol` e `dst_port`**
  - [x] Produci un grafico a barre della distribuzione globale dei protocolli (`tcp`, `udp`, `icmp`, `other`).
  - [x] Produci un grafico a barre (stacked o grouped) della distribuzione delle label condizionata al protocollo (proporzione di BENIGN vs classi di attacco per ogni protocollo).
  - [x] *Domanda*: Quali protocolli trasportano quali attacchi (es. ICMP è dominato da PROBE? UDP da DOS/DDOS)?
  - [x] Per `dst_port` (dove presente), calcola le top 20 porte di destinazione per frequenza e confrontale con le well-known ports (22, 23, 80, 443, 445, 3389, 8080).

- [x] **4. Cross-tabulation e Eterogeneità**
  - [x] Produci una cross-tabulation `source` x `label` (righe = source, colonne = label) e visualizzala come heatmap.
  - [x] *Domande*: Quali classi di attacco mancano in ciascuna sorgente? 
  - [x] *Domanda*: Discuti le conseguenze (label-set heterogeneity) dell'addestrare un classificatore multiclasse su flussi che non possono fisicamente contenere certe classi in alcune sorgenti.

## Part 3 -- Feature Engineering & Class Imbalance

- [x] **5. Creazione di nuove feature (Feature Engineering)**
  - [x] Crea `bytes_ratio = src_bytes / (dst_bytes + 1)`
  - [x] Crea `packet_rate = packet_count / max(duration, 0.001)`
  - [x] Crea `is_well_known_port = (dst_port < 1024)` (valore booleano)
  - [x] Crea `log_src_bytes = log1p(src_bytes)` e `log_dst_bytes = log1p(dst_bytes)`
  - [x] Crea `log_duration = log1p(duration)`
  - [x] Produci un grafico della distribuzione per label per **almeno due** delle feature ingegnerizzate (boxplot o violin plot, scala logaritmica dove appropriato).
  - [x] *Domanda*: Commenta quali feature sembrano discriminatorie (es. `packet_rate` separa DOS da BENIGN? `bytes_ratio` identifica gli scan?).

- [x] **6. Gestione dello sbilanciamento delle classi**
  - [x] Calcola la proporzione di ogni classe.
  - [x] Identifica la classe maggioritaria e le classi più rare.
  - [x] *Domanda*: Qual è l'imbalance ratio tra la classe più e meno frequente?
  - [x] *Domanda*: Spiega in 2-3 frasi perché usare `class_weight='balanced'` è un default ragionevole in questo caso.

- [x] **7. Regole di rilevamento basate su euristiche**
  - [x] Scrivi 2 semplici regole di rilevamento basate su euristiche classiche IDS (es. basate su `packet_rate`, `bytes_ratio`, `dst_port`, `protocol`).
  - [x] Per ogni regola, calcola l'accuracy come classificatore binario sul dataset completo.
  - [x] *Domande*: Le semplici soglie sono sufficienti? Dove falliscono?

## Part 4 -- Modeling

- [x] **8. Binary Intrusion Detection (BENIGN vs. Attack)**
  - [x] Raggruppa tutte le label non-BENIGN in un'unica classe "attack".
  - [x] Seleziona le feature numeriche comuni (`duration`, `src_bytes`, `dst_bytes`, `packet_count`) e le feature ingegnerizzate al punto 5.
  - [x] Esegui l'encoding del protocollo (one-hot o ordinal).
  - [x] Fai lo split dei dati: 70% train / 15% validation / 15% test, in modo stratificato rispetto alla `label`, usando `random_state=42`.
  - [x] Addestra un modello **Random Forest** (`n_estimators=200`, `class_weight='balanced'`).
  - [x] Addestra un modello **Gradient Boosting** (`HistGradientBoostingClassifier`).
  - [x] Riporta sul test set le metriche: accuracy, precision, recall, F1, ROC-AUC e la matrice di confusione.
  - [x] Produci un grafico della "feature importance" per la Random Forest.
  - [x] *Domanda*: Quali feature dominano (quelle raw o quelle ingegnerizzate)?

- [x] **9. Multi-class Attack Classification**
  - [x] Estendi il modello all'intera tassonomia: decidi se mantenere tutte e 16 le classi o collassare le meno popolate in "OTHER" (mantenendo le top 8). Motiva la scelta.
  - [x] Addestra un modello **Random Forest** (`class_weight='balanced'`) sulle stesse feature dell'esercizio 8.
  - [x] Produci una tabella con: macro-F1, weighted-F1 e precision/recall per ciascuna classe.
  - [x] Produci una matrice di confusione normalizzata per riga.
  - [x] *Domande*: Quali classi vengono confuse tra loro? Commenta le confusioni sistematiche (es. DOS vs DDOS). Il modello è migliore nel rilevare le classi frequenti o quelle rare?

## Part 5 -- Personal ML Project

- [x] **1. Define the task**
  - [x] Formula una domanda chiara a cui il dataset può rispondere (supervised, unsupervised o semi-supervised).
  - [x] Giustifica perché il problema è significativo e non un semplice esercizio.
- [x] **2. Pick the model family**
  - [x] Scegli un modello appropriato per i dati e il task.
  - [x] Spiega in 3-5 frasi perché questa famiglia di modelli è adatta.
- [x] **3. Train and evaluate end-to-end**
  - [x] Costruisci una pipeline pulita (splits, pre-processing, tuning, metrics).
  - [x] Includi almeno un modello di baseline (comparison baseline).
- [x] **4. Describe two or three concrete use cases**
  - [x] Identifica uno stakeholder.
  - [x] Specifica la decisione supportata dal modello.
  - [x] Analizza il costo dei falsi positivi vs falsi negativi.
- [x] **5. Limitations and ethics**
  - [x] Elenca tre limitazioni oneste del tuo approccio.
  - [x] Descrivi una considerazione etica/legale relativa al progetto.
