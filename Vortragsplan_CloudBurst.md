# Vortragsplan CloudBurst

Oct 6, 2026 · @Max

## Fahrplan

Fünf Phasen führen zum Vortrag und zur Ausarbeitung. Der Plan rechnet mit dem frühesten möglichen Termin, Dienstag, 24. November. Liegt dein Termin später, verschieben sich die Phasen 2 bis 4 nach hinten.

&#91;embedded content: Fahrplan · 5 Phasen, 4 Meilensteine · Annahme: Vortrag am 24.11.\]

Phase 1 läuft ab heute. Fest ist nur der Vortragstermin, die übrigen Daten sind Vorschläge.

### Phase 1 · Paper verstehen und Struktur bauen · bis Oct 18, 2026

- [ ] Paper in drei Durchgängen lesen (siehe „Paper verstehen“)
- [ ] Hintergrund nacharbeiten: MapReduce, Seed-and-Extend, Landau-Vishkin in Grundzügen
- [ ] Mini-Beispiel aus „CloudBurst Schritt für Schritt“ von Hand nachrechnen
- [ ] Selbsttest ohne Paper beantworten
- [ ] Kernbotschaft und Folientitel festlegen: Storyboard mit einer Aussage pro Folie
- [ ] Optional: Gliederung kurz mit dem Betreuer abstimmen

### Phase 2 · Grafiken und Folien · bis Nov 1, 2026

- [ ] Zuerst die sechs Schlüsselgrafiken aus dem Folienplan bauen
- [ ] Farbcode festlegen und überall gleich nutzen: Referenz blau, Reads orange, Seeds grün, Fehler rot
- [ ] Folien nach Storyboard füllen, Titel als Aussagesätze
- [ ] Ergebnisplots neu zeichnen oder mit Quellenangabe übernehmen
- [ ] Backup-Folien für erwartbare Fragen anlegen

### Phase 3 · Proben und Überarbeitung · bis Nov 15, 2026

- [ ] Erster Durchlauf allein mit Stoppuhr, auf 23 bis 24 Minuten kürzen
- [ ] Probevortrag vor ein bis zwei Leuten ohne Biologie-Vorwissen
- [ ] Notieren, wo die Zuhörer hängen bleiben, und nachbessern
- [ ] Antworten auf die erwartbaren Fragen laut üben

### Phase 4 · Feinschliff · bis Nov 23, 2026

- [ ] 5 bis 10 komplette Durchläufe ohne Notizen (Vorgabe der Tipps-Folien)
- [ ] Einmal aufnehmen und ansehen
- [ ] Einstieg, Übergänge und Schlusssatz sicher beherrschen
- [ ] Technik prüfen: PDF auf Laptop und USB-Stick, Adapter, Presenter

### Vortrag · Nov 24, 2026 · 10 bis 12 Uhr c.t., Raum 04-522

25 Minuten ± 1 Minute, danach Diskussion. Bewertet werden Zeit, Themenaufbereitung, Folien, Vortragsqualität und Diskussion.

### Phase 5 · Ausarbeitung · 6 Wochen nach dem Vortrag, ohne Weihnachtsferien

- [ ] Direkt nach dem Vortrag die Fragen aus der Diskussion notieren
- [ ] Gliederung: Einleitung, Hintergrund, CloudBurst, Ergebnisse, Diskussion, Fazit
- [ ] 5 Seiten Volltext im LaTeX-Template aus Moodle (ohne Abstract, Literatur, Abbildungen, Tabellen)
- [ ] Grafiken aus dem Vortrag wiederverwenden
- [ ] Genaues Abgabedatum beim Betreuer bestätigen lassen

Laufend: Bei allen anderen Vorträgen anwesend sein und mitdiskutieren, denn die Beteiligung wird bewertet.

Offene Frage: Wann genau ist dein Vortragstermin?

## Kernbotschaft und Storyline

Kernbotschaft in einem Satz: CloudBurst macht hochsensitives Read-Mapping skalierbar, weil es Seed-and-Extend direkt auf MapReduce abbildet. Seeds werden zu Schlüsseln, der Shuffle findet die Kandidaten, und mehr Rechner bringen fast linear mehr Tempo.

### Drei Botschaften, die hängen bleiben sollen

1. **Schubfachprinzip:** Hat ein Read höchstens k Fehler, enthält er einen exakt passenden Seed der Länge m/(k+1). Deshalb geht kein Treffer verloren.
2. **Seeds als Schlüssel:** Map erzeugt Seeds aus Reads und Referenz, der Shuffle bringt gleiche Seeds zusammen, Reduce prüft die Kandidaten. Sehr häufige Seeds brauchen Load-Balancing.
3. **Skalierung:** Die Laufzeit wächst linear mit der Zahl der Reads, 96 Cloud-Kerne sind über 100× schneller als RMAP. Grenzen setzen der Overhead bei kleinem k und die Kandidatenflut bei großem k.

### Storyline

1. Problem: Sequenzierer liefern Millionen kurzer Reads, sensitives Mapping dauert auf einem Rechner Stunden.
2. Idee: Erst exakte Anker suchen, dann genau prüfen. Das Schubfachprinzip garantiert, dass dabei nichts verloren geht.
3. Umsetzung: Die Anker werden zu MapReduce-Schlüsseln, Hadoop verteilt die Arbeit.
4. Beleg: Lineare Skalierung, in der Cloud werden aus 14 Stunden 8 Minuten.
5. Einordnung: Was hat sich durchgesetzt, wo liegen die Grenzen?

