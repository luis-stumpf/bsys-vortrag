##### Throwhammer eine Rowhammerattacke über das Netzwerk

Zielobjekte einer solchen Attacke sind meinst Cloudservices da auf diesen meins sensible Daten vorhanden sind und sie in den meisten Fällen auch die Voraussetzungen für eine Thorwhammer Attacke aufweisen.

Um eine Throwhammer Attacke durchzuführen zu können muss das Zielsystem über eine Netzwerkanbindung mit mindest Geschwindigkeit von ca 10 Gbps. Dies is jedoch heutzutage keine Seltenheit mehr. Zudem könnte es zu Poblemen kommen wenn der im System verwendete Speicher über eine ECC-Funktion verfügt. Nun braucht der Angreifer nur noch einen normalen Userzugang ohnen besondere Rechte.

Auf der Serversseite allokiert der Angreifer nun einen Zusammenhängenden Spreicher und füllte ständigt wiederholend mit nur Nullen oder nur Einsen. Während dessen wird jeweils überprüft ob ein Bit dieser allokierten Stellen geflippt ist.[[15]](15) Sobald dieser prozess abgeschlossen is werden die Bitfilps überprüft und es ist möglich eine Rowhammerattacke durch zuführen. Um mehr über Throwhammer zu erfahren Lesen Sie[[15]](#15)

##### Jackhammer eine Rowhammerattacke über eine Webseite (Rowhammer.js)

Die Programmiersprache Javascript hat zunächste keine möglichkeit Pointer und virtuelle Adressen an zusprechen. Also wie kommt man nun dazu direkt Speicher im Memory zuallokieren? Die Lösung hier für ist das sehr große Arrays von allen aktuellen Google Chrome und Firefox auf einem Linux System als  2MB pages allokiert werden. Dies Liegt am Betriebsystem und wird bei jeder Scriptsprache so angewendet.[[16]](16) Jetzt muss nurnoch wiederholt in hoher Frequenz über die Arrays iteriert und wir bekommen wieder unser Bit Flips. Ab hier kann man wieder nach dem Konzept des normalen Rowhammers vorgehen und das System nach belieben angreifen. Der springende Punkt an Rowhammer.js Attacken ist, dass diese zunächst nicht zuerkennen sind und direkt ausgeführt werden können so bald man eine bösartige Webseite aufruft. Zu verhindern ist die indem man ein Browser Addon verwendet das standart mäßig JavaScript unbekannter Webseiten deaktiviert. Für mehr Details siehe.[[16]](16)

(Two side attacks)

#### Wie schütz man Systemen gegen Rowhammerattacken

Eine Möglichkeit diese Schwachstelle anzugehen wäre ***bessere Chips*** zubauen die ein grundlegend anderes Design aufweisen.

Eine Bereits in manchen RAM Bausteinen verwendete Methode ist ***ECC*** (Error Corretion Code) dieses Verfahren verwendet einen Hashwert zur Identifikation von 1 und 2 bit fehlern teils auch mehr Bit jedoch werden dafür 72 Bits und nicht 64 Bits für jede Speicherzelle benötigt.[[17]](17) Da dabei aber nicht alle Fehler behoben wärden können, da dass erkennen von Mehr-bit-fehlern nicht immer funktioniert ist das auch keine optimale Lösung.

Noch ein ansatz wäre das ***erhöhen der Refresh Rate*** dies braucht jedoch extrem viel Leistung und schlägt extrem auf die Performence des Systems. (Rechnung maybe Grafik)

Man könnte auch direkt in der ***Produktion des RAMs eingreifen und sie auf verwundbare zellen Prüfen*** lassen dies könnte jedoch mehrere Tage dauern und wäre ökonomisch einer Katastrophe. Falls man diese entdeckt könnte man die Kaputten zellen auf so Ersatzzellen verlegen.

Zudem wäre es möglich dem ***Benutzer ein Programm mit auszuhändigen um das ganze selbst zumachen*** um sich vor einer Rowhammerattacke zuschützen.

Da druch das erhöhen der Refresh Rate Rowhammer attacken verhindert werden könnte man immer die ***benachbarten Zeilen von häufig genutzten Zeilen refreshen*** um in ihnen die warscheinlichkeit eines Bitflips zuverhindern.

PARA

id="15">15 Referenz 15 [Throwhammer: Rowhammer Attacks over the Network and Defenses](https://download.vusec.net/papers/throwhammer_atc18.pdf)

id="16">16 Referenz 16 [Rowhammer.js: A Remote Software-Induced
Fault Attack in JavaScript](https://arxiv.org/pdf/1507.06955.pdf)

id="17">17 Referenz 17 [Speichermodul – Wikipedia](https://de.wikipedia.org/wiki/Speichermodul#Fehlererkennung_(ECC)) **Bessere Quelle bitte!!!!**
