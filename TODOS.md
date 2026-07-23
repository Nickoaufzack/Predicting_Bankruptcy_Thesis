# TODOs — Bankruptcy Thesis

Laufende Sammelstelle für offene Punkte. Layout-Ideen sind bewusst ein eigener
Backlog (eigene Runde am Ende, kein Fokus während der Schreibphase).

## Content / Writing

- [x] **Dataset-Quelle sauber referenzieren** — erledigt 10.07.2026: Datensatz
  jetzt korrekt an **UCI ML Repository #572** attribuiert (`\parencite{uci_taiwan}`,
  DOI 10.24432/C5004D, CC BY 4.0), das exakt deine 6.819×95 abbildet. WICHTIG:
  Liang et al. (2016) ist NICHT die Datenquelle — deren Paper analysiert ein
  balanciertes 478er-Sub-Sample (239+239, 95 FR + 95 CGI); daher die frühere
  Zahlen-Diskrepanz. Die falsche "Sample-Selection Criteria"-Appendix-Sektion
  wurde entfernt. Wang & Liu (2021) nutzen dieselben Daten (Kaggle-Spiegel) →
  Vergleichbarkeit bleibt. Bib liang2016 & wangliu2021 verifiziert.

- [ ] **TWSE-Bankruptcy-Regeln: Darstellung entscheiden** — Fußnote (Empfehlung,
  Lehrstuhl erlaubt Fußnoten) vs. Appendix. Provisorischer Link:
  `https://twse-regulation.twse.com.tw/EN/law/DOC01_print.aspx?FLCODE=FL007304&FLNO=50-1`
  (Tracking-Parameter `utm_source=chatgpt.com` entfernt — gehört nicht in eine Thesis).
- [ ] **Korrekte TWSE-Regel-URL verifizieren** — Liang verweist auf
  `https://twse-regulation.twse.com.tw/ENG/error.aspx?no=9002` (wirkt kaputt/alt).
  Klären, welche TWSE-Regel die Delisting-/Bankruptcy-Kriterien tatsächlich
  definiert, und eine stabile offizielle URL fixieren.

- [ ] **Abstract in die Introduction falten** — Abstract ist raus (Gabriel-Vorgabe);
  der Text lebt noch in `chapters/00_Abstract.tex`. Beim Schreiben der Introduction
  (am Ende) die Zusammenfassung + Forschungsfrage + Kurzantwort dort einarbeiten.

- [ ] **Kritische Dataset-Fragen aufgreifen** — aktuell auskommentiert in
  `03_Dataset.tex` (Temporal-/Look-ahead-Bias: aus welchem Jahr stammt der
  Finanz-Snapshot ggü. dem Pleitejahr?; 2008-Finanzkrise; Repräsentativität des
  Datensatzes). Entscheiden, ob/wie das als kritische Brille in Discussion/
  Conclusion landet. UCI dokumentiert die zeitliche Zuordnung nicht → das ist die
  Limitation und stützt die nicht-kausale Linie.

- [ ] **Literature Review — Pham als Sampling-Übersicht** — Pham (2025) hat eine
  sehr gute Übersicht über Over-/Under-/Hybrid-Sampling; als Quelle für den
  Sampling-/Undersampling-Teil im Lit Review (und ggf. Methodology 4.2 Undersampling)
  nutzen.

- [ ] **Feature-Handling-Landkarte (Recherche 12.07.)** — wer greift Korrelation
  VORAB auf (→ Contrast in Methodology-Intro / Lit Review) vs. wer analysiert nur
  (→ FI-Kapitel):
  - *Eingriff – Feature Selection (Subset):* Veganzones 2018 (entfernt hoch
    korrelierte Ratios), Pham 2025 (BPSO-Wrapper), Ansah-Narh 2024 (GA),
    Chaudhary 2023 (Filter/Pearson).
  - *Eingriff – Dimensionality Reduction (Transform):* Chaudhary 2023 (PCA),
    Ko & Lee 2024 (Faktoranalyse + PCA), Sinap 2025 (PCA).
  - *Nur Analyse, kein Eingriff (→ FI):* Wang & Liu 2021 — explizit KEIN Screening
    „to preserve the integrity of the data" (zeigen Pearson-Diagramm, behalten alle
    Features). = Präzedenz für unseren Ansatz.