### Regeln für alle Folien

- Eine Aussage pro Folie, als ganzer Satz im Titel.
- Ein durchgehendes Mini-Beispiel (derselbe Read, dasselbe Genomstück) von Folie 6 bis 11.
- Fester Farbcode in allen Grafiken, auch in neu gezeichneten Plots.
- Biologie nur so weit wie nötig: DNA als Text aus A, C, G und T.
- Zahlen runden und vergleichen: „14 Stunden → 8 Minuten“ statt „50.400 s“.
- Nichts zeigen, was du nicht erklärst. Details wandern in Backup-Folien.
- Folien und Vortrag in einer Sprache, Fachbegriffe dürfen englisch bleiben.

## Folienplan für 25 Minuten

20 Folien füllen 23:30 Minuten und lassen 1:30 Minuten Puffer. Die Aufteilung orientiert sich an den Tipps-Folien, mit etwas mehr Raum für Grundlagen: Motivation 3 Folien, Grundlagen und Methode 8, Ergebnisse 4, Einordnung und Fazit 2.

| Nr | Folientitel (Aussage) | Inhalt | Grafik | Dauer | Ende bei |
| --- | --- | --- | --- | --- | --- |
| 1 | CloudBurst: hochsensitives Read-Mapping mit MapReduce | Titel, Autor (M. C. Schatz, Bioinformatics 2009), dein Name, Seminar | – | 0:30 | 0:30 |
| 2 | Überblick | Problem · Grundlagen · CloudBurst · Ergebnisse · Einordnung | Fortschrittsleiste, die auf jeder Folie mitläuft | 0:30 | 1:00 |
| 3 | Sequenzierer liefern Millionen kurzer DNA-Schnipsel | Reads von 25 bis 250 bp, Genom mit rund 2,9 Mrd. Basen; für ein Genom wurden 4 Mrd. Reads à 35 bp kartiert (Bentley 2008) | Genom wird in viele Reads zerschnitten | 1:30 | 2:30 |
| 4 | Read-Mapping sucht für jeden Read seine Stelle im Genom, trotz Fehlern | Mismatch und Indel · Ursachen: echte Varianten (SNPs) und Sequenzierfehler · Anwendungen: SNP-Suche, Genotypisierung | Read unter der Referenz, Mismatch und Indel markiert | 1:30 | 4:00 |
| 5 | Mehr erlaubte Fehler machen die Suche drastisch langsamer | Schnelle Tools erlauben oft nur 2 Fehler und melden einen zufälligen Treffer · Problem und Lösung: RMAP-Sensitivität plus Hadoop | Waage Sensitivität gegen Laufzeit | 1:00 | 5:00 |
| 6 | Seed-and-Extend: erst exakte Anker finden, dann genau prüfen | Seed = exakt passendes Teilstück · Hashtabelle · Verlängerung zum ganzen Alignment | Mini-Beispiel: Read mit Anker auf der Referenz | 1:30 | 6:30 |
| 7 | k Fehler lassen immer einen Seed der Länge m/(k+1) unberührt | Read in k+1 Blöcke · Beispiel 36 bp, k = 4: 5 Blöcke à 7 bp · Folge: volle Sensitivität, aber kurze Seeds bringen viele Zufallstreffer | Read in farbigen Blöcken, Fehler treffen höchstens k davon | 1:30 | 8:00 |
| 8 | MapReduce in 60 Sekunden | Map, Shuffle, Reduce am k-mer-Zählen (Beispiel aus dem Paper) · HDFS, Datenlokalität, Fehlertoleranz | Datenfluss mit 2 Mappern und 2 Reducern | 1:30 | 9:30 |
| 9 | CloudBurst macht Seeds zu Schlüsseln | Map erzeugt Seeds aus Reads und Referenz · Shuffle bringt gleiche Seeds zusammen · Reduce verlängert | Gesamtpipeline nach Fig. 2, mit dem Mini-Beispiel | 1:00 | 10:30 |
| 10 | Die Referenz liefert alle k-mere, ein Read nur k+1 disjunkte | Warum die Asymmetrie (Schubfach) · Reverse Complement der Reads · Wert = MerInfo mit Position und Flanken | Mini-Beispiel: die emittierten Schlüssel-Wert-Paare | 1:30 | 12:00 |
| 11 | Der Reducer verlängert jedes Paar aus Referenz und Read | Mengen R und Q, alle Paare R × Q · Mismatches per Vergleich, Indels per Landau-Vishkin · Duplikate verwerfen | R × Q als Raster, daneben das DP-Band | 2:00 | 14:00 |
| 12 | Ein Seed wie AAAAAAA würde einen Reducer lahmlegen | Low-Complexity-Seeds brauchten über 1 h statt unter 1 min · Redundanz-Trick verteilt sie | Reducer-Last vorher und nachher als Balken | 1:15 | 15:15 |
| 13 | Ein zweiter Job behält nur das beste eindeutige Alignment | Schlüssel = Read-ID · Top-2-Optimierung und Combiner · Ausgabe identisch zu RMAP | kleiner zweiter Datenfluss | 0:45 | 16:00 |
| 14 | Test mit 7 Mio. echten Reads auf 24 Kernen | 7,06 Mio. Illumina-Reads à 36 bp · Genom, Chromosom 1, Chromosom 22 · bis 4 Mismatches · RMAP auf 1 Kern | Icons statt Tabelle | 0:45 | 16:45 |
| 15 | Die Laufzeit wächst linear mit den Reads, aber steil mit k | Fig. 3 · kürzere Seeds, mehr Zufallstreffer · 771 Mio. perfekte Treffer bei k = 0 · k = 4 auf dem ganzen Genom scheitert an voller Platte | Fig. 3 neu gezeichnet | 1:30 | 18:15 |
| 16 | Auf 24 Kernen 2× bis 33× schneller als RMAP, je sensitiver, desto besser | Fig. 4 · bei k = 0 dominiert der Overhead · über 24× laut Paper durch Implementierungsunterschiede und mehr Cache und RAM · Ad-hoc-Aufteilung schafft 12× bis 29× | Speedup-Balken mit Linie bei 24× | 1:30 | 19:45 |
| 17 | In der Cloud werden aus 14 Stunden 8 Minuten | 24 EC2-Kerne (High-CPU) schneller als der lokale Cluster · 4× mehr Kerne, 3,5× schneller · über 100× gegenüber RMAP | Laufzeit über Kernzahl mit Ideallinie | 1:15 | 21:00 |
| 18 | Stark bei voller Sensitivität, doch heute dominieren BWT-Aligner | Stärken und Grenzen (siehe „Kritische Punkte“) · Nachfolger Crossbow | zwei Spalten: Stärken, Grenzen | 1:15 | 22:15 |
| 19 | Fazit: drei Botschaften | Schubfachprinzip · Seeds als Schlüssel · lineare Skalierung | die drei Kerngrafiken verkleinert | 1:00 | 23:15 |
| 20 | Quellen und Fragen | Paper, MapReduce, RMAP, Bildquellen | – | 0:15 | 23:30 |

