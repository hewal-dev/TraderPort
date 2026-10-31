# Händler-Funkreise – Machbarkeitsanalyse für 7 Days to Die 3.4

## Stand und Zusammenfassung

**Untersuchte Spielversion:** 3.4.0 (b9), Linux.  
**Status:** Read-only-Analyse der lokalen Spieldaten und des Spielcodes sowie des Archivs `TraderTeleport_V2.5_BeAst-9992-2-5-1774849742.zip`. Kein Prototyp, Spielstart oder Multiplayer-Livetest.

Die Idee ist grundsätzlich machbar: Eine einmalige persönliche Spezialquest schaltet das Funknetz frei. Danach kann jeder Spieler besuchte Händlerstandorte einzeln freischalten und gegen Bezahlung zwischen ihnen reisen. Die vorliegende Fremd-Mod ist eine technische Referenz, aber keine fertige Grundlage für diese Funktionen.

## Gewünschter Spielablauf

1. Ein Spieler nimmt bei einem Händler eine einmalige Spezialquest für das Funknetz an und schließt sie ab.
2. Der Spieler besucht Händler persönlich und schaltet deren konkrete Standorte frei.
3. An einem Händler-Funkgerät öffnet er ein Reisemenü.
4. Das Menü zeigt nur für diesen Spieler freigeschaltete Ziele und die Reisekosten.
5. Nach Bestätigung werden Dukes oder altes Geld abgezogen und der Spieler reist zum Ziel.

Die Quest ist die persönliche Grundfreischaltung; der persönliche Besuch schaltet jeweils einen konkreten Standort frei. Ob beide Währungen alternativ möglich sein sollen und wie hoch die Kosten sind, ist noch offen.

## Untersuchte Fremd-Mod: TraderTeleport

Das ZIP enthält eine kompilierte DLL, `Config.json`, `ModInfo.xml`, README und PDB, aber keinen Quellcode und keine erkennbare Lizenzdatei.

| Funktion | Befund |
|---|---|
| Auslösung | Chatbefehle wie `/trader jen` oder `/jen`; keine Funkgerätbedienung |
| Ziele | Manuell in der Konfiguration hinterlegte Koordinaten |
| Startbedingung | Standardmäßig innerhalb eines Radius von 35 Blöcken um eine konfigurierte Händlerposition |
| Wartezeit | Standardmäßig 30 Sekunden |
| Blutmond | Standardmäßig gesperrt |
| Zahlung | Nicht vorhanden |
| Teleport | Die DLL führt den Konsolenbefehl `teleportplayer` aus |
| Version | Ausführung und Kompatibilität mit 3.4 nicht im Spiel getestet |

Für die geplante Mod wäre eine eigenständige Umsetzung nach diesem Vorbild sinnvoll. Ob Code übernommen oder weitergegeben werden darf, ist ohne geklärte Rechte nicht belegt.

## Funkgeräte und Händler

### Funkgeräte

Der Vanilla-Block `radioHam` ist in `Data/Config/blocks.xml` definiert. Die Blockpaletten aller fünf untersuchten Vanilla-Händlergebäude – Bob, Hugh, Jen, Joel und Rekt – enthalten diesen Blocktyp. Die Paletten belegen aber weder die exakte Anzahl noch die genauen Positionen der Geräte.

`radioHam` kommt auch in anderen Gebäuden vor. Deshalb darf nicht pauschal jedes Funkgerät die Reisefunktion erhalten. Die Mod muss zusätzlich prüfen, ob sich das Gerät auf einem zulässigen Händlergelände befindet. Sonderhändler, Overhauls und fremde Händler-POIs sind gesondert zu prüfen.

### Händlerstandorte

Das Spiel verwaltet Händlergebiete und bietet unter anderem `World.GetTraderAreaAt` an. Damit gibt es eine Grundlage, Händlergebiete anhand der tatsächlichen Welt statt ausschließlich über manuell eingetragene Koordinaten zu erkennen.