- [x] **Multikollinearitäts-Zahlen VERIFIZIERT (21.07.2026)** — direkt auf
  `data.csv` (6.819 × 96) nachgerechnet, alle Werte der alten Analyse bestätigt:
  93 stetige Features (ohne die zwei Flags), 4.278 Paare, **mean |r| = 0,0809**,
  **median |r| = 0,0220**, **28 Paare mit |r| ≥ 0,9**, davon betroffen **31 von 93
  Features**, **12 Paare mit |r| ≥ 0,99**, ROA(A/B/C) zwischen **0,9401 und 0,9868**.
  Zusätzlich drei neue, stärkere Befunde:
  - **Buchhalterische Identität empirisch belegt:** `Debt ratio %` + `Net
    worth/Assets` = **exakt 1,000000 für alle 6.819 Firmen**. Keine Korrelation,
    sondern Definitionsgleichung → belegt die *Herkunft* der Redundanz.
  - **Drei Paare sind buchstäblich identische Spalten** (Werte zeilenweise gleich,
    nur anders benannt): `Current Liabilities/Liability` ≡ `Current Liability to
    Liability`; `Current Liabilities/Equity` ≡ `Current Liability to Equity`;
    `Operating Gross Margin` ≡ `Gross Profit to Sales`. → Der Datensatz enthält den
    Extremfall, was den Permutation-Kollaps zwingend statt nur plausibel macht.
  - **`Net Income Flag` ist konstant** (ein Wert über alle 6.819 Firmen) → trägt
    null Information, kann per Konstruktion nie wichtig sein. Guter Sanity-Check
    für die FI-Rankings.
  Entscheidung: die Zahlen kommen ins **FI-Kapitel** (nicht Dataset, nicht Results).

