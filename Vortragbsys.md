---

title: "Rowhammer: Problem ohne Lösung"
subtitle: "Gruppe 7"
author: 

- Luis Stumpf
- Lewin Eder
  date: 03.05.2022
  lang: de

---



# Rowhammer: Problem ohne Lösung





### Inhalt



1. Grundlagen
   
   1. Einleitung: Der Speicher als Fehler
   
   2. Dynamic Random Access Memory -DRAM
   
   3. Das Problem mit der Grösse

2. Rowhammer
   
   1. Die Idee
   
   2. Code
   
   3. Systemschwächen ausnutzen

3. Von der Theorie zur Praxis: Angriff auf den Linux-Kernel
   
   1. Linux Speicherschutz durch Virtualisierung
   
   2. Der Page-Table als Achillesverse

4. Rowhammer in der Realität
   
   1. Throwhammer: Eine Rowhammer-Attacke über Netzwerke
   
   2. Jackhammer: Eine Rowhammer-Attacke über Websites

5. Schutz gegen Rowhammer
   
   1. ECC
   
   2. Refresh-Rate

6. Fazit

 





### Grundlagen



##### Einleitung: Der Speicher als Fehler

Die Schwäche heutiger Speichersysteme gegenüber potentiellen Rowhammer-Angriffen liegt  in der Natur des DRAM-Speichers selbst. Die Speicherzellen **sind** der Fehler im System. Wir starten unsere Reise in die tiefen der Hacker-Theorie also im kleinen: genauer im D-RAM-Bit.



##### Dynamic Random Access Memory - DRAM