Wenn du beim Proben an Folie 13 später als 16:00 bist, kürze Folie 13 und 14 auf je 30 Sekunden.

### Sechs Schlüsselgrafiken, in dieser Reihenfolge bauen

1. Read unter der Referenz mit Mismatch und Indel (Folie 4)
2. Read in k+1 Blöcken, Fehler treffen höchstens k davon (Folie 7)
3. CloudBurst-Pipeline mit dem Mini-Beispiel (Folien 9 bis 11)
4. Reducer-Last vor und nach dem Redundanz-Trick (Folie 12)
5. Ergebnisplots: Laufzeit, Speedup, Cloud-Skalierung (Folien 15 bis 17)
6. Genom wird zu Reads zerschnitten (Folie 3)

### Backup-Folien

- Landau-Vishkin Schritt für Schritt
- MerInfo und Kodierung: 2 bit pro Base im Seed, 4 bit in den Flanken
- Überschläge zu Kandidatenflut und Shuffle-Volumen (siehe „Tiefer verstehen“)
- Ad-hoc-Parallelisierung im Detail
- EC2-Instanztypen und Kosten
- Amdahl-Rechnung zur Cloud-Skalierung
- Bowtie und BWT in zwei Sätzen
- Duplikat-Regel am Beispiel

## Paper verstehen

Lies das Paper dreimal mit steigender Tiefe. Nach dem zweiten Durchgang solltest du das Mini-Beispiel aus „CloudBurst Schritt für Schritt“ ohne Vorlage erklären können.

### Leseplan in drei Durchgängen

1. **Überblick, 20 Minuten:** Abstract, Einleitung, alle Überschriften, alle Abbildungen, Diskussion. Ziel: Problem, Idee und Hauptergebnis in je einem Satz sagen können.
2. **Gründlich, 2 bis 3 Stunden:** Abschnitt 1.2 und Abschnitt 2 mit Stift und Papier. Das Mini-Beispiel nachrechnen, jede Zahl aus Abschnitt 3 und 4 mit dem Zahlen-Spickzettel abgleichen, unklare Begriffe im Glossar nachschlagen.
3. **Kritisch, 1 bis 2 Stunden:** Behauptungen prüfen, die Überschläge selbst rechnen, Grenzen sammeln. Leitfrage: Was würde ich heute anders machen?

### Was in welchem Abschnitt steht

| Abschnitt | Inhalt | Rolle im Vortrag |
| --- | --- | --- |
| 1 Introduction | NGS-Datenflut, Read-Mapping, bestehende Tools | Motivation, Folien 3 bis 5 |
| 1.1 MapReduce and Hadoop | Map, Shuffle, Reduce, Combiner, Amdahl, HDFS, EC2 | kurz, Folie 8 |
| 1.2 Read mapping | Mismatch und Indel, Seed-and-Extend, Schubfach-Argument, BWT-Tools | zentral, Folien 6 und 7 |
| 2 Algorithm (2.1 bis 2.4) | Map, Shuffle, Reduce, Load-Balancing, Filterung | Kern, Folien 9 bis 13 |
| 3 Results | lokaler Cluster: Skalierung, Vergleich mit RMAP, Ad-hoc-Aufteilung | Folien 14 bis 16 |
| 4 Amazon Cloud Results | EC2-Instanztypen, Skalierung bis 96 Kerne | Folie 17 |
| 5 Discussion | Einordnung, Future Work | Folie 18 |

### Das Paper in fünf Sätzen