Die Freischaltung sollte sich auf **einen konkreten Standort**, nicht nur auf einen Händlernamen beziehen: Eine Welt kann mehrere Standorte desselben Händlertyps enthalten. Im Menü lassen sie sich beispielsweise anhand von Biom, Entfernung oder Koordinaten unterscheiden.

**Offen:** Welche Kennung für jeden Standort über Neustarts, bestehende Spielstände und verschiedene Weltvarianten zuverlässig stabil bleibt, muss praktisch geprüft werden.

## Persönlicher Besuch und Freischaltung

Das Spiel erkennt beim Betreten eines Händler-POIs bereits den Besuch und erstellt normalerweise einen persönlichen Kartenmarker. Das ist ein hilfreiches Vanilla-Vorbild.

Es unterscheidet dabei aber nicht zuverlässig zwischen eigener Anreise und Ankunft per Teleport. Ein Vanilla-Kartenmarker allein ist daher kein lückenloser Nachweis eines manuellen Besuchs; außerdem kann ein Spieler Marker möglicherweise entfernen.

**Möglicher Ansatz:** Eine eigene Besuchsliste pro Spieler und Standort führen. Als bewusst sichtbare Aktion könnte der Spieler den Standort durch die erste Bedienung des Funkgeräts freischalten. Soll auch die Ankunft durch fremde Teleports ausgeschlossen werden, braucht es zusätzliche Prüfung. Dass sie alle Fälle lückenlos erkennt, ist nicht belegt.

### Bestehende Spielstände

Bereits vorhandene Händler-Kartenmarker könnten Hinweise auf frühere Besuche liefern, sind aber keine vollständig verlässliche Besuchsliste. Die einfache und nachvollziehbare Variante wäre, Händler ab Installation der Mod durch Bedienung ihres Funkgeräts freizuschalten. Eine Übernahme früherer Besuche ist eine offene Designentscheidung.

## Einmalige Spezialquest

Für eine einmalige Freischaltquest existiert ein Vanilla-Vorbild:

- `Data/Config/quests.xml`: Die Quest `intro_buried_supplies` ist nicht wiederholbar und bei allen fünf Händlern eingetragen.
- Beim Abschluss setzt eine `SetCVar`-Aktion den persönlichen Wert `IntroComplete` auf `1`.
- Dieses Muster kann als Grundlage für eine eigene persönliche Funknetz-Freischaltung dienen.

**Ideenbeispiel:** „Das Funknetz wiederherstellen“ – elektronische Bauteile und weitere Materialien besorgen, die Funkverbindung reparieren und beim Händler melden. Questinhalt, Voraussetzungen, Belohnung und tatsächlicher Ablauf sind noch nicht festgelegt oder getestet.

## Bezahlung

In `Data/Config/items.xml` sind beide gewünschten Zahlungsmittel vorhanden:

- `casinoCoin` – Dukes
- `oldCash` – altes Geld

Möglich wären feste oder entfernungsabhängige Preise sowie getrennte Preise je Zahlungsmittel. Für einen ersten Prototyp empfiehlt sich ein **fester, konfigurierbarer Preis mit zunächst einer Währung**.

### Multiplayer-Risiken

Rucksack und Werkzeuggürtel, Abzug und Teleport müssen korrekt zwischen Client und Server abgestimmt werden. Die Codeanalyse liefert Hinweise, dass der Server sich bei entfernten Spielern nicht einfach auf einen stets aktuellen Spiegel aller Gürtelplätze verlassen sollte. Die konkrete Transaktion ist noch zu entwickeln und zu testen.

Mindestens zu prüfen:

- Geld reicht exakt oder reicht nicht.
- Geld ist auf Rucksack und Werkzeuggürtel verteilt.
- Ein Doppelklick verursacht keinen doppelten Abzug.
- Kein Verlust bei ungültigem Ziel, Abbruch oder fehlgeschlagenem Vorgang.
- Es wird nur aus dem Inventar des tatsächlich reisenden Spielers abgezogen.
- Mehrere gleichzeitige Spieler und Verbindungsabbrüche während der Reise.

