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