1. Problem: Sequenzierer erzeugen Millionen kurzer Reads. Sensitives Mapping mit vielen erlaubten Fehlern und allen Treffern ist auf einem Rechner sehr langsam.
2. Idee: CloudBurst übernimmt das Seed-and-Extend-Verfahren von RMAP und verteilt es mit Hadoop auf viele Rechner.
3. Umsetzung: Map emittiert Seeds aus Reads und Referenz, der Shuffle gruppiert gleiche Seeds, Reduce verlängert jedes Paar zu einem Alignment mit höchstens k Fehlern.
4. Ergebnis: Die Laufzeit wächst linear mit der Zahl der Reads. 24 Kerne sind bis etwa 30× schneller als RMAP, 96 Kerne auf EC2 über 100×, bei identischer Ausgabe.
5. Bedeutung: Seed-and-Extend passt natürlich zu MapReduce, und gemietete Cloud-Rechner machen sensitive Analysen auch ohne eigenen Cluster möglich.

## Hintergrund kompakt

Aus der Biologie brauchst du nur eine Handvoll Begriffe, aus der Informatik approximatives String-Matching und MapReduce. Die Tabellen sammeln alles, was das Paper voraussetzt, mit dem Abschnitt, in dem es vorkommt.

### Biologie

| Begriff | Bedeutung | Abschnitt |
| --- | --- | --- |
| DNA, Base, bp | DNA ist eine Folge der Basen A, C, G und T. Ein bp (Basenpaar) ist die Längeneinheit. | überall |
| Read | Kurzes, ausgelesenes DNA-Stück. Damalige Geräte lieferten 25 bis 250 bp, im Paper sind es 36 bp. | 1, 3 |
| Referenzgenom | Bereits bekannte Genomsequenz, auf die kartiert wird. Mensch (NCBI Build 36): 2,87 Gbp. | 1, 3 |
| NGS, Illumina/Solexa | Hochdurchsatz-Sequenzierung, die Millionen kurzer Reads pro Lauf liefert. | 1, 3 |
| SNP | Stelle, an der sich ein Genom in einer einzelnen Base von der Referenz unterscheidet. | 1 |
| Reverse Complement | DNA hat zwei komplementäre Stränge (A–T, C–G). Ein Read kann von jedem stammen, deshalb wird er auch umgedreht und komplementiert gesucht. | 2.1 |

### Informatik

| Begriff | Bedeutung | Abschnitt |
| --- | --- | --- |
| Mismatch | Eine Base ist ersetzt. Gezählt wird wie bei der Hamming-Distanz. | 1.2 |
| Indel, Edit-Distanz | Eine Base ist eingefügt oder gelöscht. Die Edit-Distanz zählt Ersetzungen, Einfügungen und Löschungen. | 1.2 |
| k | Maximal erlaubte Zahl an Unterschieden pro Read. | 2 |
| k-mer, Seed | Teilstring fester Länge s. Ein Seed ist ein k-mer, das exakt übereinstimmen muss. Achtung: Das k in „k-mer“ ist nicht das k der Fehler. | 1.2, 2 |
| Seed-and-Extend | Erst exakte Seeds finden, dann zu ganzen Alignments verlängern. | 1.2 |
| Smith-Waterman | Dynamische Programmierung für Alignments mit Indels. Laufzeit proportional zum Produkt der beiden Längen. | 1.2 |
| Landau-Vishkin | k-difference-Algorithmus: prüft nur die Diagonalen nahe der Hauptdiagonale, Laufzeit O(km). | 2 |
| RMAP | Serieller Read-Mapper (Smith et al. 2008), Vorbild und Vergleichsmaßstab. RMAPM ist die Variante mit Mismatch-Scores. | 1, 3 |
| BWT-Aligner | Bowtie, BWA, SOAP2: kompakter Index auf Basis der Burrows-Wheeler-Transformation, sehr schnell bei wenigen Fehlern. | 1.2 |
| MapReduce, Shuffle | Map erzeugt Schlüssel-Wert-Paare, der Shuffle gruppiert sie nach Schlüssel, Reduce verarbeitet jede Gruppe. | 1.1 |
| Combiner | Reduce-artige Vorverarbeitung direkt nach dem Map. Spart Daten im Shuffle. | 1.1, 2.4 |
| Hadoop, HDFS | Open-Source-MapReduce mit verteiltem Dateisystem: große Blöcke, mehrfach repliziert, Rechnen dort, wo die Daten liegen. | 1.1 |
| SequenceFile | Binäres Hadoop-Format für Schlüssel-Wert-Paare. CloudBurst wandelt die FASTA-Dateien vorher um. | 2 |
| Low-Complexity-Seed | Seed aus nur einer Base (AAAA…). Kommt extrem häufig vor und überlastet einzelne Reducer. | 2.1 |
| Speedup, Amdahl | Speedup = serielle Zeit / parallele Zeit. Ein serieller Anteil von 10 % begrenzt ihn auf 10×. | 1.1 |
| Sensitivität | Anteil der echten Treffer, die gefunden werden. CloudBurst findet alle mit höchstens k Unterschieden. | Abstract, 2 |

## CloudBurst Schritt für Schritt

CloudBurst besteht aus zwei MapReduce-Jobs. Der erste findet alle Alignments mit höchstens k Unterschieden, der zweite behält auf Wunsch nur das beste pro Read.

&#91;embedded content: CloudBurst-Ablauf nach Abschnitt 2 und Fig. 2 des Papers\]

Der Shuffle wirkt wie ein verteilter Hash-Join über die Seeds. Job 2 läuft nur, wenn pro Read das beste Alignment gebraucht wird.

### Die sieben Schritte

