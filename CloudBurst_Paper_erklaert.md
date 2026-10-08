# CloudBurst verständlich erklärt

Erklärung zu: Michael C. Schatz (2009): *CloudBurst: highly sensitive read mapping with MapReduce.* Bioinformatics 25(11), S. 1363–1369, doi:10.1093/bioinformatics/btp236. Das PDF liegt im Repo als `07-cloudburst-read-mapping.pdf`.

Dieses Dokument erklärt das Paper von Grund auf und auf Deutsch. Es setzt nur Informatik-Grundwissen voraus (Strings, Hashtabellen, O-Notation). Alles andere, also die nötige Biologie, Read-Mapping, Seed-and-Extend, MapReduce, Hadoop und die Cloud, wird erklärt, bevor es im Algorithmus gebraucht wird. Der [Vortragsplan](Vortragsplan_CloudBurst.md) baut darauf auf.

## Inhalt

1. [Zusammenfassung](#1-zusammenfassung)
2. [Biologische Grundlagen](#2-biologische-grundlagen)
3. [Das Problem: Read-Mapping](#3-das-problem-read-mapping)
4. [Seed-and-Extend](#4-seed-and-extend)
5. [MapReduce, Hadoop und die Cloud](#5-mapreduce-hadoop-und-die-cloud)
6. [Der CloudBurst-Algorithmus](#6-der-cloudburst-algorithmus)
7. [Experimente und Ergebnisse](#7-experimente-und-ergebnisse)
8. [Diskussion und Einordnung](#8-diskussion-und-einordnung)
9. [Glossar](#9-glossar)
10. [Quellen](#10-quellen)

### Lesehinweise

- Wer MapReduce schon kennt, kann Kapitel 5 überfliegen. Kapitel 3 und 4 sollte man aber lesen, weil der Algorithmus direkt darauf aufbaut.
- Angaben wie „(Paper 2.1)“ verweisen auf den Abschnitt im Originalpaper.
- Rechnungen und Einschätzungen, die nicht aus dem Paper stammen, sind als „eigene Rechnung“ oder „Einordnung“ markiert.

### Notation

| Symbol | Bedeutung | Wert im Paper |
| --- | --- | --- |
| m | Länge eines Reads (genauer: minimale Read-Länge) | 36 bp |
| k | maximal erlaubte Zahl an Unterschieden (Fehlern) pro Read | 0 bis 4 |
| s | Länge eines Seeds, s = ⌊m/(k+1)⌋ | 36, 18, 12, 9, 7 |
| L | Länge der Referenzsequenz | 49,7 Mbp bis 2,87 Gbp |
| n | Anzahl der Reads | bis 7,06 Mio. |
| N | Anzahl der Reducer bzw. Rechenkerne | 24 bis 96 Kerne |
| r | Redundanz für Low-Complexity-Seeds | 16 bis 72 |

Achtung, Verwechslungsgefahr: Das Paper sagt „k-mer“ zu einem Teilstring beliebiger fester Länge. Das k in „k-mer“ hat nichts mit dem k der erlaubten Fehler zu tun. In CloudBurst haben die k-mere die Länge s.

---

## 1. Zusammenfassung

### 1.1 Das Paper in einem Absatz

DNA-Sequenzierer der damals neuen Generation liefern pro Lauf Millionen kurzer DNA-Stücke, sogenannte Reads. Um sie auszuwerten, sucht man für jeden Read die Stelle im bekannten Referenzgenom, an der er vorkommt, und erlaubt dabei ein paar Unterschiede. Das nennt man Read-Mapping. Je mehr Unterschiede man erlaubt und je mehr Treffer man pro Read haben will, desto länger dauert das: auf einem einzelnen Rechner viele Stunden. CloudBurst übernimmt das Verfahren des Read-Mappers RMAP und verteilt es mit Hadoop, einer Open-Source-Implementierung von MapReduce, auf viele Rechner. Die Grundidee ist einfach. Kurze, exakt passende Teilstücke (Seeds) werden zu Schlüsseln, und MapReduce bringt automatisch alle Stellen mit demselben Seed aus Reads und Referenz zusammen. Dort werden die Seeds dann zu vollständigen Alignments verlängert. Die Laufzeit wächst linear mit der Zahl der Reads. Auf 24 Kernen ist CloudBurst bis zu etwa 30-mal schneller als RMAP auf einem Kern, auf 96 Kernen in der Amazon-Cloud über 100-mal, und die Ergebnisse sind identisch.

### 1.2 Problem, Idee, Umsetzung, Ergebnis

**Problem.** Ein menschliches Genom hat knapp 3 Milliarden Basen. Ein Sequenzierlauf liefert Millionen bis Milliarden Reads von 25 bis 250 Basen Länge. Für jeden Read sollen alle Stellen gefunden werden, an denen er mit höchstens k Unterschieden passt. Damals verfügbare schnelle Programme erlaubten oft nur wenige Unterschiede oder meldeten nur einen zufällig gewählten Treffer pro Read. Sensitive Programme wie RMAP waren dagegen langsam, weil sie nur auf einem Rechner liefen.

**Idee.** Seed-and-Extend: Wenn ein Read höchstens k Fehler hat und man ihn in k+1 Blöcke teilt, ist mindestens ein Block fehlerfrei (Schubfachprinzip). Also sucht man zuerst exakte Treffer dieser Blöcke und prüft nur dort genauer. Dieses Verfahren passt sehr gut zu MapReduce, weil das Gruppieren nach exakt gleichen Seeds genau das ist, was die Shuffle-Phase von MapReduce ohnehin tut.

**Umsetzung.** CloudBurst besteht aus zwei MapReduce-Jobs.

- Job 1, Map: Aus der Referenz wird jeder Teilstring der Länge s als Seed ausgegeben, aus jedem Read nur die k+1 nicht überlappenden Blöcke, zusätzlich auch vom Gegenstrang.
- Job 1, Shuffle: Hadoop gruppiert alle Einträge mit demselben Seed.
- Job 1, Reduce: Jedes Paar aus Referenzstelle und Read wird zu einem vollständigen Alignment verlängert und geprüft. Doppelte Funde werden verworfen.
- Job 2 (optional): Pro Read bleibt nur das eindeutig beste Alignment übrig.

**Ergebnis.** Getestet wurde mit 7,06 Mio. echten Reads (36 bp) gegen das menschliche Genom und zwei Chromosomen, mit bis zu 4 Mismatches.

- Die Laufzeit wächst linear mit der Zahl der Reads und überlinear mit k.
- 24 Kerne sind je nach Einstellung 2- bis 33-mal schneller als RMAP auf einem Kern.
- In der Amazon-Cloud skaliert CloudBurst von 24 auf 96 Kerne mit Faktor 3,5. Ein Lauf, für den RMAP über 14 Stunden braucht, dauert rund 8 Minuten.

**Bedeutung laut Autor.** Seed-and-Extend-Verfahren mit Hashtabelle (BLAST, SOAP, MAQ, ZOOM) ließen sich alle so mit MapReduce umsetzen. Gemietete Cloud-Rechner machen solche Analysen auch ohne eigenen Cluster möglich.

### 1.3 Die wichtigsten Zahlen

| Was | Wert |
| --- | --- |
| Testdaten | 7,06 Mio. Illumina/Solexa-Reads à 36 bp (1000 Genomes Project, SRR001113) |
| Referenzen | ganzes Genom 2,87 Gbp, Chromosom 1 mit 247,2 Mbp, Chromosom 22 mit 49,7 Mbp |
| Lokaler Cluster | 12 Knoten, je 2 Kerne, also 24 Kerne, Hadoop 0.15.3 |
| Speedup gegenüber RMAP (24 Kerne) | 2× bis 33×, je nach k und Referenz |
| Perfekte Treffer bei k = 0, ganzes Genom | 771 Mio. für 7 Mio. Reads |
| k = 4, ganzes Genom | nach rund 25 Mrd. Treffern abgebrochen, weil die Festplatten voll waren |
| Amazon EC2, 24 → 96 Kerne | 3,5-mal schneller (ideal wäre 4×) |
| RMAP gegen CloudBurst auf 96 Kernen | über 14 h gegen rund 8 min, also über 100× |

### 1.4 Aufbau des Papers

| Abschnitt | Inhalt | Hier erklärt in |
| --- | --- | --- |
| Abstract, 1 Introduction | Datenflut durch neue Sequenzierer, Read-Mapping, bestehende Tools | Kap. 2 und 3 |
| 1.1 MapReduce and Hadoop | Programmiermodell, Combiner, Amdahl, GFS/HDFS, Cloud | Kap. 5 |
| 1.2 Read mapping | Mismatches, Indels, Seed-and-Extend, Schubfach-Argument, andere Tools | Kap. 3 und 4 |
| 2 Algorithm (2.1 bis 2.4) | Map, Load-Balancing, Shuffle, Reduce, Filterung | Kap. 6 |
| 3 Results | Messungen auf dem lokalen Cluster | Kap. 7.1 bis 7.4 |
| 4 Amazon Cloud Results | Messungen auf EC2 | Kap. 7.5 und 7.6 |
| 5 Discussion | Bewertung und Ausblick | Kap. 8 |

---

## 2. Biologische Grundlagen

Für das Paper braucht man nur sehr wenig Biologie. Man kann DNA fast vollständig als Text behandeln.

### 2.1 DNA als Text

DNA ist eine lange Kette aus vier Bausteinen, den Basen Adenin, Cytosin, Guanin und Thymin. Man schreibt sie als Buchstaben A, C, G und T. Ein Genom ist damit ein sehr langer String über dem Alphabet {A, C, G, T}.

- Die Längeneinheit heißt bp (base pair, Basenpaar). 1 kb = 1.000 bp, 1 Mbp = 1 Mio. bp, 1 Gbp = 1 Mrd. bp.
- Das menschliche Genom hat rund 3 Gbp. Die im Paper verwendete Version (NCBI Build 36) hat 2,87 Gbp.
- Ein Genom ist auf Chromosomen verteilt. Chromosom 1 ist das größte (247,2 Mbp), Chromosom 22 eines der kleinsten (49,7 Mbp). Das Paper nutzt beide als kleinere Testreferenzen.
- Manchmal ist eine Base unbekannt. Dann steht dort ein N.

### 2.2 Die zwei Stränge und das Reverse Complement

DNA ist doppelsträngig. Die beiden Stränge sind komplementär: Gegenüber von A steht immer T, gegenüber von C immer G. Außerdem laufen die Stränge in entgegengesetzter Richtung. Liest man den Gegenstrang in seiner eigenen Leserichtung, erhält man das **Reverse Complement**: den String umdrehen und jede Base durch ihren Partner ersetzen.

```
Strang:              5' - A C G G T - 3'
Gegenstrang:         3' - T G C C A - 5'
Reverse Complement:  A C C G T   (Gegenstrang von rechts nach links gelesen, also von 5' nach 3')
```

Für das Mapping heißt das: Ein Read kann von jedem der beiden Stränge stammen. Die Referenz enthält aber nur einen Strang. Deshalb muss man jeden Read zweimal suchen, einmal so wie er ist und einmal als Reverse Complement.

### 2.3 Sequenzierung und Reads

Ein Sequenzierer kann ein Genom nicht am Stück auslesen. Die DNA wird in viele kleine Stücke zerbrochen, und von jedem Stück wird ein kurzer Abschnitt gelesen. Ein solcher Abschnitt heißt **Read**. Weil man viele Kopien der DNA zerbricht, überlappen sich die Reads und decken das Genom mehrfach ab.

```
Genom:   ...ACGTTGCAAGCTTAGGCTAACGTTAGCCATGGATCCA...
Reads:         TGCAAGCTTA
                  AAGCTTAGGCTA
                          GCTAACGTTAGC
                                  TTAGCCATGGAT
```

Die Sequenzierer der „nächsten Generation“ (next-generation sequencing, NGS) von 454 Life Sciences, Illumina (Solexa) oder Applied Biosystems lesen in wenigen Tagen mehr DNA als ein klassisches Sanger-Gerät in einem Jahr (Paper 1). Die Reads sind dafür kurz, damals 25 bis 250 bp. Das Paper nennt als Beispiele zwei menschliche Genome, für die 4,0 bzw. 3,3 Milliarden Reads à 35 bp kartiert wurden.

Reads werden meist im FASTA-Format gespeichert. Jeder Eintrag hat eine Kopfzeile mit `>` und dann die Sequenz. Eine Datei mit vielen Einträgen heißt Multi-FASTA:

```
>read_1
GACCAGGC
>read_2
GTCCTGGC
```

Einige Begriffe, die im Paper fallen:

- **Single-end:** Von jedem DNA-Stück wird nur ein Ende gelesen. CloudBurst ist dafür gebaut.
- **Paired-end:** Beide Enden eines Stücks werden gelesen, ihr ungefährer Abstand ist bekannt. Das Paper nennt die Unterstützung dafür als zukünftige Arbeit.
- **Qualitätswerte:** Der Sequenzierer gibt zu jeder Base an, wie sicher er sich ist. CloudBurst nutzt sie nicht.

### 2.4 Referenzgenom, Varianten und Fehler

Ein **Referenzgenom** ist eine bereits bekannte, zusammengesetzte Genomsequenz einer Art. Neue Reads werden darauf abgebildet, um herauszufinden, wo sich das untersuchte Genom von der Referenz unterscheidet.

Ein Read passt fast nie perfekt. Es gibt zwei Ursachen für Unterschiede:

1. Echte genetische Unterschiede. Der häufigste Fall ist der **SNP** (single nucleotide polymorphism): An einer Stelle steht eine andere Base als in der Referenz. Es kommen auch kleine Einfügungen oder Löschungen vor.
2. Sequenzierfehler: Das Gerät liest eine Base falsch.

Genau diese Unterschiede will man oft finden, etwa um SNPs zu entdecken oder einen Menschen zu genotypisieren. Das Paper betont: Schon eine einzelne andere Base kann biologisch bedeutsam sein. Deshalb braucht man Mapping-Verfahren, die Unterschiede zuverlässig zulassen.

---

## 3. Das Problem: Read-Mapping

### 3.1 Was Read-Mapping ist

Gegeben sind eine Referenz (ein langer String) und viele Reads (kurze Strings). Gesucht ist für jeden Read jede Stelle in der Referenz, an der der Read mit höchstens k Unterschieden vorkommt. Das ist approximatives String-Matching, nur in sehr großem Maßstab.

```
Ref-Position   0123456789012
Referenz       CTGACCTGGCATG
Read                CTGGCTT      passt ab Position 5 mit 1 Unterschied (Pos. 10: Read T, Referenz A)
```

Das Ergebnis pro Treffer heißt **Alignment**: Read-ID, Position in der Referenz, Strang (vorwärts oder Reverse Complement) und Zahl der Unterschiede. „End-to-end“ heißt, dass der ganze Read ausgerichtet wird und nicht nur ein Teil davon.

### 3.2 Mismatches und Indels

Es gibt zwei Arten von Unterschieden:

```
Mismatch (Ersetzung):            Indel (Einfügung oder Löschung):

Referenz  G A C C T G G C        Referenz  G A C C T G G C
Read      G A C C A G G C        Read      G A C - T G G C
                  ^                              ^
           eine Base ersetzt              eine Base fehlt im Read
```

- Ein **Mismatch** ist eine ersetzte Base. Die Länge bleibt gleich.
- Ein **Indel** (insertion/deletion) ist eine eingefügte oder gelöschte Base. Danach ist alles um eine Stelle verschoben.

### 3.3 Hamming-Distanz und Edit-Distanz

Die beiden Fehlerarten führen zu zwei Abstandsmaßen.

**Hamming-Distanz:** Bei gleich langen Strings zählt man die Positionen, an denen sie sich unterscheiden. Das geht mit einem einzigen Durchlauf in O(m). Wer nur Mismatches erlaubt, braucht nur das.

**Edit-Distanz (Levenshtein-Distanz):** die minimale Zahl an Ersetzungen, Einfügungen und Löschungen, die einen String in den anderen überführt. Man berechnet sie mit dynamischer Programmierung (DP). Für zwei Strings a und b füllt man eine Tabelle D, in der D[i][j] die Edit-Distanz der ersten i Zeichen von a und der ersten j Zeichen von b ist:

```
D[i][j] = min( D[i-1][j]   + 1,              # Zeichen aus a löschen
               D[i][j-1]   + 1,              # Zeichen aus b einfügen
               D[i-1][j-1] + (a[i] ≠ b[j]) ) # Treffer (0) oder Ersetzung (1)
```

Beispiel: Read GATC gegen Referenzausschnitt GAATC.

```
         -  G  A  A  T  C
    -    0  1  2  3  4  5
    G    1  0  1  2  3  4
    A    2  1  0  1  2  3
    T    3  2  1  1  1  2
    C    4  3  2  2  2  1     ← Edit-Distanz 1 (dem Read fehlt ein A)
```

Die Tabelle hat (m+1)·(n+1) Zellen, die Laufzeit ist also proportional zum Produkt der Längen. Der bekannteste Algorithmus dieser Art für Sequenzen ist **Smith-Waterman** (1981). Er berechnet lokale Alignments mit Bewertungen statt reiner Fehlerzählung, folgt aber demselben DP-Prinzip. Das Paper nennt ihn als Beispiel für die teurere Variante: Für ein einzelnes Paar kurzer Strings ist DP schnell, bei Millionen Reads und Milliarden Positionen aber nicht mehr.

### 3.4 Sensitivität und warum sie teuer ist

**Sensitivität** heißt: Wie viele der tatsächlich vorhandenen Treffer findet ein Verfahren? CloudBurst ist voll sensitiv. Es findet garantiert jedes Alignment mit höchstens k Unterschieden.

Laut Paper erlauben Mapping-Programme typischerweise Unterschiede in Höhe von 1 bis 10 % der Read-Länge. Bei 36 bp entspricht das bis zu etwa 4 Fehlern, genau der Bereich, den das Paper testet. Eine Studie der RMAP-Autoren (Smith et al. 2008) ergab, dass man für längere Reads mehr als zwei Mismatches erlauben muss, um sie korrekt zuzuordnen.

Zwei Wünsche treiben die Kosten hoch:

1. **Mehr erlaubte Fehler.** Wie Kapitel 4 zeigt, werden die Seeds dann kürzer und finden viel mehr zufällige Kandidaten.
2. **Alle Treffer statt einem.** Viele Reads stammen aus sich wiederholenden Genomabschnitten und passen an Hunderten Stellen.

Die damals neuen, sehr schnellen Programme (Bowtie, BWA, SOAP2) erlaubten in ihrer Standardeinstellung höchstens zwei Unterschiede am Anfang des Reads und meldeten nur einen zufällig gewählten Treffer (Paper 1.2). In sensitiveren Einstellungen wurden sie deutlich langsamer. Genau diese Lücke will CloudBurst füllen.

### 3.5 Größenordnung (eigene Rechnung)

Der naive Ansatz vergleicht jeden Read mit jeder Position der Referenz. Bei 7 Mio. Reads und 2,87 Mrd. Positionen sind das etwa 2·10¹⁶ Vergleiche, jeder mit bis zu 36 Basen. Selbst bei einer Milliarde Vergleichen pro Sekunde wären das über 200 Tage. Man braucht also einen Filter, der fast alle Positionen sofort ausschließt. Das leistet Seed-and-Extend.

---

## 4. Seed-and-Extend

### 4.1 Die Grundidee

Seed-and-Extend arbeitet in zwei Schritten:

1. **Seed:** Finde kurze Teilstücke, die exakt in Read und Referenz vorkommen. Exakte Suche ist mit einer Hashtabelle sehr schnell.
2. **Extend:** Nur an diesen Ankerstellen prüft man mit einem genaueren (langsameren) Verfahren, ob sich der ganze Read mit höchstens k Unterschieden ausrichten lässt.

Alle Regionen ohne gemeinsamen Seed werden gar nicht erst angeschaut. Das ist erlaubt, weil man beweisen kann, dass dort kein gutes Alignment liegt (Abschnitt 4.3). BLAST, SOAP, MAQ, RMAP und ZOOM arbeiten alle nach diesem Prinzip (Paper 1).

### 4.2 k-mere und Hashtabellen

Ein k-mer ist ein Teilstring der Länge k. Ein String der Länge L hat L − k + 1 überlappende k-mere:

```
Referenz   CTGACCTGGCAT
4-mere     CTGA  TGAC  GACC  ACCT  CCTG  CTGG  TGGC  GGCA  GCAT
```

Legt man alle k-mere in eine Hashtabelle (Schlüssel: k-mer, Wert: Liste der Positionen), findet man zu jedem k-mer in erwarteter Zeit O(1) alle Vorkommen. Man kann entweder die Referenz indexieren (so macht es BLAST) oder die Reads (so macht es RMAP: Hashtabelle über die Read-Seeds, dann ein Durchlauf über die Referenz).

### 4.3 Das Schubfachprinzip: warum kein Treffer verloren geht

Das zentrale Argument des Papers (Paper 1.2, nach Baeza-Yates et al. 1992):

> Ein Alignment eines Reads der Länge m mit höchstens k Unterschieden enthält mindestens einen exakt übereinstimmenden Abschnitt der Länge ⌊m/(k+1)⌋.

**Beweis.** Teile den Read in k+1 nicht überlappende Blöcke der Länge s = ⌊m/(k+1)⌋. Jeder Unterschied (Mismatch oder Indel) liegt in höchstens einem Block. Bei k Unterschieden sind also höchstens k Blöcke betroffen. Es gibt aber k+1 Blöcke, also bleibt mindestens einer fehlerfrei. Dieser Block kommt exakt in der Referenz vor, genau an der Stelle des Alignments. Das ist das Schubfachprinzip: k Fehler können nicht k+1 Fächer belegen.

Beispiel aus dem Paper: 36 bp, k = 4 ergibt s = ⌊36/5⌋ = 7.

```
Read (36 bp):  [ B1 ][ B2 ][ B3 ][ B4 ][ B5 ]+1 Base Rest
               7 bp  7 bp  7 bp  7 bp  7 bp
Fehler:          x     x           x     x
                             ↑
                B3 ist fehlerfrei und kommt exakt in der Referenz vor
```

Ein zweites Beispiel aus dem Paper: Ein 30-bp-Read mit höchstens einem Unterschied enthält immer 15 aufeinanderfolgende, exakt passende Basen, egal wo der Unterschied liegt.

Folge: Wenn man für jeden der k+1 Blöcke alle exakten Vorkommen in der Referenz sucht und jedes davon verlängert, findet man garantiert jedes Alignment mit höchstens k Unterschieden. Das Verfahren ist voll sensitiv.

### 4.4 Der Preis kurzer Seeds

Je größer k, desto kürzer wird s, und kurze Seeds kommen sehr oft zufällig vor. Für einen zufälligen Seed der Länge s in einer zufälligen Referenz der Länge L ist die erwartete Zahl der Vorkommen (Paper 3):

```
E = (L − s + 1) / 4^s
```

Jede Position passt mit Wahrscheinlichkeit (1/4)^s, und es gibt L − s + 1 Positionen. Für 36-bp-Reads gegen das ganze Genom (eigene Rechnung mit der Formel des Papers):

| k | s | Seeds pro Read (beide Stränge) | erwartete Zufallstreffer je Seed im Genom |
| --- | --- | --- | --- |
| 0 | 36 | 2 | praktisch 0 |
| 1 | 18 | 4 | 0,04 |
| 2 | 12 | 6 | rund 170 |
| 3 | 9 | 8 | rund 11.000 |
| 4 | 7 | 10 | rund 175.000 |

Jeder dieser Zufallstreffer muss im Extend-Schritt geprüft werden, obwohl fast alle scheitern. Darum wächst die Laufzeit mit k so stark. Zwei Dinge kommen zusammen: Kürzere Seeds treffen exponentiell öfter, und jeder Read liefert mehr Seeds.

Hinweis: Im Paper steht für das 7-mer im ganzen Genom „> 17 500“. Nach der Formel des Papers sind es rund 175.000, dort fehlt also eine Null. Die Werte für Chromosom 1 (> 15.000) und 22 (> 3.000) stimmen.

Echte Genome sind ungünstiger als diese Zufallsrechnung, weil sie viele Wiederholungen enthalten. Schon bei k = 0 fand CloudBurst für 7 Mio. Reads 771 Mio. perfekte Treffer im Genom, im Schnitt also über 100 pro Read.

### 4.5 Der Extend-Schritt

Wie verlängert man einen Seed? Es gibt zwei Fälle.

**Nur Mismatches.** Man legt den Read an der Stelle an, die der Seed vorgibt (Startposition = Position in der Referenz − Offset im Read), und vergleicht Base für Base. Sobald mehr als k Unterschiede gezählt sind, kann man abbrechen. Kosten: O(m).

**Mit Indels.** Durch Einfügungen und Löschungen verschiebt sich die Ausrichtung, ein einfacher Vergleich reicht nicht. Man braucht DP wie in 3.3. CloudBurst nutzt dafür eine Variante des Algorithmus von **Landau und Vishkin** (1986).

### 4.6 Landau-Vishkin: Alignment mit höchstens k Unterschieden in O(km)

Die volle DP-Tabelle aus 3.3 kostet O(m²). Wenn man aber ohnehin nur Alignments mit höchstens k Unterschieden will, ist der größte Teil der Tabelle uninteressant.

**Beobachtung 1: Nur ein schmales Band zählt.** Nummeriere die Diagonalen der Tabelle mit d = j − i. Ein Schritt nach rechts oder unten (Indel) wechselt die Diagonale um eins und kostet einen Fehler. Ein Alignment mit höchstens k Fehlern kann sich also höchstens k Diagonalen von der Hauptdiagonale entfernen. Relevant sind nur die 2k+1 Diagonalen von −k bis +k.

```
Band für k = 1 (nur die Zellen mit |i − j| ≤ 1 werden gebraucht):

         -  G  A  A  T  C
    -    ■  ■  .  .  .  .
    G    ■  ■  ■  .  .  .
    A    .  ■  ■  ■  .  .
    T    .  .  ■  ■  ■  .
    C    .  .  .  ■  ■  ■
```

**Beobachtung 2: Passende Basen kosten nichts.** Entlang einer Diagonale bleibt die Fehlerzahl gleich, solange die Basen übereinstimmen. Man muss diese Zellen nicht einzeln speichern, man kann darüber „hinwegrutschen“.

Landau-Vishkin speichert deshalb nur einen Wert pro Diagonale d und Fehlerzahl e:

```
L(d, e) = die weiteste Zeile, die man auf Diagonale d mit höchstens e Fehlern erreicht
```

Für e = 0, 1, …, k berechnet man L(d, e) aus den drei Möglichkeiten für den e-ten Fehler (Mismatch auf derselben Diagonale, Indel von einer Nachbardiagonale) und rutscht dann so weit entlang der Diagonale, wie die Basen übereinstimmen. Erreicht eine Diagonale das Ende des Reads, ist ein Alignment mit e Fehlern gefunden.

Laufzeit: Es gibt (2k+1)·(k+1) Werte L(d, e). Das Rutschen auf einer Diagonale bewegt sich nur vorwärts, insgesamt also höchstens m Schritte pro Diagonale. Zusammen ergibt das O(km) statt O(m²). Bei k = 4 und m = 36 ist das ein großer Unterschied. Details stehen in Gusfields Lehrbuch (1997), auf das auch das Paper verweist.

### 4.7 Andere Ansätze, die das Paper erwähnt

- **Spaced Seeds** (SOAP, MAQ, ZOOM): Der Seed besteht nicht aus aufeinanderfolgenden Basen, sondern folgt einem Muster wie `1101011`, bei dem nur die 1-Stellen übereinstimmen müssen. So sind längere Seeds bei gleicher Sensitivität möglich, man braucht aber eventuell mehrere Muster.
- **Suffixbäume:** Datenstruktur für schnelle exakte Suche, als Seed-Quelle für längere ungenaue Alignments.
- **BWT-Aligner** (Bowtie, BWA, SOAP2): Sie nutzen die Burrows-Wheeler-Transformation, einen komprimierten Index des Genoms, und finden damit exakte Treffer sehr schnell. Mismatches werden durch Backtracking erlaubt. Sie sind extrem schnell, solange man wenige Fehler und wenige Treffer pro Read verlangt.

---

## 5. MapReduce, Hadoop und die Cloud

### 5.1 Warum ein Framework?

Wer ein Programm auf hunderten Rechnern laufen lassen will, muss sich normalerweise um vieles selbst kümmern: Daten verteilen, Rechner koordinieren, Nachrichten austauschen, ausgefallene Rechner erkennen und deren Arbeit neu verteilen. Andere Frameworks für paralleles Rechnen verlangen, dass Entwickler die Kommunikation zwischen Prozessen selbst organisieren (Paper 1.1).

MapReduce, 2004 von Google vorgestellt, nimmt einem all das ab. Man schreibt nur zwei Funktionen, `map` und `reduce`. Das Framework führt sie automatisch parallel auf beliebig vielen Rechnern aus. Laut Paper liefen bei Google täglich tausende MapReduce-Programme auf Petabytes von Daten, und zwar auf gewöhnlicher, billiger Hardware („commodity hardware“).

### 5.2 Das Programmiermodell

Ein MapReduce-Job hat drei Phasen. Map und Reduce schreibt man selbst, der Shuffle dazwischen passiert automatisch.

```
map:     (Schlüssel₁, Wert₁)          → Liste von (Schlüssel₂, Wert₂)
shuffle: alle Paare mit gleichem Schlüssel₂ einsammeln
reduce:  (Schlüssel₂, Liste von Wert₂) → Liste von Ausgaben
```

**Map.** Die Eingabe wird automatisch in Stücke (Splits) geteilt. Jeder Mapper bearbeitet ein Stück und gibt beliebig viele Schlüssel-Wert-Paare aus, auch mehrere pro Eingabedatensatz. Mapper arbeiten unabhängig voneinander, also beliebig parallel.

**Shuffle.** Wenn alle Mapper fertig sind, sortiert und gruppiert das Framework alle Paare nach Schlüssel. Jeder Schlüssel landet bei genau einem Reducer, zusammen mit der Liste aller zugehörigen Werte. Das Paper beschreibt das so: Der Shuffle erzeugt im Grunde eine riesige verteilte Hashtabelle, mit dem Schlüssel als Index und einer Werteliste pro Schlüssel.

**Reduce.** Für jeden Schlüssel wird die Reduce-Funktion einmal mit der ganzen Werteliste aufgerufen. Sie kann beliebig komplex sein, darf aber nicht von der Reihenfolge der Werte abhängen, denn die ist nicht festgelegt. Verschiedene Schlüssel sind unabhängig voneinander, es können also so viele Reducer parallel laufen, wie es verschiedene Schlüssel gibt.

Fig. 1 im Paper zeigt dieses Schema mit zwei Mappern (m₁, m₂), den Schlüsseln k₁ … kₙ im Shuffle und zwei Reducern (r₁, r₂):

```
 Eingabe         Map             Shuffle                 Reduce

 ┌────────┐     ┌────┐        ┌───────────────┐
 │ Teil 1 │ ──► │ m1 │ ──┐    │ k1: [w, w, w] │ ──┐     ┌────┐
 └────────┘     └────┘   │    │ k2: [w]       │   ├───► │ r1 │ ──► Ausgabe 1
                         ├──► │ ...           │ ──┘     └────┘
 ┌────────┐     ┌────┐   │    │               │         ┌────┐
 │ Teil 2 │ ──► │ m2 │ ──┘    │ kn: [w, w]    │ ──────► │ r2 │ ──► Ausgabe 2
 └────────┘     └────┘        └───────────────┘         └────┘

 w = ein Wert. Jeder Mapper kann Paare für jeden Schlüssel liefern,
 der Shuffle sammelt sie über alle Mapper hinweg ein.
```

### 5.3 Beispiel: k-mere zählen

Das Paper erklärt MapReduce am Zählen aller k-mere in DNA-Sequenzen. Hier mit 3-meren und zwei Sequenzen:

- Map: Für jedes 3-mer gib (3-mer, 1) aus.
- Shuffle: Sammle alle Einsen pro 3-mer.
- Reduce: Summiere die Liste.

```
Eingabe          Map-Ausgabe                                Shuffle             Reduce
Seq 1 ACGACGT → (ACG,1) (CGA,1) (GAC,1) (ACG,1) (CGT,1)     ACG: [1, 1, 1]  →  ACG 3
Seq 2 GACGT   → (GAC,1) (ACG,1) (CGT,1)                     CGA: [1]        →  CGA 1
                                                            CGT: [1, 1]     →  CGT 2
                                                            GAC: [1, 1]     →  GAC 2
```

Das Muster ist dasselbe wie in CloudBurst: Map zerlegt Sequenzen in k-mere und benutzt sie als Schlüssel. Der Shuffle bringt gleiche k-mere zusammen, egal aus welcher Sequenz und von welchem Rechner sie stammen.

### 5.4 Combiner

Ein **Combiner** ist eine Art Mini-Reduce, das direkt nach dem Map auf demselben Rechner im Speicher läuft und nur die Paare dieses einen Mappers sieht. Im Beispiel gibt Mapper 1 zweimal (ACG,1) aus. Ein Combiner fasst das zu (ACG,2) zusammen, und das Reduce summiert dann Teilsummen statt Einsen.

```
ohne Combiner:  m₁ schickt (ACG,1) (CGA,1) (GAC,1) (ACG,1) (CGT,1)   → 5 Paare übers Netz
mit Combiner:   m₁ schickt (ACG,2) (CGA,1) (GAC,1) (CGT,1)           → 4 Paare übers Netz
```

Das spart Daten im Shuffle, der oft der teuerste Teil ist. Combiner gehen nicht immer. Die Operation muss auch auf einer Teilmenge der Werte sinnvoll sein (Summe, Maximum, „die besten zwei“). CloudBurst nutzt einen Combiner in Job 2 (Abschnitt 6.10).

### 5.5 Partitionierung und Load-Balancing

Welcher Schlüssel geht an welchen Reducer? Standardmäßig entscheidet eine Hashfunktion: Reducer = hash(Schlüssel) mod N. So bekommt jeder Reducer etwa 1/N aller Schlüssel.

Das verteilt die Arbeit nur dann gleichmäßig, wenn alle Schlüssel etwa gleich viel Arbeit machen. Gibt es einzelne Schlüssel mit sehr vielen Werten („heiße Schlüssel“), muss deren Reducer viel länger arbeiten als alle anderen, und der ganze Job wartet auf ihn. Das Paper sagt: Die Gesamtlaufzeit ist durch die längste Einzelaufgabe begrenzt. Man kann dann eine eigene Partitionsfunktion schreiben oder ändern, wie die Schlüssel ausgegeben werden. CloudBurst macht das Zweite (Abschnitt 6.5).

Ein weiterer Trick: Man startet mehr Aufgaben (Tasks), als es Kerne gibt. Kleine Tasks füllen die Lücken, wenn andere länger dauern. Im Paper laufen auf 24 Kernen 240 Mapper und 48 Reducer.

### 5.6 Ausführung im Cluster, Fehlertoleranz und Datenlokalität

Ein MapReduce-Cluster hat einen Master, der die Arbeit verteilt, und viele Worker, die Map- und Reduce-Tasks ausführen. Bei Hadoop 0.x hießen diese Dienste JobTracker und TaskTracker.

- **Fehlertoleranz:** Fällt ein Worker aus, startet der Master dessen Tasks auf einem anderen Rechner neu. Weil Map und Reduce keine Nebenwirkungen haben sollen, ist das gefahrlos. Das Paper zählt „monitoring and restart“ zu den Vorteilen gegenüber selbstgebauter Parallelisierung.
- **Datenlokalität:** Daten über das Netz zu schicken ist teuer. MapReduce versucht deshalb, einen Map-Task auf dem Rechner zu starten, auf dessen Festplatte das zugehörige Datenstück schon liegt („data aware scheduling“).
- **Zwischenergebnisse auf Platte:** MapReduce ist für Datenmengen gebaut, die weit über den Hauptspeicher hinausgehen. Zwischenergebnisse und der Datenaustausch zwischen Map und Reduce laufen deshalb über Dateien. Das ist robust, aber der Shuffle kann zum Flaschenhals werden.

### 5.7 Verteiltes Dateisystem: GFS und HDFS

Damit der Datenaustausch über Dateien nicht zum Engpass wird, entwickelte Google das Google File System (GFS). Dessen Open-Source-Gegenstück ist HDFS (Hadoop Distributed File System).

- Dateien werden in große Blöcke geteilt, standardmäßig 64 MB.
- Jeder Block wird auf mehrere Rechner kopiert, standardmäßig dreimal.
- Weil viele Festplatten gleichzeitig lesen, ist der Gesamtdurchsatz viel höher als der einer einzelnen Platte.
- Fällt eine billige Platte aus, gibt es noch Kopien.

HDFS ist für große Dateien gedacht, die einmal geschrieben und oft vollständig gelesen werden. Genau so werden Referenz und Reads in CloudBurst genutzt.

### 5.8 Hadoop

Hadoop ist die Open-Source-Implementierung von MapReduce und HDFS, geschrieben in Java. Laut Paper wurde es von Amazon, Yahoo, Google, IBM und anderen unterstützt. Es lief auf Produktionsclustern mit über 10.000 Knoten, und ein Hadoop-Cluster aus 910 Rechnern stellte damals einen Rekord auf, als er 1 TB Daten in 209 Sekunden sortierte. Wie beim Original schreibt man nur die Map- und Reduce-Funktion, Hadoop erledigt den Rest.

Daten speichert Hadoop gern in **SequenceFiles**, einem binären Format für Schlüssel-Wert-Paare. CloudBurst wandelt die FASTA-Dateien vor dem ersten Lauf in dieses Format um.

### 5.9 Cloud Computing und Amazon EC2

Cloud Computing heißt: Man mietet Rechenleistung bei Bedarf über das Internet, statt eigene Rechner zu kaufen. Man zahlt pro Stunde und Maschine und kann für eine große Aufgabe kurzfristig viele Rechner dazunehmen.

Amazons Elastic Compute Cloud (EC2) bot 2009 laut Paper fünf Klassen virtueller Maschinen für 0,10 bis 0,80 $ pro Stunde. Es gab fertige Images und Skripte, um einen Hadoop-Cluster zu starten. Danach kopiert man seine Daten ins HDFS des neuen Clusters und arbeitet wie auf einem eigenen Cluster. Ein Nachteil, den das Paper nennt: Große Datenmengen erst einmal in die Cloud hochzuladen kann lange dauern. Amazon spiegelte deshalb bereits Teile von Ensembl und GenBank, zwei großen Sequenzdatenbanken, innerhalb von EC2.

### 5.10 Speedup, Effizienz und das Gesetz von Amdahl

**Speedup** ist das Verhältnis aus serieller und paralleler Laufzeit:

```
Speedup S(N) = T(1) / T(N)          Effizienz = S(N) / N
```

Ideal ist S(N) = N: Mit 10 Kernen dauert es ein Zehntel der Zeit. Das Paper nennt zwei Gründe, warum das in der Praxis selten klappt.

1. **Serieller Anteil (Amdahl).** Ist ein Anteil f der Arbeit nicht parallelisierbar, gilt

   ```
   S(N) = 1 / ( f + (1 − f)/N )   ≤   1/f
   ```

   Bei 10 % seriellem Anteil (f = 0,1) ist der Speedup also höchstens 10, egal wie viele Kerne man einsetzt.

2. **Ungleiche Lastverteilung.** Die Gesamtlaufzeit ist die Laufzeit der längsten Einzelaufgabe. Wenn ein Reducer doppelt so viel Arbeit hat wie die anderen, warten alle auf ihn.

Ein Speedup über N (superlinear) ist möglich, wenn die parallele Version Vorteile hat, die die serielle nicht hat, etwa mehr Cache und Hauptspeicher insgesamt oder eine schnellere Implementierung. Das kommt in Kapitel 7 vor.

---

## 6. Der CloudBurst-Algorithmus

### 6.1 Die Kernidee: Seeds werden zu Schlüsseln

Seed-and-Extend braucht eine Hashtabelle, in der man zu einem Seed alle Vorkommen findet. MapReduce baut genau so eine Tabelle im Shuffle, nur verteilt über viele Rechner. CloudBurst nutzt das direkt:

| Seed-and-Extend | MapReduce in CloudBurst |
| --- | --- |
| Seeds aus Reads und Referenz erzeugen | Map |
| Hashtabelle: gleiche Seeds zusammenbringen | Shuffle |
| Seeds zu Alignments verlängern | Reduce |
| bestes Alignment pro Read auswählen | zweiter MapReduce-Job |

Kein einzelner Rechner muss den ganzen Index im Speicher halten, und Referenz und Reads werden je nur einmal gelesen (Paper 5).

### 6.2 Überblick über den Ablauf

```
   Reads (Multi-FASTA)          Referenz (Multi-FASTA)
            │                             │
            └────────────┬────────────────┘
                         ▼
      Umwandlung in SequenceFiles, Kopieren ins HDFS
                         │
  ═══════════ Job 1: Alignment ═══════════════════════════════════════
                         ▼
   Map      Referenz: jedes s-mer als Seed
            Reads:    k+1 nicht überlappende Blöcke, beide Stränge
                         ▼
   Shuffle  gruppiert alle Einträge mit gleichem Seed
                         ▼
   Reduce   prüft jedes Paar (Referenzstelle, Read), verlängert den Seed,
            behält Alignments mit ≤ k Unterschieden, verwirft Duplikate
                         ▼
            alle Alignments aller Reads
                         │
  ═══════════ Job 2: Filterung (optional) ════════════════════════════
                         ▼
   Map      Schlüssel = Read-ID
   Shuffle  alle Alignments eines Reads zusammen
   Reduce   eindeutig bestes Alignment behalten
                         ▼
            ein Alignment pro Read (wie RMAPM)
```

Fig. 2 im Paper zeigt Job 1 grafisch. Dort werden aus einer Referenz und zwei Reads Seeds erzeugt. Ein grauer Seed kommt zweimal in der Referenz und einmal in einem Read vor. Der Reducer prüft beide Paare und findet ein Alignment mit zwei Fehlern und eins mit null Fehlern. Ein schwarzer Seed wird zu einem Alignment mit drei Fehlern verlängert.

### 6.3 Eingabe und Vorbereitung (Paper 2)

- Eingabe sind zwei Multi-FASTA-Dateien: eine mit den Reads, eine mit einer oder mehreren Referenzsequenzen.
- Beide werden in Hadoop-SequenceFiles umgewandelt und ins HDFS kopiert. Jede Sequenz wird als Paar (id, SeqInfo) gespeichert, wobei SeqInfo = (sequence, start_offset).
- Die Referenz wird in Stücke von 65 kb geteilt, die sich um 1 kb überlappen. So kann jeder Mapper ein Stück unabhängig bearbeiten, und start_offset sagt, wo das Stück in der ganzen Sequenz beginnt. Die Überlappung sorgt dafür, dass ein Alignment, das über eine Stückgrenze reicht, trotzdem vollständig in mindestens einem Stück liegt. Für Reads über 1 kb kann man die Überlappung vergrößern.
- Aus der kürzesten Read-Länge m und dem Fehlerlimit k wird die Seed-Länge berechnet:

  ```
  s = ⌊ m / (k+1) ⌋
  ```

  Für m = 36: k = 0 → 36, k = 1 → 18, k = 2 → 12, k = 3 → 9, k = 4 → 7.

### 6.4 Map: Seeds erzeugen (Paper 2.1)

Die Map-Funktion liest Sequenzen und gibt Paare (Seed, MerInfo) aus. Der Seed ist der Schlüssel. MerInfo ist ein Tupel mit allem, was der Reducer später braucht:

| Feld | Bedeutung |
| --- | --- |
| id | ID der Sequenz (welcher Read bzw. welche Referenz) |
| position | Position des Seeds: in der Referenz absolut, im Read der Offset |
| isRef | 1 für Referenz, 0 für Read |
| isRC | 1, wenn der Seed aus dem Reverse Complement des Reads stammt |
| left_flank | Basen links vom Seed, bis zu m − s + k Stück |
| right_flank | Basen rechts vom Seed, bis zu m − s + k Stück |

Referenz und Reads werden unterschiedlich behandelt.

**Referenz:** Für jede Position wird das dort beginnende s-mer ausgegeben, also alle überlappenden s-mere. Dabei ist isRef = 1 und isRC = 0.

**Reads:** Es werden nur die nicht überlappenden s-mere ausgegeben, also die Blöcke aus dem Schubfach-Argument. Dasselbe passiert noch einmal für das Reverse Complement des Reads, mit isRC = 1.

Warum diese Asymmetrie?

- Bei der Referenz weiß man nicht, wo ein Read hingehört. Der fehlerfreie Block eines Reads kann an jeder beliebigen Genomposition liegen. Deshalb müssen alle Positionen als Seed verfügbar sein.
- Beim Read reichen nach dem Schubfachprinzip die k+1 festen Blöcke. Mindestens einer davon ist fehlerfrei. Mehr Seeds würden nur mehr doppelte Kandidaten erzeugen.
- Das Reverse Complement bildet man von den Reads und nicht von der Referenz. Das ist viel billiger, als die riesige Referenz zweimal auszugeben.

Warum die Flanken? Der Reducer bekommt nur die Werte zu einem Seed, aber nicht die ganzen Sequenzen. Um den Seed zu einem Alignment des ganzen Reads zu verlängern, braucht er die Basen links und rechts davon. Ein Read hat neben dem Seed höchstens m − s weitere Basen. Weil Indels das Alignment verschieben können, braucht die Referenz bis zu k Basen mehr. Daher die Flankenlänge m − s + k. Die Flanken machen MerInfo größer, aber der Reducer kann dadurch völlig unabhängig arbeiten, ohne auf andere Daten zuzugreifen.

**Kodierung.** Seeds werden mit 2 Bit pro Base gespeichert (4 Zeichen A, C, G, T). Flanken nutzen 4 Bit pro Base, weil dort zusätzlich N für unbekannte Basen und ein Trennzeichen „.“ vorkommen können. Kompakte Kodierung spart Daten im Shuffle.

### 6.5 Load-Balancing für häufige Seeds (Paper 2.1)

Mit dem Seed als Schlüssel bekommt jeder Reducer etwa 1/N der 4^s möglichen Seeds. Meistens ist das gut verteilt, weil die meisten Seeds ähnlich oft vorkommen.

Eine Ausnahme sind **Low-Complexity-Seeds**, die nur aus einer Base bestehen, etwa AAAAAAA oder TTTTTTT. Solche Abschnitte gibt es im Genom und in den Reads überproportional oft. Ein Reducer, der AAAAAAA bekommt, muss |R| × |Q| Paare prüfen, also sehr viele Referenzstellen mal sehr viele Reads. Im Experiment brauchten diese Reducer über eine Stunde, alle anderen unter einer Minute (Paper 3). Der ganze Job hätte auf sie gewartet.

CloudBursts Lösung ist ein Redundanz-Trick mit Parameter r:

1. Jedes Vorkommen eines Low-Complexity-Seeds in der **Referenz** wird r-mal ausgegeben, mit den Schlüsseln AAAA-0, AAAA-1, …, AAAA-(r−1).
2. Jedes Vorkommen in einem **Read** wird nur einmal ausgegeben, an eine zufällig gewählte Kopie AAAA-R mit 0 ≤ R < r.

```
ohne Redundanz:                     mit Redundanz r = 4:

Reducer für AAAA:                   AAAA-0: alle Ref-Stellen × ¼ der Reads
  alle Ref-Stellen × alle Reads     AAAA-1: alle Ref-Stellen × ¼ der Reads
  (ein Reducer, sehr lange)         AAAA-2: alle Ref-Stellen × ¼ der Reads
                                    AAAA-3: alle Ref-Stellen × ¼ der Reads
                                    (vier Reducer, parallel)
```

Jeder Read trifft weiterhin auf alle Referenzstellen des Seeds, denn jede Kopie enthält die vollständige Referenzseite. Jedes Paar wird also genau einmal geprüft, nicht öfter und nicht seltener. Die Gesamtarbeit bleibt gleich, sie wird nur auf r Reducer verteilt. Der Preis ist, dass die Referenzseite dieser Seeds r-mal durch den Shuffle geht.

Im Paper wurde r = 16 auf dem lokalen Cluster verwendet, auf EC2 bis zu 72.

### 6.6 Shuffle: gemeinsame Seeds sammeln (Paper 2.2)

Wenn alle Mapper fertig sind, gruppiert Hadoop alle Paare nach Seed. Ein Reducer bekommt dann z. B. für den Seed GACC alle Referenzstellen, an denen GACC steht, und alle Read-Blöcke, die GACC lauten. Damit ist katalogisiert, welche Seeds Reads und Referenz gemeinsam haben. Seeds, die nur auf einer Seite vorkommen, erzeugen keine Arbeit.

Man kann sich den Shuffle als verteilten Hash-Join zwischen der Seed-Tabelle der Referenz und der Seed-Tabelle der Reads vorstellen.

### 6.7 Reduce: Seeds verlängern (Paper 2.3)

Für einen Seed und seine MerInfo-Liste macht der Reducer Folgendes:

1. Teile die Liste in die Menge R (Einträge aus der Referenz) und die Menge Q (Einträge aus den Reads).
2. Prüfe jedes Paar aus dem kartesischen Produkt R × Q. Der Read wird so an die Referenz gelegt, dass die Seeds übereinanderliegen: Startposition = Referenzposition − Offset im Read.
3. Verlängere mit den Flanken:
   - Mismatch-Modus: linke und rechte Flanke Base für Base vergleichen und Mismatches zählen.
   - Indel-Modus: Landau-Vishkin auf beiden Seiten des Seeds.
4. Hat das Alignment höchstens k Unterschiede, wird geprüft, ob es ein Duplikat ist (6.8). Wenn nicht, wird es ausgegeben.

Die Paare werden blockweise über Teilmengen von R und Q abgearbeitet, damit die Daten möglichst lange im Cache bleiben.

### 6.8 Duplikate erkennen (Paper 2.3)

Ein Alignment kann mehrere exakt passende Blöcke enthalten. Ein perfekt passender Read hat sogar k+1 fehlerfreie Blöcke und würde k+1-mal gefunden, in k+1 verschiedenen Reducern. Die Ausgabe soll aber jedes Alignment nur einmal enthalten.

Die Regel ist einfach: Ein Alignment wird nur von dem Seed mit dem kleinsten Offset im Read behalten. Wenn ein Reducer ein Alignment findet und sieht, dass ein Block mit kleinerem Offset in diesem Alignment ebenfalls exakt passt, verwirft er sein Ergebnis. Diesen Block hat dann ein anderer Reducer gefunden.

Wichtig dabei: Jeder Reducer kann das allein anhand der Flanken entscheiden, ohne Kommunikation mit anderen Reducern. Weil k klein ist, werden nur wenige Alignments verworfen.

### 6.9 Ausgabe

Job 1 schreibt binäre Dateien mit jedem Alignment jedes Reads, das höchstens k Mismatches oder Unterschiede hat. Sie lassen sich in eine tabulatorgetrennte Textdatei im selben Format wie RMAP umwandeln.

### 6.10 Job 2: das beste Alignment pro Read (Paper 2.4)

Oft braucht man nicht alle Alignments, sondern pro Read nur das beste, also das mit den wenigsten Unterschieden. Wenn es mehrere gleich gute gibt, ist der Read nicht eindeutig zuzuordnen und es wird gar nichts gemeldet. So macht es auch RMAPM.

Das ist ein zweiter, kleiner MapReduce-Job:

- Map: Gib jedes Alignment mit der Read-ID als Schlüssel aus.
- Shuffle: Alle Alignments eines Reads landen beim selben Reducer.
- Reduce: Suche das beste Alignment. Ist es eindeutig, gib es aus, sonst nichts.

Zwei Optimierungen ändern das Ergebnis nicht:

1. Schon die Reducer aus Job 1 melden pro Read nur ihre zwei besten Alignments.
2. Job 2 nutzt einen Combiner, der lokal pro Read ebenfalls nur die zwei besten weiterreicht.

Warum genügen zwei? Um zu entscheiden, ob das beste Alignment eindeutig ist, muss man nur wissen, ob das zweitbeste genauso gut ist. Ein drittes ändert daran nichts. Deshalb ist „die besten zwei“ auch eine Operation, die sich als Combiner eignet: Die besten zwei aus Teilmengen enthalten immer die besten zwei der Gesamtmenge.

### 6.11 Durchgerechnetes Beispiel

Das Beispiel ist klein genug, um es von Hand nachzurechnen. Es ist dasselbe wie im Vortragsplan.

**Parameter:** Reads der Länge m = 8, erlaubt ist k = 1 Mismatch. Also s = ⌊8/2⌋ = 4, jeder Read hat 2 Blöcke. Die Flanken haben bis zu m − s + k = 5 Basen.

```
Ref-Position   0 1 2 3 4 5 6 7 8 9 10 11
Referenz       C T G A C C T G G C A  T
Read 1             G A C C A G G C           Mismatch bei Ref-Pos. 6 (Read A, Referenz T)
Read 2             G T C C T G G C           Mismatch bei Ref-Pos. 3 (Read T, Referenz A)
```

Beide Reads gehören an Position 2.

**Map, Referenz:** alle 9 überlappenden 4-mere.

| Seed | Position | linke Flanke | rechte Flanke |
| --- | --- | --- | --- |
| CTGA | 0 | – | CCTGG |
| TGAC | 1 | C | CTGGC |
| GACC | 2 | CT | TGGCA |
| ACCT | 3 | CTG | GGCAT |
| CCTG | 4 | CTGA | GCAT |
| CTGG | 5 | CTGAC | CAT |
| TGGC | 6 | TGACC | AT |
| GGCA | 7 | GACCT | T |
| GCAT | 8 | ACCTG | – |

**Map, Reads:** je 2 Blöcke pro Strang.

| Read | Strang | Block (Seed) | Offset | linke Flanke | rechte Flanke |
| --- | --- | --- | --- | --- | --- |
| Read 1 | vorwärts | GACC | 0 | – | AGGC |
| Read 1 | vorwärts | AGGC | 4 | GACC | – |
| Read 1 | RC (GCCTGGTC) | GCCT | 0 | – | GGTC |
| Read 1 | RC | GGTC | 4 | GCCT | – |
| Read 2 | vorwärts | GTCC | 0 | – | TGGC |
| Read 2 | vorwärts | TGGC | 4 | GTCC | – |
| Read 2 | RC (GCCAGGAC) | GCCA | 0 | – | GGAC |
| Read 2 | RC | GGAC | 4 | GCCA | – |

**Shuffle und Reduce:**

| Seed | R (Referenz) | Q (Reads) | Was der Reducer tut |
| --- | --- | --- | --- |
| GACC | Pos. 2 | Read 1, Offset 0 | Start 2 − 0 = 2. Links nichts zu prüfen. Rechts AGGC gegen TGGC: 1 Mismatch. Treffer mit 1 Fehler. Kein Block mit kleinerem Offset, also kein Duplikat. Ausgabe. |
| TGGC | Pos. 6 | Read 2, Offset 4 | Start 6 − 4 = 2. Links GTCC gegen GACC (die letzten 4 Basen der Flanke TGACC): 1 Mismatch. Rechts nichts. Treffer mit 1 Fehler. Block mit Offset 0 (GTCC) passt nicht exakt, also kein Duplikat. Ausgabe. |
| AGGC, GTCC | – | je ein Read | Seed kommt nicht in der Referenz vor. Keine Arbeit. |
| GCCT, GGTC, GCCA, GGAC | – | je ein Read (RC) | Seed kommt nicht in der Referenz vor. Keine Arbeit. |
| alle übrigen Referenz-Seeds | je 1 Position | – | kein Read hat diesen Seed. Keine Arbeit. |

Read 1 wird über seinen ersten Block gefunden, Read 2 über den zweiten. Genau das garantiert das Schubfachprinzip: Der eine Fehler kann nur einen der beiden Blöcke treffen.

**Duplikat-Fall.** Ein fehlerfreier Read GACCTGGC hätte die Blöcke GACC (Offset 0) und TGGC (Offset 4). Beide passen exakt, also finden die Reducer für GACC und für TGGC dasselbe Alignment an Position 2. Der Reducer für TGGC sieht in der linken Flanke, dass auch der Block mit Offset 0 exakt passt, und verwirft sein Ergebnis. Übrig bleibt genau eine Ausgabe.

### 6.12 Vereinfachter Pseudocode

Der folgende Pseudocode fasst Job 1 zusammen. Load-Balancing, Kodierung und Blockverarbeitung sind weggelassen. Es ist eine eigene Zusammenfassung, kein Code aus dem Paper.

```
map(id, (seq, start)):
    if seq ist Referenz:
        for p in 0 .. len(seq) − s:
            emit( seq[p : p+s],
                  MerInfo(id, start + p, isRef=1, isRC=0, linke/rechte Flanke) )
    else:  # Read
        for (isRC, r) in [(0, seq), (1, reverse_complement(seq))]:
            for p in 0, s, 2s, … solange p + s ≤ len(r):     # nicht überlappende Blöcke
                emit( r[p : p+s],
                      MerInfo(id, p, isRef=0, isRC, linke/rechte Flanke) )

reduce(seed, werte):
    R = [w for w in werte if w.isRef]
    Q = [w for w in werte if not w.isRef]
    for ref in R:
        for read in Q:
            aln = verlaengere(ref, read)       # Mismatch-Scan oder Landau-Vishkin
            if aln.fehler ≤ k and not duplikat(aln, read.position):
                emit(read.id, aln)
```

---

## 7. Experimente und Ergebnisse

### 7.1 Versuchsaufbau (Paper 3)

- **Daten:** zufällige Teilmengen von 7,06 Mio. öffentlich verfügbaren Illumina/Solexa-Reads aus dem 1000 Genomes Project (SRR001113). Alle Reads sind genau 36 bp lang.
- **Referenzen:** ganzes menschliches Genom (NCBI Build 36, 2,87 Gbp), Chromosom 1 (247,2 Mbp) und Chromosom 22 (49,7 Mbp).
- **Fehler:** bis zu 4 Mismatches. Der Indel-Modus wurde nicht gemessen.
- **Cluster:** 12 Knoten mit je einem 32-bit Dual-Core Intel Xeon (3,2 GHz) und 250 GB Platte, zusammen 24 Kerne. Hadoop 0.15.3 mit zwei Tasks pro Knoten.
- **Einstellungen:** 240 Mapper, 48 Reducer, Redundanz 16 für Low-Complexity-Seeds.
- **Messung:** Mittel aus drei Läufen. Die Zeit zum Umwandeln und Hochladen ins HDFS ist nicht enthalten, weil sie für alle Läufe gleich war und die Daten mehrfach genutzt wurden.

### 7.2 Skalierung mit Readzahl und Sensitivität (Fig. 3)

Fig. 3 zeigt die Laufzeit über der Zahl der Reads, je ein Diagramm für Genom, Chromosom 1 und Chromosom 22, und je eine Linie für k = 0 bis 4.

Ergebnisse:

- **Linear in der Zahl der Reads.** Doppelt so viele Reads brauchen etwa doppelt so lange. Das erwartet man, denn jeder Read erzeugt eine feste Zahl von Seeds und im Mittel gleich viele Kandidaten.
- **Überlinear in k.** Mit mehr erlaubten Fehlern steigt die Laufzeit stark, weil die Seeds kürzer werden und exponentiell mehr Zufallstreffer haben (Abschnitt 4.4). Die meisten dieser Kandidaten scheitern beim Verlängern, kosten aber trotzdem Zeit.
- **Größe der Referenz.** Die y-Achsen reichen beim Genom bis etwa 150.000 s, bei Chromosom 1 bis etwa 15.000 s, bei Chromosom 22 bis etwa 3.000 s.
- **Grenzen.** Alle 7 Mio. Reads mit k = 4 gegen das ganze Genom liefen nicht durch. Nach rund 25 Mrd. gemeldeten Alignments waren die Festplatten voll. Selbst bei k = 0 entstanden 771 Mio. perfekte Treffer. Die meisten anderen Tools hätten nur einen pro Read gemeldet.

### 7.3 Vergleich mit RMAP (Fig. 4)

Verglichen wurde CloudBurst auf 24 Kernen mit RMAPM 0.41 auf einem Kern. RMAP braucht ein 64-bit-System und lief deshalb auf einem anderen Rechner: AMD Opteron 250, 2,4 GHz, 8 GB RAM. CloudBurst lief mit eingeschalteter Filterung (Job 2), damit die Ausgaben identisch sind.

Erwartet wäre ein Speedup von 24. Gemessen wurden 2× bis 33×, abhängig von k und Referenz. Aus Fig. 4 grob abgelesen:

- Chromosom 1 und 22: bei k = 0 nur etwa 2× bis 4×, bei k = 4 etwa 25× (Chr. 22) bis 33× (Chr. 1).
- Ganzes Genom: über alle k etwa 10× bis 12×, ohne klaren Anstieg.

Die Erklärungen des Papers:

- **Kleines k: Overhead dominiert.** Bei k = 0 gibt es pro Seed kaum Arbeit. Die Kosten, alle Referenz-Seeds auszugeben, über das Netz zu schicken und zu sortieren, überwiegen. RMAP schaut dagegen nur in einer Hashtabelle im Speicher nach. Das ist Amdahl in der Praxis: Der feste Anteil begrenzt den Speedup.
- **Großes k: Rechenarbeit dominiert.** Mit mehr Kandidaten wird der Overhead relativ kleiner, und die gut parallelisierbare Arbeit im Reduce bestimmt die Laufzeit.
- **Über 24×:** Das Paper führt das auf Implementierungsunterschiede zwischen RMAP und CloudBurst und auf die zusätzlichen Ressourcen im Cluster zurück (Cache, Platten-I/O, RAM).
- **Ganzes Genom bleibt flach:** Die Referenz ist so groß, dass der feste Aufwand für ihre Seeds auch bei großem k viel ausmacht. Das Paper empfiehlt, mehr Reads in einem Lauf zu kartieren, damit sich dieser Aufwand auf mehr Reads verteilt.

### 7.4 Vergleich mit einer Ad-hoc-Parallelisierung (Paper 3)

Naheliegende Alternative zu CloudBurst: die Reads in 24 Dateien aufteilen und RMAP 24-mal unabhängig starten. Getestet wurde das mit 24 Dateien à 294.000 Reads gegen Chromosom 22. Gemessen wurde nur die reine RMAP-Zeit, ohne Aufteilen, Starten und Überwachen. Der Speedup ergibt sich aus der langsamsten Datei.

| k | Laufzeit der 24 Teilläufe | Speedup |
| --- | --- | --- |
| 0 | 18–41 s | 12× |
| 1 | 26–67 s | 14× |
| 2 | 34–98 s | 16× |
| 3 | 132–290 s | 21× |
| 4 | 1379–1770 s | 29× |

Die Teilläufe dauern sehr unterschiedlich lang, je nachdem, welche Reads in einer Datei gelandet sind, und die langsamste bestimmt das Ergebnis. Der superlineare Wert bei k = 4 kommt laut Paper vermutlich daher, dass RMAP mit weniger Reads den Cache besser nutzt.

Das Paper wertet: Die Ad-hoc-Lösung erreicht ähnliche Speedups wie CloudBurst, kümmert sich aber nicht um Lastverteilung und ist zerbrechlicher. Ihr fehlen die Vorteile von Hadoop: datenbewusstes Scheduling, Überwachung und Neustart, das verteilte Dateisystem.

### 7.5 Amazon EC2: Instanztypen im Vergleich (Paper 4)

In der Cloud lassen sich Leistung und Größe des Clusters frei wählen. Der erste Test hielt die Kernzahl bei 24 fest und kartierte alle 7 Mio. Reads gegen Chromosom 22 mit k = 4.

| Konfiguration | Preis | Laufzeit |
| --- | --- | --- |
| 24 × „Small Instance“ (1 virtueller Kern, etwa ein 1,0–1,2-GHz-Xeon von 2007), Hadoop 0.17.0 | 0,10 $ pro Instanz und Stunde | 3805 s |
| 12 × „High-CPU Medium Instance“ (2 virtuelle Kerne, etwa 5-fache Leistung einer Small Instance), Hadoop 0.17.0 | 0,20 $ pro Instanz und Stunde | 1667 s |
| lokaler Cluster, 24 Kerne | – | 1921 s |

Beide EC2-Konfigurationen kosten 2,40 $ pro Stunde (eigene Rechnung: 24 × 0,10 $ = 12 × 0,20 $). Die High-CPU-Variante ist aber mehr als doppelt so schnell und damit pro Dollar deutlich besser. Sie war sogar schneller als der eigene Cluster.

### 7.6 Skalierung mit der Kernzahl (Fig. 5)

Der letzte Test ließ das Problem gleich (7 Mio. Reads, Chromosom 22, k = 4) und vergrößerte den Cluster aus High-CPU-Medium-Instanzen.

```
Kerne   Laufzeit (aus Fig. 5 abgelesen)
  24    ████████████████████████████  ca. 1670 s
  48    ███████████████               ca.  880 s
  72    ███████████                   ca.  660 s
  96    ████████                      ca.  480 s
```

- 96 Kerne waren 3,5-mal schneller als 24. Ideal wäre 4-mal, die Effizienz liegt also bei rund 88 %.
- Gegenüber RMAP auf einem Kern sank die Laufzeit von über 14 Stunden auf etwa 8 Minuten, ein Speedup über 100.
- Hauptgrund für die Abweichung vom Ideal: ungleiche Last. Einige wenige Reducer liefen länger als die anderen. Teilweise half es, mehr Reducer und mehr Redundanz einzustellen:

| Kerne | Reducer | Redundanz |
| --- | --- | --- |
| 24 | 48 | 16 |
| 48 | 60 | 24 |
| 72 | 144 | 72 |
| 96 | 196 | 72 |

**Amdahl-Abschätzung (eigene Rechnung).** Nimmt man an, dass ein Anteil f des 24-Kern-Laufs nicht mitskaliert, dann gilt bei 4-mal so vielen Kernen:

```
T₉₆ / T₂₄ = f + (1 − f)/4 = 1/3,5   ⇒   f ≈ 0,048
```

Rund 5 % der Laufzeit skalieren also nicht mit. Mit diesem f brächten 192 Kerne gegenüber 24 nur etwa 6× statt 8×, und mehr als etwa 21× ginge nie. Besseres Load-Balancing würde hier mehr bringen als mehr Hardware.

### 7.7 Was die Experimente zusammen zeigen

| Frage | Antwort aus den Experimenten |
| --- | --- |
| Skaliert CloudBurst mit der Datenmenge? | Ja, linear in der Zahl der Reads. |
| Skaliert CloudBurst mit der Rechnerzahl? | Fast linear, 3,5× bei 4× Kernen. |
| Lohnt sich das gegenüber RMAP? | Vor allem bei hoher Sensitivität: bis 33× auf 24 Kernen, über 100× auf 96. Bei k = 0 kaum. |
| Ist das Ergebnis dasselbe? | Ja, mit Filterung identisch zu RMAPM. |
| Braucht man einen eigenen Cluster? | Nein, gemietete EC2-Rechner waren sogar schneller. |

---

## 8. Diskussion und Einordnung

### 8.1 Was der Autor selbst folgert (Paper 5)

- CloudBurst macht hochsensitives Mapping großer Read-Mengen praktisch durchführbar. Es skaliert linear mit den Reads und fast linear mit der Clustergröße.
- Es ergänzt die BWT-Aligner, die besonders gut darin sind, schnell wenige Alignments pro Read zu liefern.
- Seed-and-Extend-Algorithmen passen natürlich zu MapReduce. Jeder hashtabellenbasierte Aligner wie BLAST, SOAP, MAQ oder ZOOM ließe sich so umsetzen. Auch BWT-Aligner könnten Hadoop und HDFS zum Parallelisieren nutzen.
- Hadoop bringt Skalierbarkeit, Redundanz, automatische Überwachung und Neustart sowie schnellen verteilten Dateizugriff mit. Kein Rechner braucht den ganzen Index im Speicher, und Referenz und Reads werden nur einmal gelesen.
- Geplant waren: Qualitätswerte nutzen, Paired-End-Reads besser unterstützen und CloudBurst in eine RNA-seq-Pipeline einbauen, die auch Spleißstellen modelliert.

### 8.2 Stärken

- **Volle Sensitivität mit Beweis.** Das Schubfachprinzip garantiert, dass kein Alignment mit höchstens k Unterschieden verloren geht. Das gilt auch für Indels.
- **Einfache, passende Abbildung.** Seeds als Schlüssel, Shuffle als Hash-Join. Die Idee ist leicht zu erklären und auf andere Verfahren übertragbar.
- **Lokale Entscheidungen.** Dank der Flanken arbeitet jeder Reducer unabhängig, auch beim Erkennen von Duplikaten.
- **Ehrliche Vergleichsbasis.** Die Ausgabe ist identisch zu RMAPM, man vergleicht also wirklich dieselbe Aufgabe.
- **Praktischer Nutzen.** Open Source, läuft auf gemieteter Hardware, Laufzeiten von Stunden auf Minuten reduziert.
- **Lastverteilung beachtet.** Der Redundanz-Trick löst das Problem häufiger Seeds, ohne das Ergebnis zu ändern.

### 8.3 Schwächen und offene Punkte (Einordnung)

- **Nur der Mismatch-Modus wurde gemessen.** Der Indel-Modus mit Landau-Vishkin ist beschrieben, aber nicht evaluiert.
- **Ungleiche Hardware im RMAP-Vergleich.** RMAP lief auf einem 2,4-GHz-Opteron (64 bit), CloudBurst auf 3,2-GHz-Xeons (32 bit). Speedups über 24× sind deshalb kein reiner Algorithmus-Effekt, wie das Paper auch selbst andeutet.
- **Vorbereitungszeit herausgerechnet.** Umwandeln und Hochladen ins HDFS zählen nicht zur Laufzeit. In der Cloud kann gerade der Upload lange dauern, wie das Paper in 1.1 selbst sagt.
- **Nur ein Datensatz.** 36-bp-Reads, single-end, ohne Qualitätswerte.
- **Riesige Ausgaben.** „Alle Treffer“ erzeugt bei k = 0 schon 771 Mio. Alignments, bei k = 4 scheiterte der Lauf am Speicherplatz.
- **Fester Aufwand für die Referenz.** Die ganze Referenz wird in jedem Lauf neu in Seeds zerlegt und durch den Shuffle geschickt, unabhängig davon, wie viele Reads es gibt. Bei der Größe des menschlichen Genoms sind das rund 2,9 Mrd. Schlüssel-Wert-Paare pro Lauf, jedes mit zwei Flanken. Das erklärt den kleinen Speedup bei k = 0 und auf dem ganzen Genom.
- **Kleiner Rechenfehler.** Ein zufälliges 7-mer kommt im Genom etwa 175.000-mal vor, nicht „> 17 500“-mal (siehe 4.4).

### 8.4 Wie es weiterging (Einordnung, nicht aus dem Paper)

- Im selben Jahr erschien Crossbow (Langmead, Schatz et al. 2009), eine Hadoop-Pipeline aus dem BWT-Aligner Bowtie und dem SNP-Caller SoapSNP. Die Idee, Genomanalysen mit Hadoop in die Cloud zu bringen, wurde also direkt weitergeführt.
- Beim Read-Mapping haben sich BWT-basierte Aligner wie BWA und Bowtie 2 durchgesetzt. Für die meisten Anwendungen reicht ein bestes Alignment pro Read, und dafür sind sie sehr schnell und speichersparend.
- Reads sind heute länger und oft gepaart. Bei langen Reads mit hoher Fehlerrate werden die Blöcke nach dem Schubfach-Schema sehr kurz, und die Kandidatenflut aus 4.4 schlägt voll zu. Moderne Aligner nutzen deshalb andere Seed-Strategien, etwa maximale exakte Treffer (BWA-MEM) oder Minimizer (minimap2).
- Für Big-Data-Pipelines hat Apache Spark Hadoop MapReduce weitgehend abgelöst. Spark kann Zwischenergebnisse im Speicher halten. Für CloudBurst hieße das zum Beispiel, die Referenz-Seeds einmal zu erzeugen und für mehrere Read-Batches wiederzuverwenden.
- Die Grundidee bleibt lehrreich: Ein Problem so umformulieren, dass die teure Suche ein Gruppieren nach Schlüssel wird. Dann übernimmt das Framework die Verteilung.

---

## 9. Glossar

| Begriff | Erklärung |
| --- | --- |
| 1000 Genomes Project | Internationales Projekt zur Sequenzierung vieler menschlicher Genome. Quelle der Testdaten. |
| Alignment | Ausrichtung eines Reads an einer Stelle der Referenz, mit Angabe der Unterschiede. |
| Amdahl'sches Gesetz | Der nicht parallelisierbare Anteil f begrenzt den Speedup auf höchstens 1/f. |
| Base, bp | Baustein der DNA (A, C, G, T) und Längeneinheit (Basenpaar). |
| BLAST | Klassisches Programm zur Sequenzsuche. Hashtabelle über die Referenz-k-mere, dann gebändertes Smith-Waterman. |
| BWT, BWT-Aligner | Burrows-Wheeler-Transformation, Grundlage eines komprimierten Genomindex. Aligner: Bowtie, BWA, SOAP2. |
| Combiner | Mini-Reduce direkt nach dem Map auf demselben Rechner, reduziert die Datenmenge im Shuffle. |
| EC2 | Amazon Elastic Compute Cloud, Mietrechner nach Stunden. |
| Edit-Distanz | Minimale Zahl an Ersetzungen, Einfügungen und Löschungen zwischen zwei Strings. |
| End-to-end-Alignment | Der ganze Read wird ausgerichtet, nicht nur ein Teil. |
| FASTA, Multi-FASTA | Textformat für Sequenzen: Kopfzeile mit „>“, dann die Sequenz. Multi-FASTA enthält viele Einträge. |
| Flanke | Die Basen links und rechts eines Seeds, in MerInfo mitgeschickt. |
| GFS, HDFS | Verteilte Dateisysteme von Google bzw. Hadoop. Große Blöcke, mehrfach repliziert. |
| Hadoop | Open-Source-Implementierung von MapReduce und HDFS in Java. |
| Hamming-Distanz | Zahl der Positionen, an denen sich zwei gleich lange Strings unterscheiden. |
| Indel | Eingefügte oder gelöschte Base. |
| k | In CloudBurst: maximal erlaubte Zahl an Unterschieden pro Read. |
| k-mer | Teilstring fester Länge. In CloudBurst haben die k-mere die Länge s. |
| Landau-Vishkin | Algorithmus für Alignments mit höchstens k Unterschieden in O(km), arbeitet nur auf 2k+1 Diagonalen. |
| Load-Balancing | Gleichmäßige Verteilung der Arbeit auf alle Rechner. |
| Low-Complexity-Seed | Seed aus nur einer Base (AAAAAAA). Kommt extrem oft vor. |
| MapReduce | Programmiermodell: Map erzeugt Schlüssel-Wert-Paare, Shuffle gruppiert nach Schlüssel, Reduce verarbeitet jede Gruppe. |
| MerInfo | Wert zu einem Seed in CloudBurst: (id, position, isRef, isRC, left_flank, right_flank). |
| Mismatch | Ersetzte Base. |
| NGS | Next-generation sequencing, Hochdurchsatz-Sequenzierung mit Millionen kurzer Reads pro Lauf. |
| Paired-end | Beide Enden eines DNA-Stücks werden gelesen. Von CloudBurst nicht unterstützt. |
| Partitionierung | Zuordnung der Schlüssel zu Reducern, meist per Hash. |
| Read | Kurzes, vom Sequenzierer gelesenes DNA-Stück. |
| Read-Mapping | Für jeden Read die Stelle(n) in der Referenz finden, mit wenigen erlaubten Unterschieden. |
| Redundanz-Trick | Häufige Seeds der Referenz r-mal kopieren, Read-Vorkommen zufällig auf die Kopien verteilen. |
| Referenzgenom | Bekannte Genomsequenz, auf die kartiert wird. |
| Reverse Complement | Sequenz des Gegenstrangs: umdrehen, A↔T und C↔G tauschen. |
| RMAP, RMAPM, RMAPQ | Serieller Read-Mapper von Smith et al. (2008). M: mit Mismatch-Scores, Q: mit Qualitätswerten. Vorbild und Vergleich für CloudBurst. |
| Seed | Exakt übereinstimmender Teilstring, der als Anker für ein Alignment dient. |
| Seed-and-Extend | Erst exakte Seeds finden, dann zu vollständigen Alignments verlängern. |
| Sensitivität | Anteil der vorhandenen Treffer, die gefunden werden. CloudBurst findet alle mit höchstens k Unterschieden. |
| SequenceFile | Binäres Hadoop-Format für Schlüssel-Wert-Paare. |
| Shuffle | Phase zwischen Map und Reduce, die alle Paare nach Schlüssel gruppiert und verteilt. |
| Single-end | Nur ein Ende eines DNA-Stücks wird gelesen. |
| Smith-Waterman | DP-Algorithmus für lokale Alignments, Laufzeit proportional zum Produkt der Längen. |
| SNP | Single nucleotide polymorphism, Unterschied in einer einzelnen Base gegenüber der Referenz. |
| Spaced Seed | Seed, bei dem nur bestimmte Positionen nach einem Muster übereinstimmen müssen. |
| Speedup | Serielle Laufzeit geteilt durch parallele Laufzeit. |
| Schubfachprinzip | Verteilt man k Fehler auf k+1 Blöcke, bleibt mindestens ein Block fehlerfrei. |

---

## 10. Quellen

- Schatz, M. C. (2009): [CloudBurst: highly sensitive read mapping with MapReduce](https://doi.org/10.1093/bioinformatics/btp236). Bioinformatics 25(11), 1363–1369. Quellcode laut Paper unter cloudburst-bio.sourceforge.net.
- Dean, J.; Ghemawat, S. (2008): [MapReduce: Simplified Data Processing on Large Clusters](https://doi.org/10.1145/1327452.1327492). Communications of the ACM 51(1), 107–113.
- Ghemawat, S. et al. (2003): The Google File System. SOSP 2003, 29–43.
- Smith, A. D. et al. (2008): Using quality scores and longer reads improves accuracy of Solexa read mapping. BMC Bioinformatics 9, 128. Das RMAP-Paper.
- Landau, G. M.; Vishkin, U. (1986): Introducing efficient parallelism into approximate string matching and a new serial algorithm. STOC 1986, 220–230.
- Baeza-Yates, R. A. et al. (1992): Fast and practical approximate string matching. CPM 1992, 185–192. Quelle des Schubfach-Arguments.
- Gusfield, D. (1997): Algorithms on Strings, Trees, and Sequences. Cambridge University Press. Lehrbuch zu Alignment und Landau-Vishkin.
- Langmead, B. et al. (2009): [Ultrafast and memory-efficient alignment of short DNA sequences to the human genome](https://doi.org/10.1186/gb-2009-10-3-r25). Genome Biology 10, R25. Bowtie.
- Langmead, B.; Schatz, M. C. et al. (2009): Searching for SNPs with cloud computing. Genome Biology 10, R134. Crossbow.
