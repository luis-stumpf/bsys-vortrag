# Rowhammer-Theorie

### Grundlagen

#### Der Speicher als Fehler

Die Schwäche heutiger Speichersysteme gegenüber potentiellen Rowhammer-Angriffen liegt  in der Natur des DRAM-Speichers selbst. Die Speicherzellen **sind** der Fehler im System. Wir starten unsere Reise in die tiefen der Hacker-Theorie also im kleinen: genauer im D-RAM-Bit.

##### Dynamic Random Acsess Memory - DRAM

DRAM hat viele Vorteile. Er ist schnell, kostet wenig und bleibt dabei konzeptionell eigentlich simpel. "simpel" in Anführungszeichen, weil das nur im Vergleich zu anderen Speicherbausteinen gilt. DRAM-Herstellung ist natürlich High-Tech, allerdings reicht uns hier eine grobe Idee, der Architektur. [[1]](#1)

Um die Probleme der Speicherbausteine gegen Rowhammer-Angriffe zu verstehen, betrachten wir nachfolgend den theoretischen Aufbau solcher Zellen.

DRAM-Bits bestehen prinzipiell nur aus einem Transistor und einem Kondensator (Capacitor), der Information speichert (Siehe Bild). 

<img title="" src="Abb1Markdown.PNG" alt="" width="356" data-align="center">

Abb. 1 [[1]](#1) Bild aus DRAM Circuit Design S. 10 Fig, 1.12

DRAM ist als eine Matrix von Bits aufgebaut (Abb2). Jedes Bit in dieser Speichermatrix wird über eine sogenannte Wort-Leitung und eine Bit-Leitung adressiert - auch Row und Column genannt. 

Wenn zum Beispiel in ein Bit geschrieben werden soll, muss die Wordline nur Spannung auf den Gate-Anschluss (Abb1 G) des Transistors geben. Damit wird eine leitende Verbindung von Drain- (Abb1 D) zu Source-Gebieten (Abb1 S) hergestellt und der Kondensator kann über die Bit-Leitung geladen werden. Ganz ähnlich wird auch aus den Kondensatoren gelesen, auf eine genaue Erklärung zur Funktion von D-RAM Bits verzichten wir hier aber. 

Bereits die kleinsten Bausteine enthalten über 1000 Speicherzellen in einer Matrix. In Abbildung zwei wird ein Zugriff in einer solchen Matrix vereinfacht dargestellt.

<img title="" src="Abb2Markdown.PNG" alt="" data-align="center" width="580">

Abbildung 2 vereinfachte Matrix-Darstellung der Bits 

##### Das Problem mit der Grösse

DRAM kann extrem kompakt hergestellt werden. Eine Grafik aus dem 2012 veröffentlichten Buch "CHIPS 2020" zeigt, das der durchscnittliche DRAM-Baustein schon vor Jahren aus Billionen von Bits je Quadratzentimeter bestand. [[3]](#3) Das erlaubt Speicherzugriffe im Nanosekundenbereich. 

<img title="" src="Abb3Markdown.PNG" alt="" width="404" data-align="center">

Abbildung 3 *Evolution of CMOS memory density for SRAM, DRAM and Flash memory.* Aus CHIPS 2020 [[2]](#2)  [[3]](#3)

Allerdings bringen Chips mit dieser nur schwer vorstellbaren Kompaktheit und geradezu irren Geschwindigkeit ihre eigenen Herausforderungen mit sich. Kurz: Es gibt auch Fehlerhafte Bits. 

Diese machen zwar den Chip nicht unbrauchbar, können aber zum Beispiel kleine Lecks verursachen, die andere Bits in direkter Nachbarschaft beinflussen. Im Prinzip können Bits so Ladung austauschen, obwohl sie eigentlich voneinander isoliert sein sollten. Und da kommen wir endlich zur Krux: Denn genau diese "schwachen" Bits versucht Rowhammer auszunutzen.  [[4]](#4)

#### Row-Hammer

##### Die Idee

Das Prinzip von Rowhammer ist äusserst einfach. Schwache Bits im Speicher sollen gezielt ausgenutzt werden. Über **schnelle Speicherzugriffe des immer selben Speicherbereichs**, wird im Nanosekundenbereich wieder und wieder die gleiche Wort-Leitung (Row) unter Spannung gesetzt. Das Programm "hämmert" sozusagen auf eine Reihe ein. Daher der Name *Row-Hammer*.

Durch dieses Hämmern erhöht sich die Chance, das ein "schwaches" Bit nachgibt und ein Leck verursacht deutlich.[[4]](#4) Im schlimmsten Fall löst das entsprechende Bit, dann einen Statuswechsel in einer anderen Reihe aus. Oder ganz direkt formuliert: Das Leck könnte in einemnaheliegenden Bit eine 0 zu einer 1 machen.

##### Code

Der für das reine "hämmern" nötige Code beschränkt sich für gängige Prozessoren auf ein paar Zeilen x86-Assembler.

<img src="Abb4Markdown.PNG" title="" alt="" data-align="center">

Abbildung 4  Beispiel aus *Flipping Bits in Memory Without Accessing Them:  
An Experimental Study of DRAM Disturbance Errors* [[5]](#5)

In Zeile 1 und 2 werden Registerinhalte in Reihe X und Y gespeichert. Gleich darauf löscht (invalidiert) das Programm in Zeile 4 und 5 die Reihen wieder[[6]](#6) und stellt mit mfence in Zeile 6 sicher,  dass die Speicherzugriffe in der richtigen Reihenfolge ablaufen. In Zeile 7 wird das Programm rekursiv aufgerufen. 

Der Code ist einfach und noch viel schlimmer: um diesen Code ausführen zu können, sind keine besonderen Rechte notwendig.

Das speichern und flushen in zwei unterschiedlichen Reihen ist ein kleiner Umweg, weil gängiger RAM eine Reihe zwischenspeichert, um diese nicht wiederholt aufrufen zum müssen. [[5]](#5)  Ohne den abwechselnden Zugriff auf eine andere Reihe würde das System also nur vordergründig anfragen an die immer gleiche Zeile richten.  

Damit lassen sich Schwachstellen zwar noch nicht gezielt ausnutzen, ein Testprogramm von Google [[0]](#0) kommt aber zum Beispiel bereits mit einigen hundert Zeilen Code aus. Aktuelle Systeme lassen sich nicht so einfach austricksen. Wichtig ist uns hier zu zeigen, dass Rowhammer auf einem sehr grundlegenden Level agiert.

<img src="Abb5Markdown.PNG" title="" alt="" data-align="center">

Abbildung 5 *RowHammer error rate vs. manufacturing dates of 129  
DRAM modules we tested.* Aus RowHammer: a Retrospektive [[4]](#4) (Die Namen der Hersteller wurden anonymisiert)

Abbildung 5 zeigt elementar, dass die Anzahl der anfälligen Bits in Speicherblöcken bis etwa 2012 immer weiter zugenommen hat. Je näher die Bits also zusammenliegen, umso anfälliger sind sie für Row-Hammering. Den Abwärtstrend in späteren Jahren thematisieren wir in einem der nachfolgenden Kapitel. 

 

##### Systemschwächen ausnutzen

Ab hier beginnt der kompliziertere Teil des Angriffes. Bis jetzt kann unser Prozess nur einzelne Bits zum kippen bringen, dies geschieht allerdings eher zufällig und bringt im allerschlimmsten Fall vielleicht ein Programm zum Absturz.

Um einen wirklich nutzbaren Row-Hammer-Angriff anzusetzen, muss sich der Angreifer  in der Paging-Trickkiste bedienen. Wir nehmen hier als Beispiel ein Linux-System, dass so auch von Google-Forschern gehackt wurde.



### Rowahmmer-Angriff auf den Linux-Kernel



##### Linux Speicherschutz durch virtualisierung

Der Linux-Kernel hat prinzipiell eine sichere Methode um Speicher zu verwalten. Das System **virtualisiert** den physikalischen Speicher für Nutzer quasi unsichtbar über Software- und Hardware-Bausteine. [[7]](#7) Auch andere Betriebssysteme wie Windows und MAC-OS machen sich diesen Trick zu nutze. 

Das Betriebsystem teilt den physikalischen Speicher in Seiten (Seitenrahmen ode rauch Seitenkacheln) ein, und kartiert die Kacheln in einem separaten Speicherbereich. Die Karte für eine solche Seite, im folgenden Pageframe genannt, ist nur für das Betriebsystem sichtbar. Nutzer-Prozessen wird vom Betriebsystem schlicht nicht erlaubt, auf Pageframes zuzugreifen.

So schützt das Betriebssystem sich selbst und Prozesse voreinander. Sie können nur auf für sie speziell kartierte Bereiche zugreifen. 

Nachfolgend zeigt sich aber: genau diese Pageframes, also die im Speicher abgelgeten Karten können Linux zum Verhängnis werden. 



##### Der Pagetable als Achillesverse

Der Schutz von Speicher gegen bösartige Prozesse basiert im Grunde genommen darauf, dass Prozesse ihren eigenen Pagetable nicht verändern können, weil dieser ausserhalb des für sie kartierten Bereiches liegt. 

Hier schlagen wir eine Bresche in das schöne Sicherheitskonstrukt von Linux. Um das Beispiel zu vereinfachen, nehmen wir an, dass der physikalische Speicher, von Betriebsystem-Daten einmal abgesehen, leer ist.

<u>Schritt 1</u>: Wir starten unser bösartiges Programm und erstellen einen geteilten Speicherbereich (shared Page), auf welchen wiederholt zugegriffen werden kann.[[8]](#8)

~~Schitt 1.5 ... schwaches Bit suchen???????????? vllt. weglassen da zu lang~~

<u>Schritt 2</u>: Das Programm allokiert mit dem Systemaufruf mmap() [[9]](#9) immer wieder denselben Speicher. Es befindet sich also weiter **nur eine Page** im physikalischen Speicher.

**ABER**: Jedes mapping erhält weiterhin einen eigenen Pagetable für den geteilten Speicherbereich. Der Speicher wir regelrecht mit **millionen Page-Tables** zugepflastert. Diese benötigt zugegeben nicht sonderlich viel Speicher (z.B. 4 kB)[[8]](#8). Werden wie in unserem Beispiel allerdings Milllionen von Page-Tables angelegt, füllt das den Speicher schnell. 

<u>Schritt 3</u>: **HAMMER-TIME**. Jetzt wo der Speicher praktisch nur noch aus einer Page und Millionen von Page-Tables besteht, kommt der grosse Moment des kleinen Code-Abschnitts in Abbildung 4. Das Programm hämmert im Nanosekundenbereich auf eine Reihe im allokierten Speicher ein. Mit etwas Glück leckt ein Bit. Im benachbarten Pagetable wechselt eine 0 zu einer 1.

<u>Schritt 4</u>: Dieses eine Bit, kann nun das komplette System kompromittieren. Denn ändert das Bit an der richtigen Stelle seinen Wert, ändert dies die Kartierung des zugehörigen Prozesses. Oder einfacher: Der Page-Table zeigt plötzlich auf einen völlig anderen Bereich im physikalischen Speicher.

Und noch viel besser: Weil der komplette Speicher ja immer noch voller Page-Tables ist, zeigt unser neuer Pointer mit grosser wahrscheinlichkeit auf genau so einen Page-Table.

Ziel ist jetzt, den Speicherbereich zu finden, der zum attakierten Page-Table gehört. Ist dieser gefunden, sind der Fantasie ab da eigentlich keine Grenzen gesetzt.[[8]](#8)

<u>Schritt 5</u>: Wir haben Schreib- und Lesezugriff auf einen Pagetable und das im User-Mode. Damit können wir den Pagetable ohne Probleme den kompletten physikalischen Speicher kartieren lassen. Das System gehört praktisch uns. 



Die Vorgehensweise ist hier natürlich sehr vereinfacht dargestellt. Die Suche nach schwachen Bits und das erkennen der richtigen Tables, sowie das korrete allokieren und teilweise Freigeben von Speicher brauchen entsprechende Vorbereitung und Fachwissen. 





...

...

...

##### Rowhammer at Home

Google hat einen Row-Hammer-Test für x86-Computer als Open-Source-Projekt veröffentlicht. Der Test funktioniert allerdings nur mit älteren Ram-Riegeln (DDR3).[[0]](#0)

<img title="" src="Abb0Markdown.PNG" alt="" width="547" data-align="center">


id="1">[1] Referenz 1[DRAM Circuit Design: Fundamental and High-Speed Topics - Brent Keeth, R. Jacob Baker, Brian Johnson, Feng Lin - Google Books](https://books.google.de/books?id=TgW3LTubREQC&pg=PA33&hl=de&source=gbs_toc_r&cad=3#v=onepage&q&f=false)

id="2">[2] Referenz 2[Towards Terabit Memories | Chips 2020](https://www.chips2020.net/chapter/towards-terabit-memories)

id="3">[3] Referenz 3  [[Chips 2020 | SpringerLink](https://link.springer.com/book/10.1007/978-3-642-23096-7)]

id="4">[4] Referenz 4 [[[1904.09724] RowHammer: A Retrospective](https://arxiv.org/abs/1904.09724)]

id="5">[5] Referenz 5 [[Flipping bits in memory without accessing them: An experimental study of DRAM disturbance errors | IEEE Conference Publication | IEEE Xplore](https://ieeexplore.ieee.org/document/6853210)]

id="6">[6] Referenz 6 [Into the Void: x86 Instruction Set Reference](https://c9x.me/x86/html/file_module_x86_id_30.html)

id="7">[7] Referenz 7 [https://pages.cs.wisc.edu/~remzi/OSTEP/vm-paging.pdf ]

id="8">[8] Referenz 8 [Project Zero: Exploiting the DRAM rowhammer bug to gain kernel privileges](https://googleprojectzero.blogspot.com/2015/03/exploiting-dram-rowhammer-bug-to-gain.html)

id="9">[9] Referenz 9 [mmap(2) - Linux manual page](https://man7.org/linux/man-pages/man2/mmap.2.html)

id="0">[0] Referenz 0 [GitHub - google/rowhammer-test: Test DRAM for bit flips caused by the rowhammer problem](https://github.com/google/rowhammer-test)