DRAM hat viele Vorteile. Er ist schnell, kostet wenig und bleibt dabei konzeptionell eigentlich simpel. "simpel" in Anführungszeichen, weil das nur im Vergleich zu anderen Speicherbausteinen gilt. DRAM-Herstellung ist natürlich High-Tech, allerdings reicht uns hier eine grobe Idee, der Architektur. [[1]](#1)

Um die Probleme der Speicherbausteine gegen Rowhammer-Angriffe zu verstehen, betrachten wir nachfolgend den theoretischen Aufbau solcher Zellen.

DRAM-Bits bestehen prinzipiell nur aus einem Transistor und einem Kondensator (Capacitor), der Information speichert (Siehe Bild). 

<img title="" src="file:///C:/Users/leder/AppData/Roaming/marktext/images/2022-04-28-18-39-03-image.png" alt="" width="356" data-align="center">

Abb. 1 [[1]](#1) Bild aus DRAM Circuit Design S. 10 Fig, 1.12

DRAM ist als eine Matrix von Bits aufgebaut (Abb2). Jedes Bit in dieser Speichermatrix wird über eine sogenannte Wort-Leitung und eine Bit-Leitung adressiert - auch Row und Column genannt. 

Wenn zum Beispiel in ein Bit geschrieben werden soll, muss die Wordline nur Spannung auf den Gate-Anschluss (Abb1 G) des Transistors geben. Damit wird eine leitende Verbindung von Drain- (Abb1 D) zu Source-Gebieten (Abb1 S) hergestellt und der Kondensator kann über die Bit-Leitung geladen werden. Ganz ähnlich wird auch aus den Kondensatoren gelesen, auf eine genaue Erklärung zur Funktion von D-RAM Bits verzichten wir hier aber. 

Bereits die kleinsten Bausteine enthalten über 1000 Speicherzellen in einer Matrix. In Abbildung zwei wird ein Zugriff in einer solchen Matrix vereinfacht dargestellt.

<img title="" src="file:///C:/Users/leder/AppData/Roaming/marktext/images/2022-04-28-19-40-28-image.png" alt="" data-align="center" width="580">

Abbildung 2 vereinfachte Matrix-Darstellung der Bits 



##### Das Problem mit der Grösse

DRAM kann extrem kompakt hergestellt werden. Eine Grafik aus dem 2012 veröffentlichten Buch "CHIPS 2020" zeigt, das der durchscnittliche DRAM-Baustein schon vor Jahren aus Billionen von Bits je Quadratzentimeter bestand. [[3]](#3) Das erlaubt Speicherzugriffe im Nanosekundenbereich. 

<img title="" src="file:///C:/Users/leder/AppData/Roaming/marktext/images/2022-04-28-20-03-26-image.png" alt="" width="404" data-align="center">

Abbildung 3 *Evolution of CMOS memory density for SRAM, DRAM and Flash memory.* Aus CHIPS 2020 [[2]](#2)  [[3]](#3)

Allerdings bringen Chips mit dieser nur schwer vorstellbaren Kompaktheit und geradezu irren Geschwindigkeit ihre eigenen Herausforderungen mit sich. Kurz: Es gibt auch Fehlerhafte Bits. 

Diese machen zwar den Chip nicht unbrauchbar, können aber zum Beispiel kleine Lecks verursachen, die andere Bits in direkter Nachbarschaft beinflussen. Im Prinzip können Bits so Ladung austauschen, obwohl sie eigentlich voneinander isoliert sein sollten. Und da kommen wir endlich zur Krux: Denn genau diese "schwachen" Bits versucht Rowhammer auszunutzen.  [[4]](#4)





### Row-Hammer

##### Die Idee

Das Prinzip von Rowhammer ist äusserst einfach. Schwache Bits im Speicher sollen gezielt ausgenutzt werden. Über **schnelle Speicherzugriffe des immer selben Speicherbereichs**, wird im Nanosekundenbereich wieder und wieder die gleiche Wort-Leitung (Row) unter Spannung gesetzt. Der Angreifer "hämmert" sozusagen auf eine Reihe ein. Daher der Name *Row-Hammer*.

Durch dieses Hämmern erhöht sich die Chance, das ein "schwaches" Bit nachgibt und ein Leck verursacht deutlich.[[4]](#4) Im schlimmsten Fall löst das entsprechende Bit dann einen Statuswechsel in einer anderen Reihe aus. Oderetwas konkreter formuliert: Das Leck kann in einem naheliegenden Bit eine 0 zu einer 1 machen.

##### Code

Der für das reine "hämmern" nötige Code beschränkt sich für gängige Prozessoren auf ein paar Zeilen x86-Assembler.

<img title="" src="file:///C:/Users/leder/AppData/Roaming/marktext/images/2022-04-28-21-30-36-image.png" alt="" width="253" data-align="center">

Abbildung 4  Beispiel aus *Flipping Bits in Memory Without Accessing Them:  
An Experimental Study of DRAM Disturbance Errors* [[5]](#5)

In Zeile 1 und 2 werden Registerinhalte in Reihe X und Y gespeichert. Gleich darauf spült (invalidiert) das Programm in Zeile 4 und 5 die Reihen wieder[[6]](#6) und stellt mit mfence in Zeile 6 sicher,  dass die Speicherzugriffe in der richtigen Reihenfolge ablaufen. In Zeile 7 wird der Code dann erneut aufgerufen, die Speicherzugriffe wiederholen sich.

Das speichern und flushen in zwei unterschiedlichen Reihen ist ein kleiner aber nötiger Umweg, da gängiger RAM eine Reihe zwischenspeichert, um diese nicht wiederholt aufrufen zum müssen. [[5]](#5)  Ohne den abwechselnden Zugriff auf eine andere Reihe würde das System also nur vordergründig anfragen an die immer gleiche Zeile richten. 

Trotzdem könnte das Programm einfacher nicht sein und fast noch schlimmer: um diesen Code auszuführen, sind keine Root-Rechte notwendig.

Damit lassen sich Schwachstellen zwar noch nicht gezielt ausnutzen, ein Testprogramm von Google [[0]](#0) kommt allerdings  bereits mit einigen hundert Zeilen Code aus. Ja -Aktuelle Systeme lassen sich nicht ganz so einfach überlisten. Wichtig ist uns hier einfach zu zeigen, dass Rowhammer auf einem sehr grundlegenden Level agiert und einen Fehler ausnutzt, der sich nicht so einfach "weg-patchen" lässt.

<img src="file:///C:/Users/leder/AppData/Roaming/marktext/images/2022-04-28-21-07-21-image.png" title="" alt="" data-align="center">

Abbildung 5 *RowHammer error rate vs. manufacturing dates of 129  
DRAM modules we tested.* Aus RowHammer: a Retrospektive [[4]](#4) (Die Namen der Hersteller wurden anonymisiert)

Abbildung 5 zeigt elementar, dass die Anzahl der anfälligen Bits in Speicherblöcken bis etwa 2012 immer weiter zugenommen hat. Je näher die Bits also zusammenliegen, umso anfälliger sind sie für Row-Hammering. Den Abwärtstrend in späteren Jahren thematisieren wir in einem der nachfolgenden Kapitel. 

 

##### Systemschwächen ausnutzen

Ab hier beginnt der kompliziertere Teil des Angriffes. Bis jetzt kann unser Prozess nur einzelne Bits zum kippen bringen, dies geschieht allerdings eher zufällig und bringt im allerschlimmsten Fall vielleicht ein Programm zum Absturz.

Um einen wirklich nutzbaren Row-Hammer-Angriff anzusetzen, muss sich der Angreifer  in der Paging-Trickkiste bedienen. Wir nehmen hier als Beispiel ein Linux-System, dass so auch von Google-Forschern gehackt wurde.





### Von der Theorie zur Praxis: Angriff auf den Linux-Kernel

##### Linux Speicherschutz durch Virtualisierung

Der Linux-Kernel hat prinzipiell eine sichere Methode um Speicher zu verwalten. Das System **virtualisiert** den physikalischen Speicher für Nutzer quasi unsichtbar über Software- und Hardware-Bausteine. [[7]](#7) Auch andere Betriebssysteme wie Windows und MAC-OS machen sich diesen Trick zu nutze. 

Das Betriebsystem teilt den physikalischen Speicher in Seiten (Seitenrahmen ode rauch Seitenkacheln) ein, und kartiert die Kacheln in einem separaten Speicherbereich. Die Karte für eine solche Seite, im folgenden Pageframe genannt, ist nur für das Betriebsystem sichtbar. Nutzer-Prozessen wird vom Betriebsystem schlicht nicht erlaubt, auf Pageframes zuzugreifen.

So schützt das Betriebssystem sich selbst und Prozesse voreinander. Sie können nur auf für sie speziell kartierte Bereiche zugreifen. 

Nachfolgend zeigt sich aber: genau diese Pageframes, also die im Speicher abgelgeten Karten können Linux zum Verhängnis werden. 



##### Der Pagetable als Achillesverse (Schritt für Schritt-Anleitung)

Der Schutz von Speicher gegen bösartige Prozesse basiert im Grunde genommen darauf, dass Prozesse ihren eigenen Pagetable nicht verändern können, weil dieser ausserhalb des für sie kartierten Bereiches liegt. 

Hier schlagen wir eine Bresche in das schöne Sicherheitskonstrukt von Linux. Um das Beispiel zu vereinfachen, nehmen wir an, dass der physikalische Speicher, von Betriebsystem-Daten einmal abgesehen, leer ist.

<u>Schritt 1</u>:

Wir starten unser bösartiges Programm und erstellen einen geteilten Speicherbereich (shared Page), auf welchen wiederholt zugegriffen werden kann.[[8]](#8)

<u>Schritt 2</u>: 

Das Programm allokiert mit dem Systemaufruf mmap() [[9]](#9) immer wieder denselben Speicher. Es befindet sich also weiter **nur eine Page** im physikalischen Speicher.

**ABER**: Jedes mapping erhält weiterhin einen eigenen Pagetable für den geteilten Speicherbereich. Der Speicher wir regelrecht mit **millionen Page-Tables** zugepflastert. Diese benötigt zugegeben nicht sonderlich viel Speicher (z.B. 4 kB)[[8]](#8). Werden wie in unserem Beispiel allerdings Milllionen von Page-Tables angelegt, füllt das den Speicher schnell. 

<u>Schritt 3</u>: 

**HAMMER-TIME**. Jetzt wo der Speicher praktisch nur noch aus einer Page und Millionen von Page-Tables besteht, kommt der grosse Moment des kleinen Code-Abschnitts in Abbildung 4. Das Programm hämmert im Nanosekundenbereich auf eine Reihe im allokierten Speicher ein. Mit etwas Glück leckt ein Bit. Im benachbarten Pagetable wechselt eine 0 zu einer 1.

<u>Schritt 4</u>: 

Dieses eine Bit, kann nun das komplette System kompromittieren. Denn ändert das Bit an der richtigen Stelle seinen Wert, ändert dies die Kartierung des zugehörigen Prozesses. Oder einfacher: Der Page-Table zeigt plötzlich auf einen völlig anderen Bereich im physikalischen Speicher.

Und noch viel besser: Weil der komplette Speicher ja immer noch voller Page-Tables ist, zeigt unser neuer Pointer mit grosser wahrscheinlichkeit auf genau so einen Page-Table.

Ziel ist jetzt, den Speicherbereich zu finden, der zum attakierten Page-Table gehört. Ist dieser gefunden, sind einem Angreifer eigentlich keine Grenzen gesetzt.[[8]](#8)

<u>Schritt 5</u>: 

Wir haben Schreib- und Lesezugriff auf einen Pagetable und das im User-Mode. Damit können wir den Pagetable ohne Probleme den kompletten physikalischen Speicher kartieren lassen. Das System gehört praktisch uns. 

Die Vorgehensweise ist hier natürlich sehr vereinfacht dargestellt. Die Suche nach schwachen Bits und das erkennen der richtigen Tables, sowie das korrete allokieren und teilweise Freigeben von Speicher brauchen entsprechende Vorbereitung und Fachwissen. 





### Rowahmmer in der Realität

Der Beschriebene Angriff auf das Linux-System ist für Hacker natürlich nur bedingt nutzbar. Der Angreifer muss lokal ein Programm einschleusen. Allerdings können Angriffe nachgewiesenermassen sehr vielseiteig erfolgen. Im folgenden führen wir zwei Beispiele auf, die mögliche "Eintritts-Punkte" offenlegen.



##### Throwhammer:

##### Eine Rowhammer-Attacke über Netzwerke

Ganz allgemein lässt sich Rowhammer nur dann ausnutzen, wenn wir mehr oder weniger direkt Prozesse starten und Speicher anfordern können. Im Gegensatz zu lese- und Schreibzugriffen im Hauptspeicher sind Netzwerk-Anfragen geradezu lachhaft langsam. 

Das heisst allerdings keineswegs, dass Rowhammer-Attacken von einem "Ausenstehenden" nicht möglich sind. Rowhammer kann "remote" eingesetzt werden und ganz treffend, wird diese Methode dann auch "Throw"-Hammer genannt.[[15]](15)

Zielobjekte einer solchen Attacke wären in der Regel Cloudservices, da auf diesen nicht selten auch sensible Daten abgelegt werden.  

Um eine Throwhammer Attacke durchführen zu können, braucht das Zielsystem eine Netzwerkanbindung mit einer Geschwindigkeit von mindest 10 Gigabit pro Sekunde. Das ist heutzutage keine Seltenheit mehr. **[Wo Zitat]** Bei älteren Speicher-Systemen braucht der Angreifer nur einen normalen User-Zugang ohnen besondere Rechte.

Auf der Serversseite allokiert der "Hacker" nun einen Zusammenhängenden Spreicher und füllt diesen Speicherbereich wiederholend mit nur Nullen oder Einsen. Währenddessen wird überprüft, ob ein Bit im allokierten Speicher "geflippt" ist. Gibt ein Bit nach, lässt sich so auch in Speicherbereiche schreiben, auf die ein Nutzer keinen Zugriff haben sollte. Allerdings ist hier eine gewisse Kentniss über das System nötig, da der Nutzer Speicher intelligent allokieren und freigeben muss. [[15]](15) 

Fazit: Sensible Daten nur auf dem eigenen Server, oder auf Servern die nicht "jeden" Kunden akzeptieren.



##### Jackhammer

##### Eine Rowhammer-Attacke über Websites (Rowhammer.js)

Auch Attacken auf Heimcomputer sind nicht unmöglich. Hier dient, dass bei Programmierern äusserst beliebte Java-Script, einmal mehr als Ansetzpunkt für unser virtuelles Brecheisen. 

Die Programmiersprache Javascript hat zwar zunächste keine möglichkeit Pointer und virtuelle Adressen anzusprechen. Kann sich aber sehr große Arrays generieren lassen. Diese werden auf allen aktuellen Google-Chrome- und Firefox-Browser in einem Linux System als  2-MB-Pages allokiert. Verantwortung trägt hier das Betriebsystem. Die Methode wird bei jeder Scriptsprache so angewendet.[[16]](16) Ein bösartiges Skript kann so wiederholt in hoher Frequenz über die Arrays iterieren und damit auch von einer Website aus auf Reihen einhämmern, ergo ein Bit zum kippen bringen. Ab hier geht ein Angreifer wieder nach dem bereits bekannten Konzept vor. 

Der springende Punkt: Ein Angriff startet auf dem Heimcomputer direkt beim Aufrufen einer bösartigen Webseite und ist als solcher auch nicht einfach zu erkennen. 

Immerhin bleibt es auch als Heimcomputer-Nutzer denkbar einfach, sich zumindest gegen solche Angriffe zu schützen. Alle gängigen Browser könnnen Java-Script für unbekannte Webseiten deaktivieren, was wir auch ganz allgemein empfehlen würden. [[16]](16)



(Two side attacks)**WAS HIER NOCH FEHLT??**



### Schutz gegen Rowhammerattacken



Hier beginnen die richtig schlechten Nachrichten. Die meissten Autoren, auf die wir gestossen sind, bezweifeln dass sich Systeme aktuell mit volkommener Sicherheit gegen Rowhammer schützen lassen [[10]](10).  Das liegt auch daran, dass ein entsprechender Schutz Systeme langsamer macht und deshalb nicht immer praktikabel wäre.

Zudem ist kein bösartiger Rowhammer-Angriff öffentlich bekannt, eine unmittelbare Gefahr und damit auch sofortiger Handlungsbedarf besteht also (scheinbar) nicht.



##### ECC

Die denkbar idealste Lösung für das Rowhammer-Problem wäre schlicht **bessere Chips** zu bauen. Ohne "schwache" Bits oder mit besserer Kontrolle haben Angreifer keinen Hebel.

Ein gutes Beispiel ist die bereits in manchen RAM Bausteinen verwendete **ECC-Methode**. Das Error-Corretion-Code-Verfahren verwendet einen Hashwert zur Identifikation von Ein- und Zwei- teilweise auch Mehr-Bit-Fehlern. Für dieses Verfahren werden 72 statt den üblichen 64 Bits für jede Speicherzelle benötigt.[[17]](17) Da auch ECC nicht sämtliche Fehler ausmerzt , bietet es aber weiterhin keinen optimalen Schutz. Auch-ECC-Systeme wurden bereits nachweiselich mit Rowhammering "geknackt".[[10]](10)

Trotzdem gilt: Ein bisschen mehr Schutz, ist besser als gar keiner. Einfache Rowhammer-Angriffe scheitern schon an DDR4-RAM, ein Upgrade schadet also nie :).



##### Refresh-Rate

Noch ein Ansatz für bessern Schutz wäre das **Erhöhen** der sogenannten **Refresh-Rate** innerhalb der Hardware.

In Kürze: D-RAM-Bits verlieren ihre Ladung müssen deshalb in sehr kurzen Zeitabständen ausgelesen- und wieder geladen werden. D-RAM-Bausteine werden dafür typischerweise alle paar dutzend Millisekunden aufgefrischt.[[14]](14)  Der Rowhammer-Angriff muss innerhalb einer Refresh-Periode abgeschlossen sein, da die Bits ansonsten wieder Ordnungsgemäss geladen sind.

Im Prinzip reicht die Zeit in einem typischen Refresh-Zyklus für mehr als eine Million Speicherinstruktionen. Wird die Refresh-Rate allerdings erhöht, sinkt auch die Chance darauf, dass ein "schwaches" Bit rechtzeitig nachgibt.

Problem hier: Eine höhere Refresh-Rate braucht extrem viel Leistung und schlägt deutlich auf die Performence des Systems (mehr Overhead). **(Rechnung maybe Grafik) [wo Zitat?] vllt auch 14**





### Fazit: Wo Frösche sind, da sind auch Störche

Stand heute ist noch kein gelungener "bösartiger" Hack mittels Rowhammer bekannt. Alle nachweislich funktionstüchtigen Angriffs-Methoden wurden von System-Entwicklern und Sicherheitsspezialisten genutzt, um die Schwachpunkte im Hauptspeicher nachzuweisen. Da entsprechende Attacken auf älteren Systemen allerdings volkommen unbemerkt ablaufen können, lässt sich ein Hack auch nicht vollkommen ausschliessen. Die Benken unter System-Sicherheitsexperten sind zudem real.[[10]](10) Gesetz dem Sprichwort im Titel, kommen wir zu dem Schluss, dass Chip- und System-Entwickler nicht erst auf ihren "Storch" warten sollten.



##### Für Interessierte: (Rowhammer@Home)

Google hat einen Row-Hammer-Test für x86-Computer als Open-Source-Projekt veröffentlicht. Der Test findet nur mit älteren Ram-Riegeln (DDR3) falsche Bits.[[0]](#0)

<img title="" src="file:///C:/Users/leder/AppData/Roaming/marktext/images/2022-04-28-21-05-28-image.png" alt="" width="547" data-align="center">





id="0">[0] Referenz 0 [GitHub - google/rowhammer-test: Test DRAM for bit flips caused by the rowhammer problem](https://github.com/google/rowhammer-test)

id="1">[1] Referenz 1[DRAM Circuit Design: Fundamental and High-Speed Topics - Brent Keeth, R. Jacob Baker, Brian Johnson, Feng Lin - Google Books](https://books.google.de/books?id=TgW3LTubREQC&pg=PA33&hl=de&source=gbs_toc_r&cad=3#v=onepage&q&f=false)
id="2">[2] Referenz 2[Towards Terabit Memories | Chips 2020](https://www.chips2020.net/chapter/towards-terabit-memories)

id="3">[3] Referenz 3  [[Chips 2020 | SpringerLink](https://link.springer.com/book/10.1007/978-3-642-23096-7)]

id="4">[4] Referenz 4 [[[1904.09724] RowHammer: A Retrospective](https://arxiv.org/abs/1904.09724)]

id="5">[5] Referenz 5 [[Flipping bits in memory without accessing them: An experimental study of DRAM disturbance errors | IEEE Conference Publication | IEEE Xplore](https://ieeexplore.ieee.org/document/6853210)]

id="6">[6] Referenz 6 [Into the Void: x86 Instruction Set Reference](https://c9x.me/x86/html/file_module_x86_id_30.html)

id="7">[7] Referenz 7 [Operating Systems: Three Easy Pieces](https://pages.cs.wisc.edu/~remzi/OSTEP/vm-paging.pdf)

id="8">[8] Referenz 8 [Project Zero: Exploiting the DRAM rowhammer bug to gain kernel privileges](https://googleprojectzero.blogspot.com/2015/03/exploiting-dram-rowhammer-bug-to-gain.html)

id="9">[9] Referenz 9 [mmap(2) - Linux manual page](https://man7.org/linux/man-pages/man2/mmap.2.html)

id="10">[10] Referenz 10 [Exploiting Correcting Codes: On the Effectiveness of ECC Memory Against Rowhammer Attacks | IEEE Conference Publication | IEEE Xplore](https://ieeexplore.ieee.org/document/8835222)

id="14">[14] Referenz 14 [Understanding Reduced-Voltage Operation  
in Modern DRAM Devices](https://ghose.web.illinois.edu/papers/17sigmetrics_voltron.pdfhttps://ghose.web.illinois.edu/papers/17sigmetrics_voltron.pdf)

id="15">15 Referenz 15 [Throwhammer: Rowhammer Attacks over the Network and Defenses](https://download.vusec.net/papers/throwhammer_atc18.pdf)

id="16">16 Referenz 16 [Rowhammer.js: A Remote Software-Induced
Fault Attack in JavaScript](https://arxiv.org/pdf/1507.06955.pdf)

id="17">17 Referenz 17 [Speichermodul – Wikipedia](https://de.wikipedia.org/wiki/Speichermodul#Fehlererkennung_(ECC)) **Bessere Quelle bitte!!!!**