1. **Vorbereiten:** Reads und Referenz kommen als Multi-FASTA und werden in Hadoop-SequenceFiles im HDFS umgewandelt. Die Referenz wird in Stücke von 65 kb mit 1 kb Überlappung geteilt. Die Seed-Länge ist s = ⌊m/(k+1)⌋, mit minimaler Read-Länge m und erlaubten Unterschieden k.
2. **Map:** Für die Referenz wird jedes k-mer der Länge s emittiert, für jeden Read nur die k+1 nicht überlappenden, dazu dieselben vom Reverse Complement. Schlüssel = Seed (2 bit pro Base), Wert = MerInfo: ID, Position, isRef, isRC und die Flanken links und rechts (bis m − s + k Basen, 4 bit pro Base).
3. **Load-Balancing:** Low-Complexity-Seeds bekommen auf der Referenzseite r Kopien (AAAA-0 bis AAAA-(r−1)), jedes Read-Vorkommen geht zufällig an eine Kopie. Jedes Paar wird weiterhin genau einmal geprüft, nur verteilt auf r Reducer.
4. **Shuffle:** Hadoop gruppiert alle Werte mit demselben Seed. Jeder Reducer bekommt ungefähr 1/N der 4^s möglichen Seeds.
5. **Reduce:** Die Werte werden in R (Referenz) und Q (Reads) getrennt. Jedes Paar aus R × Q wird mithilfe der Flanken verlängert: Mismatches per Vergleich Base für Base, Indels mit Landau-Vishkin. Gearbeitet wird blockweise, damit die Daten im Cache bleiben.
6. **Duplikate verwerfen:** Enthält ein Alignment mehrere exakte Seeds, wird es mehrfach gefunden, ein perfekter Treffer sogar (k+1)-mal. Nur der Fund über den Seed mit dem kleinsten Offset im Read bleibt. Das entscheidet jeder Reducer allein anhand der Flanken.
7. **Optional filtern (Job 2):** Map emittiert (Read-ID, Alignment), Reduce behält pro Read das eindeutig beste Alignment, bei Gleichstand keins, genau wie RMAPM. Zur Beschleunigung melden die Reducer aus Job 1 pro Read nur ihre zwei besten Treffer, und ein Combiner filtert vor.

### Landau-Vishkin in drei Sätzen

Smith-Waterman füllt die ganze Tabelle aus Read und Referenzausschnitt. Bei höchstens k Unterschieden kann ein Alignment aber nur auf den 2k+1 Diagonalen um die Hauptdiagonale verlaufen. Landau-Vishkin merkt sich pro Diagonale und Fehlerzahl, wie weit man höchstens kommt, rutscht über passende Basen hinweg und braucht so O(km) statt O(m²).

### Mini-Beispiel zum Nachrechnen

Reads der Länge m = 8, erlaubt ist k = 1 Unterschied, also Seeds der Länge s = 4 und zwei Blöcke pro Read. Beide Reads gehören an Referenzposition 2 und haben je einen Mismatch.

```
Ref-Position   012345678901
Referenz       CTGACCTGGCAT
Read 1           GACCAGGC      Mismatch an Pos. 6 (Read A, Referenz T)
Read 2           GTCCTGGC      Mismatch an Pos. 3 (Read T, Referenz A)
```

So sieht der Shuffle die Schlüssel, und das tun die Reducer:

| Schlüssel (Seed) | Aus der Referenz (R) | Aus den Reads (Q) | Reducer |
| --- | --- | --- | --- |
| GACC | Position 2 | Read 1, Offset 0 | Start 2 − 0 = 2, Vergleich ergibt 1 Mismatch: Treffer |
| TGGC | Position 6 | Read 2, Offset 4 | Start 6 − 4 = 2, Vergleich ergibt 1 Mismatch: Treffer |
| AGGC | – | Read 1, Offset 4 | nur eine Seite, nichts zu tun |
| GTCC | – | Read 2, Offset 0 | nur eine Seite, nichts zu tun |
| CTGA, TGAC, ACCT, CCTG, CTGG, GGCA, GCAT | je eine Position | – | nichts zu tun |
| GCCT, GGTC, GCCA, GGAC (Reverse Complements) | – | je ein Read | nichts zu tun |

Read 1 wird über seinen ersten Block gefunden, Read 2 über den zweiten. Genau das garantiert das Schubfachprinzip: Der eine Fehler kann nur einen der zwei Blöcke treffen.

Duplikat-Fall: Ein fehlerfreier Read GACCTGGC trifft über beide Seeds, also in zwei verschiedenen Reducern. Der Reducer für TGGC (Offset 4) sieht in den Flanken, dass auch der Seed mit Offset 0 exakt passt, und verwirft seinen Fund.

## Ergebnisse und Zahlen

Alle Messungen nutzen 7,06 Mio. echte Reads à 36 bp und erlauben bis zu 4 Mismatches. Der Indel-Modus mit Landau-Vishkin wurde nicht gemessen.

### Zahlen-Spickzettel