Eine vollständig fälschungssichere Server-Transaktion ist bislang nicht nachgewiesen.

## Sichere Ankunft und Regeln

Die Fremd-Mod verwendet feste Koordinaten. Für ein automatisches Händlernetz müssen geeignete Ankunftspunkte stattdessen bestimmt oder eingerichtet werden, damit Spieler nicht in Blöcken, auf Dächern oder an unzugänglichen Stellen landen.

Das Spiel kennt Händlerbereiche und Teleport-Volumes. Ob diese sich als sichere Ankunftspunkte eignen, ist nicht allein aus ihrer Existenz bewiesen und muss getestet werden. Ebenfalls offen:

- Reisen während Händleröffnungszeiten oder auch bei geschlossenem Händler?
- Reisen während des Blutmonds?
- Wartezeit zwischen Reisen?
- Reisen mit Fahrzeug, Drohne oder offenem Inventarfenster?
- Verhalten bei blockiertem Ziel, nicht geladenem Bereich oder Verbindungsabbruch?

## Server und Clients

Die Original-Mod fängt Chatbefehle serverseitig ab. Die gewünschte E-Interaktion mit einem richtigen Funkgerät-Menü beginnt dagegen über die lokale Spieleroberfläche.

Der naheliegende Ansatz ist daher:

- **Server:** Regeln, Händlerziele, persönliche Freischaltungen und Prüfung der Reise.
- **Client:** Bedienung und Anzeige am Funkgerät sowie abgestimmter Inventar-/Zahlungspfad.

Eine Installation auf Server **und** Clients ist damit voraussichtlich erforderlich. Ein rein serverseitiger Sonderweg ist nicht abschließend ausgeschlossen, wäre aber zu untersuchen und könnte eine andere oder eingeschränkte Bedienung bedeuten.

## Vorschlag für einen ersten Prototyp

1. Eine persönliche, einmalige Quest schaltet die Nutzung des Funknetzes frei.
2. Funkgerät am Händler aktivieren und konkreten Standort pro Spieler speichern.
3. Nur persönlich freigeschaltete Ziele anzeigen.
4. Einen festen konfigurierbaren Preis in einer Währung berechnen.
5. Sichere Ankunft und Zahlungsablauf testen – zuerst allein, anschließend mit mindestens zwei Spielern auf einem Dedicated Server.
6. Danach zweite Währung, Preisvarianten und zusätzliche Spielregeln ergänzen.

## Prüfstand und Quellen

**Bestätigt durch lokale Dateien bzw. Analyse des Spielcodes:** Block- und Itemnamen, Funkgerät-Einträge in den Paletten der fünf genannten Händlergebäude, Vanilla-Questmuster, Vorhandensein von Händlergebieten und -Positionsabfragen sowie Funktionsweise der Original-DLL.

**Noch nicht bestätigt:** Funktion im Spiel, exakte Funkgerätpositionen, automatische Standortkennung für alle Welten, sichere Ankunft, zuverlässige Erkennung jeder manuellen Anreise und Multiplayer-Transaktion.

Wesentliche lokale Quellen:

- `Data/Config/blocks.xml` – `radioHam`.
- `Data/Config/items.xml` – `casinoCoin` und `oldCash`.
- `Data/Config/quests.xml` – `intro_buried_supplies`, `SetCVar`, Händler-Questlisten.
- `Data/Prefabs/POIs/trader_*.xml` und zugehörige `.blocks.nim`-Dateien – Händlergebiete und Blockpaletten.
- `7DaysToDie_Data/Managed/Assembly-CSharp.dll` – analysierte Spiel-APIs und Abläufe.
- Originalarchiv `TraderTeleport_V2.5_BeAst-9992-2-5-1774849742.zip` – README, Konfiguration und kompilierte DLL.