- [ ] **Multikollinearität — empirisch aus eigenen Daten (12.07.)** — Storyline
  wasserdicht: global ist die Korrelation NIEDRIG (mean |r|=0.08, median 0.02) →
  das ist, was Wang & Liu ("no solid correlation") und Ansah-Narh ("no discernible
  multicollinearity") meinen; sie sind global nicht falsch. ABER lokal gibt es
  Redundanz-Nester: 28 Paare mit |r|≥0.9, 12 quasi identisch, und **31 von 93
  Features haben mindestens einen Beinahe-Zwilling (|r|≥0.9)**. Beispiele:
  Debt ratio % ↔ Net worth/Assets (1.00), Net Value Per Share A/B/C (~1.00),
  ROA(A/B/C) (0.94–0.99). → Framing: global harmlos für Prädiktion (drum behalten
  wir alle Features wie Wang & Liu), aber die lokalen Nester zerlegen die
  Feature-Importance (permutation collapse). NICHT "severe multicollinearity"
  behaupten. Optional: Korrelations-Heatmap als Figure fürs Evidenzargument.

- [ ] **Sinap 2025 als SHAP-Vergleichspunkt (FI-Kapitel/Discussion)** — Sinap ist
  das *einzige* andere Paper auf diesem Datensatz, das SHAP macht: SHAP auf einen
  Stacking Classifier, aber auf **SMOTE-balancierten** Daten + **PCA**; Top-Feature
  „Net Income to Total Assets". Kontrast zu uns: wir SHAP auf **echten,
  undersampled, vollen** Features. Gute Gegenüberstellung fürs FI/Discussion.

- [ ] **Undersampling-Kosten → Discussion** — der (auskommentierte) Absatz in §4.2:
  Undersampling verwirft Mehrheitsdaten (Info-Verlust, kleineres Trainingsset).
  Abgemildert: repräsentative statt zufällige Firmen + Evaluation auf unberührtem
  Test-Fold. Gehört in die Discussion/Limitations, hier bewusst ausgelassen.

- [ ] **Test auf voller Verteilung: Vor- UND Nachteil → Discussion** — wir samplen nur
  im Train, getestet wird auf den unberührten $682$ Firmen (96.8/3.2).
  *Vorteil:* die Schätzung entspricht dem Einsatzfall; auf einem rebalancierten
  Test-Fold wäre die Precision künstlich aufgebläht, weil die Bankrott-Rate dort weit
  über der realen läge. Unsere Zahlen sind also unverzerrt gegenüber der Realität.
  *Nachteil:* dieselbe Medaille — bei nur 22 Bankrotten unter 682 Firmen erzeugt auch
  ein gutes Modell viele Fehlalarme, weshalb Precision und damit F2 niedrig
  *aussehen* (0.55 vs. die 98–99 % der SMOTE-Paper, die teils auf synthetischen oder
  balancierten Testdaten messen). Zusätzlich: 22 Positive pro Fold → hohe Streuung,
  was die 30 Fits und die berichteten Standardabweichungen erst nötig macht.
  → Starkes Discussion-Argument: unsere Zahlen sind niedriger, messen aber etwas
  anderes als die Literatur.

- [ ] **Hyperparameter-Befunde als Argumente nutzen (Discussion / Verteidigung)** —
  zwei Beobachtungen aus `config.py` vs. `tab:selected`:
  *(a) `max_features=0.2` gewinnt gegen `sqrt` und `log2`* — also weniger Features
  pro Split als die Standardheuristik vorsieht. Das stützt das Dekorrelations-
  Argument aus §Random Forest: bei 95 teils redundanten Ratios zahlt sich stärkere
  Dekorrelation aus. Guter Brückenschlag Hyperparameter → Multikollinearitäts-
  Storyline.
  *(b) `max_depth=10` liegt am Rand des RF-Suchraums* (3, 5, 10) — klassische
  Nachfrage („liegt dein Optimum am Rand?"). Antwort steht schon im Appendix: die
  Complexity-Kurve sweept bis 20 und zeigt, dass darüber nichts gewonnen wird.
  Nicht zwingend in den Text, aber für die Verteidigung parat haben.

## Aus letztem Meeting mit Gabriel (17.07.2026)

- [ ] **Undersampling-Kritik (Cluster-Varianz-Trade-off) → Discussion** — Gabriels
  zentraler intellektueller Punkt: viele Cluster (660) = jedes Centroid
  *repräsentativer* für die zugeordneten Firmen, ABER jedes Centroid wird aus im
  Schnitt nur ~9 Firmen (5.939/660) geschätzt → **höhere Varianz / instabiler**.
  Bei einem neuen Datensatz aus demselben Data-Generating-Process würden sich die
  660 Centroids stärker verschieben als wenige grobe Cluster. Es ist ein
  **Trade-off (fein/repräsentativ vs. grob/stabil), KEIN inhärenter Vorteil**
  unserer Methode. Die Wahl der vielen Cluster ist **nur durch die Vergleichbarkeit
  mit Wang & Liu (sampling_strategy=0.3) gerechtfertigt**, nicht durch einen eigenen
  Vorteil. Wichtig: die 30-fache CV **behebt das Grundsätzliche nicht** (Gabriel:
  „ändert das Grundsätzliche nicht"). Extremfall zur Illustration: 220 Cluster =
  1:1, dann ist jedes Cluster = 1 Firma = keine Zusammenfassung mehr. → ehrlich in
  Discussion/Limitations aufnehmen (ergänzt den Undersampling-Kosten-Punkt oben).

- [~] **Quellen an ALLE Figures & Tables** — Mechanismus ist geklärt: **`\floatfoot{}`**
  (kommt von `floatrow`, im Template geladen), rendert am Fuß des Floats.
  *Erledigt:* Fig. 1 K-Means (`Source: \textcite{islp2023}`), Fig. 2 `cv_folds`
  (`Source: own`), Dataset-Tabelle (`Source: \textcite{uci_taiwan}`), alle vier
  Appendix-Floats (`Source: own`).
  *Noch offen:* sämtliche Floats in **Results** (Performance-, Selected-,
  Confusion-, GB-SHAP-Tabelle sowie alle SHAP-/Permutation-Figures) und die
  Confusion-Schema-Tabelle in **Performance Metrics** → alle `Source: own`.
  Mechanische Runde, am besten zusammen mit dem Results-Kapitel.

- [ ] **ROC-/Precision-Recall-Kurven: erzeugen oder streichen** — in
  `06_Results.tex` steht `fig:roc` noch als leere Platzhalter-Box, und die
  Bilddateien existieren nicht. Entweder aus dem Code exportieren oder die Figure
  ersatzlos streichen (ROC-AUC steht ohnehin in `tab:performance`, und die Kurven
  sind unter starker Imbalance ohnehin die schmeichelhafte Darstellung, gegen die
  du argumentierst). Entscheidung spätestens beim Results-Kapitel.

- [x] **§4.4 Titel → „Cross-Validation"** — erledigt 21.07.2026. Titel gekürzt, die
  Langform („repeated stratified $k$-fold cross-validation with a nested grid
  search") steht jetzt im zweiten Satz des Einstiegs, die Wort-für-Wort-Sezierung
  bleibt.

- [ ] **Mehr Figures wo möglich** — Gabriel ermutigt ausdrücklich: Figures zählen
  NICHT zu den 20 Seiten, geben Intuition, gern eigene in matplotlib. Optional
  (nur „wenn ewig Zeit"): ClusterCentroids-Figur mit **auf ~60 Firmen
  heruntergesampelten echten Daten** statt synthetisch — schöner, aber nicht nötig.

- [ ] **Seitenzahl final melden** — Gabriel nicht streng: 20 Vorgabe, +10 % ok
  (~22). Aktuell ~19 Textseiten. Plan: am Ende den Text final durchlaufen lassen,
  konkrete Zahl an Gabriel mitgeben; Ziel bleibt 20, leichte Überschreitung
  unproblematisch.

- [x] **AI-Workflow bestätigt (17.07.2026)** — Gabriel hat unseren Ablauf explizit
  abgesegnet: selbst schreiben, dann KI nur für Grammatik/Stil/Kürzen einsetzen
  („I'm worried this is too long / stylistically off, please change a little") ist
  völlig ok; NICHT ok wäre „write me a conclusion from scratch". Code mit KI-Hilfe
  ist ok, weil du ihn verstehst, verantwortest und die wichtigen Teile
  (Hyperparameter etc.) selbst interpretiert hast. Gabriel garantiert: bei normalem
  Vorgehen keine AI-Detection-Probleme. Optional trotzdem: finalen Text durch eine
  Detection laufen lassen (Sicherheit, falls der Lehrstuhl prüft).

## Layout (Backlog — eigene Runde am Ende)

### Offen
- [ ] **Header/Footer inhaltlich füllen** (aus Claude-Code-Stand, 10.07.2026):
  Für professionelleren Look Kopf-/Fußzeile mit Inhalt — z. B. Titel, Chair/
  Lehrstuhl, ggf. laufender Kapitelname. Über `fancyhdr` (bereits aktiv).
- [ ] **Feintuning der leeren Kopfzeile** (aus Claude-Code-Stand, 10.07.2026):
  Leerer Header lässt oben eine weiße Lücke; Seitenzahl sitzt nah am unteren
  Rand. Über `headheight` / `footskip` justierbar.

### Erledigt
- [x] Seitenzahl von oben-rechts nach unten-mittig (`\fancyfoot[C]{\thepage}`) — 10.07.2026
- [x] Schriftgröße A4 auf 12pt — 10.07.2026