| Was | Wert | Stelle |
| --- | --- | --- |
| Reads | 7,06 Mio. Illumina/Solexa-Reads à 36 bp aus dem 1000 Genomes Project (SRR001113) | Abschn. 3 |
| Referenzen | ganzes Genom 2,87 Gbp (NCBI Build 36), Chromosom 1 mit 247,2 Mbp, Chromosom 22 mit 49,7 Mbp | Abschn. 3 |
| Lokaler Cluster | 12 Knoten mit je Dual-Core Xeon 3,2 GHz (32 bit), zusammen 24 Kerne, 250 GB Platte pro Knoten, Hadoop 0.15.3 | Abschn. 3 |
| Einstellungen | 240 Mapper, 48 Reducer, Redundanz 16 für Low-Complexity-Seeds | Abschn. 3 |
| Low-Complexity-Seeds ohne Balancing | über 1 h statt unter 1 min | Abschn. 3 |
| Laufzeit über Read-Zahl | linear; mit wachsendem k steigt sie überlinear | Fig. 3 |
| k = 0, ganzes Genom | 771 Mio. perfekte Treffer aus den 7 Mio. Reads | Abschn. 3 |
| k = 4, ganzes Genom | abgebrochen nach rund 25 Mrd. Treffern, Platte voll | Abschn. 3, Fig. 3 |
| Vergleichssystem | RMAPM 0.41 auf 1 Kern eines AMD Opteron 250 (2,4 GHz, 64 bit, 8 GB RAM) | Abschn. 3 |
| Speedup gegenüber RMAP, 24 Kerne | 2× bis 33× je nach k und Referenz, im Abstract „bis zu 30×“ | Fig. 4 |
| Ad-hoc-Aufteilung: 24 RMAP-Läufe à 294.000 Reads auf Chr. 22 | 12× bei k = 0 bis 29× bei k = 4, gebremst durch ungleiche Laufzeiten | Abschn. 3 |
| EC2 mit 24 Kernen, Chr. 22, k = 4 | High-CPU Medium 1667 s, Small 3805 s, lokaler Cluster 1921 s | Abschn. 4 |
| EC2-Preise 2009 | Small 0,10 $ pro Stunde (1 virtueller Kern), High-CPU Medium 0,20 $ pro Stunde (2 virtuelle Kerne, etwa 5× Leistung) | Abschn. 4 |
| EC2-Skalierung | 96 Kerne 3,5× schneller als 24 (ideal 4×); RMAP über 14 h, CloudBurst rund 8 min | Abschn. 4, Fig. 5 |
| Tuning für größere Cluster | Reducer / Redundanz: 60 / 24 bei 48 Kernen, 144 / 72 bei 72, 196 / 72 bei 96 | Abschn. 4 |

### Welche Abbildung wofür

- Fig. 1, MapReduce-Schema: Folie 8, besser neu zeichnen
- Fig. 2, CloudBurst-Überblick: Folie 9
- Fig. 3, Laufzeit über Read-Zahl für Genom, Chr. 1 und Chr. 22: Folie 15
- Fig. 4, Speedup gegenüber RMAP: Folie 16
- Fig. 5, Skalierung auf EC2: Folie 17

## Tiefer verstehen: eigene Überschläge

Vier kleine Rechnungen erklären, warum CloudBurst sich so verhält, wie Fig. 3 bis 5 zeigen. Es sind eigene Überschläge unter vereinfachten Annahmen, keine Zahlen aus dem Paper. Sie eignen sich als Backup-Folien und für die Diskussion.

### 1. Zufallstreffer pro Seed

Ein zufälliger Seed der Länge s kommt in einer Referenz der Länge L im Mittel so oft vor (Formel aus Abschnitt 3 des Papers):

```latex
E = \frac{L - s + 1}{4^{s}}
```

Im ganzen Genom trifft ein 18-mer (k = 1) im Mittel 0,04-mal, ein 7-mer (k = 4) rund 175.000-mal. Im Paper steht für das 7-mer „> 17 500“. Nach seiner eigenen Formel fehlt dort eine Null.

### 2. Kandidatenflut

Jeder Read liefert k+1 Seeds pro Strang. Die Zahl der zufälligen Kandidatenpaare, die ein Reducer prüfen muss, ist ungefähr:

```latex
N_{\mathrm{Kand}} \approx 2\,(k+1)\cdot n_{\mathrm{Reads}}\cdot\frac{L - s + 1}{4^{s}}
```

&#91;embedded content: Eigener Überschlag nach der Formel aus Abschnitt 3 · 7,06 Mio. Reads, k = 1 bis 4\]

Zwei Effekte wirken zusammen: kürzere Seeds treffen exponentiell öfter, und jeder Read liefert mehr Seeds. Echte Genome sind noch ungünstiger als die Zufallsannahme. Wiederholungen liefern schon bei k = 0 insgesamt 771 Mio. perfekte Treffer, gut 100 pro Read.

### 3. Shuffle-Volumen

Jede Referenzposition erzeugt ein Schlüssel-Wert-Paar mit Flanken von bis zu m − s + k Basen auf jeder Seite. Bei 36-bp-Reads sind das grob 35 bis 45 Byte pro Position (angenommen: 10 Byte für ID, Position und Flags, ohne Hadoop-Overhead). Für das ganze Genom ergibt das rund 100 bis 130 GB pro Lauf, für Chromosom 22 etwa 2 GB, unabhängig von der Zahl der Reads.

Dieser fixe Block erklärt, warum der Speedup bei k = 0 klein bleibt und auf dem ganzen Genom kaum steigt. Er erklärt auch den Rat des Papers, mehr Reads pro Lauf zu kartieren: Der Aufwand für die Referenz verteilt sich dann auf mehr Reads.

### 4. Amdahl auf der Cloud-Skalierung

4× mehr Kerne brachten 3,5× Tempo. Nach Amdahl skalieren damit etwa 5 % des 24-Kern-Laufs nicht mit, laut Paper vor allem wegen einzelner Reducer, die länger laufen als die anderen.

```latex
\frac{T_{96}}{T_{24}} = f + \frac{1-f}{4} = \frac{1}{3{,}5} \quad\Rightarrow\quad f \approx 0{,}048
```

Mit diesem f brächten 192 Kerne nur 6× statt 8× Tempo gegenüber 24 Kernen, und mehr als 21× ist nie drin. Gute Diskussionsfrage: Lohnt sich mehr Hardware, oder zuerst besseres Load-Balancing?

## Kritische Punkte und erwartbare Fragen

Das Paper ist sauber, lässt aber einige Fragen offen. Die folgenden Punkte gehören auf die Einordnungsfolie (Folie 18) und in deine Vorbereitung auf die Diskussion.

### Kritische Punkte

- Nur der Mismatch-Modus wurde gemessen. Der Indel-Modus mit Landau-Vishkin ist beschrieben, aber nicht evaluiert.
- RMAP lief auf anderer Hardware (64-bit Opteron, 2,4 GHz) als CloudBurst (32-bit Xeon, 3,2 GHz). Speedups über 24× sind deshalb kein reiner Algorithmus-Effekt.
- Die Zeit zum Umwandeln und Laden ins HDFS ist aus den Laufzeiten herausgerechnet (Abschnitt 3).
- Es gibt nur einen Datensatz: 36-bp-Reads, single-end, ohne Qualitätswerte.
- „Alle Treffer“ erzeugt riesige Ausgaben: 771 Mio. bei k = 0, Abbruch nach 25 Mrd. bei k = 4.
- Die ganze Referenz wird in jedem Lauf neu emittiert und geshuffelt. Nur größere Read-Batches verteilen diesen Aufwand.
- Kleiner Rechenfehler: Ein zufälliges 7-mer trifft das Genom rund 175.000-mal, nicht „> 17 500“-mal.
- Heute dominieren BWT-Aligner wie Bowtie und BWA, Reads sind länger und gepaart, und für Big-Data-Pipelines hat Spark Hadoop MapReduce weitgehend abgelöst.

### Erwartbare Fragen

| Frage | Antwortskizze | Backup-Folie |
| --- | --- | --- |
| Findet CloudBurst garantiert alle Treffer mit höchstens k Unterschieden? | Ja. k+1 disjunkte Blöcke, k Fehler treffen höchstens k davon. Das gilt auch für Indels (Baeza-Yates 1992, im Paper zitiert). | Schubfach (Folie 7) |
| Warum alle k-mere der Referenz, aber nur disjunkte der Reads? | Der fehlerfreie Block kann an jeder Genomposition beginnen. Beim Read reicht die Zerlegung in k+1 Blöcke, das spart Paare. | Mini-Beispiel |
| Warum nicht einfach die Reads aufteilen und RMAP 24-mal starten? | Im Paper getestet: nur 12× bis 29×, weil die Teilläufe unterschiedlich lange brauchen. Dazu fehlen Fehlertoleranz und Datenlokalität, und jeder Rechner braucht die ganze Referenz. | Ad-hoc-Parallelisierung |
| Warum ist der Speedup bei k = 0 so klein? | Der fixe Aufwand, alle Referenz-k-mere zu emittieren und zu shufflen, dominiert. Pro Seed gibt es kaum Rechenarbeit (Amdahl). | Shuffle-Volumen |
| Wie verhindert CloudBurst doppelte Treffer? | Nur der Seed mit dem kleinsten Offset behält ein Alignment. Jeder Reducer entscheidet das lokal anhand der Flanken. | Duplikat-Regel |
| Was passiert mit sehr häufigen Seeds wie AAAAAAA? | Redundanz-Trick: Referenzkopien, Reads zufällig verteilt. Gleiche Gesamtarbeit, aber parallel. | Folie 12 |
| Warum haben sich BWT-Aligner durchgesetzt? | Kompakter Index, sehr schnell bei wenigen Fehlern und Suche nach dem besten Treffer, und das reicht den meisten Anwendungen. CloudBurst lohnt sich, wenn alle Treffer oder viele Fehler gebraucht werden. | Bowtie und BWT |
| Würde man das heute mit Spark bauen? | Möglich. Vorteil: Referenz-k-mere im Speicher halten und für mehrere Read-Batches wiederverwenden. Das Shuffle-Volumen pro Batch bleibt. | – |
| Nutzt CloudBurst Basenqualitäten oder Paired-End-Reads? | Nein, beides nennt das Paper als Future Work. | – |
| Was kostet ein Lauf in der Cloud? | Eigene Rechnung: 96 Kerne = 48 High-CPU-Medium-Instanzen × 0,20 $ ≈ 10 $ pro Stunde Clusterbetrieb, der Lauf dauert rund 8 Minuten. Dazu kommen Start und Datenupload. | EC2-Instanztypen und Kosten |
| Wie viel bringt mehr Hardware noch? | Amdahl-Schätzung: Etwa 5 % skalieren nicht, also höchstens 21× gegenüber 24 Kernen. Besseres Load-Balancing hilft mehr. | Amdahl-Rechnung |

## Selbsttest

Beantworte die Fragen laut und ohne Paper, erst dann die Antworten unten lesen. Wenn du alle elf sicher beantwortest, ist Phase 1 inhaltlich geschafft.

- [ ] 1\. Was unterscheidet einen Mismatch von einem Indel, und wie verlängert CloudBurst jeweils einen Seed?
- [ ] 2\. Reads mit 36 bp, k = 3: Wie lang ist ein Seed, und wie viele Seeds emittiert ein Read insgesamt?
- [ ] 3\. Was emittiert der Mapper für ein Stück Referenz, was für einen Read (Schlüssel und Wert)?
- [ ] 4\. Warum braucht CloudBurst das Reverse Complement, und warum nur bei den Reads?
- [ ] 5\. Was bekommt ein Reducer, und welche Paare prüft er?
- [ ] 6\. Wann wird dasselbe Alignment mehrfach gefunden, und wie verhindert CloudBurst doppelte Ausgaben?
- [ ] 7\. Wie funktioniert der Redundanz-Trick, und warum ändert er das Ergebnis nicht?
- [ ] 8\. Warum wächst die Laufzeit mit k überlinear?
- [ ] 9\. Warum liegt der Speedup gegenüber RMAP mal bei 2× und mal über 24×?
- [ ] 10\. Wozu dient der zweite MapReduce-Job?
- [ ] 11\. Was zeigt der Vergleich von 24 und 96 Kernen in der Cloud?

### Antworten

1. Mismatch: eine Base ersetzt. Indel: eine Base eingefügt oder gelöscht. Mismatches werden durch Vergleich der Flanken gezählt, Indels mit Landau-Vishkin in O(km) geprüft.
2. s = ⌊36/4⌋ = 9. Pro Strang 4 Seeds, mit Reverse Complement 8.
3. Referenz: für jede Position (Seed, MerInfo mit isRef = 1, Position, Flanken). Read: für die k+1 disjunkten Blöcke beider Stränge (Seed, MerInfo mit isRef = 0, isRC, Offset, Flanken).
4. Ein Read kann vom Gegenstrang stammen. Die Reads umzudrehen ist billiger, als die ganze Referenz zweimal zu emittieren.
5. Alle Werte zu einem Seed, getrennt in R (Referenz) und Q (Reads). Er prüft jedes Paar aus R × Q.
6. Wenn ein Alignment mehrere exakte Seeds enthält, ein perfekter Treffer etwa k+1. Nur der Fund über den Seed mit dem kleinsten Offset bleibt.
7. Die Referenz-Vorkommen eines häufigen Seeds werden r-mal kopiert, jedes Read-Vorkommen geht an genau eine zufällige Kopie. Jedes Paar wird also genau einmal geprüft, nur verteilt auf r Reducer.
8. Größeres k bedeutet kürzere Seeds mit exponentiell mehr Zufallstreffern (L/4^s), dazu mehr Seeds pro Read.
9. Bei k = 0 dominiert der fixe Aufwand für das Shufflen der Referenz. Bei großem k dominiert Rechenarbeit, die gut skaliert. Über 24× erklärt das Paper mit Implementierungsunterschieden und den zusätzlichen Ressourcen im Cluster (Cache, RAM, Platten-I/O).
10. Er behält pro Read nur das eindeutig beste Alignment, bei Gleichstand keins. So entsteht dieselbe Ausgabe wie bei RMAPM.
11. Kein eigener Cluster nötig, die Größe ist frei wählbar. 4× mehr Kerne bringen 3,5× Tempo, rund 88 % Effizienz, und über 100× gegenüber RMAP.

## Literatur und Quellen

Für Phase 1 reichen das Paper, die ersten drei Abschnitte des MapReduce-Artikels und der Leseleitfaden von Keshav. Der Rest ist Vertiefung für Backup-Folien und Ausarbeitung.

### Für Phase 1

- Schatz, M. C. (2009): [CloudBurst: highly sensitive read mapping with MapReduce](https://doi.org/10.1093/bioinformatics/btp236). Bioinformatics 25(11), 1363–1369. Open Access, Quellcode laut Paper unter cloudburst-bio.sourceforge.net.
- Dean, J.; Ghemawat, S. (2008): [MapReduce: Simplified Data Processing on Large Clusters](https://doi.org/10.1145/1327452.1327492). Communications of the ACM 51(1), 107–113.
- Keshav, S. (2007): [How to read a paper](https://doi.org/10.1145/1273445.1273458). ACM SIGCOMM Computer Communication Review 37(3), 83–84. Grundlage des Leseplans.

### Vertiefung, aus der Literaturliste des Papers

- Smith, A. D. et al. (2008): Using quality scores and longer reads improves accuracy of Solexa read mapping. BMC Bioinformatics 9, 128. Das RMAP-Paper.
- Landau, G. M.; Vishkin, U. (1986): Introducing efficient parallelism into approximate string matching and a new serial algorithm. STOC 1986, 220–230.
- Gusfield, D. (1997): Algorithms on Strings, Trees, and Sequences. Cambridge University Press. Lehrbuch-Erklärung zum k-difference-Problem.
- Baeza-Yates, R. A. et al. (1992): Fast and practical approximate string matching. CPM 1992, 185–192. Quelle des Schubfach-Arguments.

### Einordnung

- Langmead, B. et al. (2009): [Ultrafast and memory-efficient alignment of short DNA sequences to the human genome](https://doi.org/10.1186/gb-2009-10-3-r25). Genome Biology 10, R25. Bowtie, der BWT-Gegenentwurf.
- Langmead, B.; Schatz, M. C. et al. (2009): Searching for SNPs with cloud computing. Genome Biology 10, R134. Laut [Crossbow-Projektseite](https://bowtie-bio.sourceforge.net/crossbow/index.shtml) eine Hadoop-Pipeline aus Bowtie und SoapSNP, aus derselben Arbeitsgruppe.

### Vortragstipps aus den Seminarfolien

- [How to present a paper](https://cc.gatech.edu/faculty/ashwin/wisdom/how-to-present-a-paper.html)
- [Oral presentation advice](http://pages.cs.wisc.edu/~markhill/conference-talk.html)
- [Pointers on giving a talk](https://people.eecs.berkeley.edu/~messer/Bad_talk.html)
