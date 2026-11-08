# SPEC.md – Wolff's Naturkosmetik

## 1. Quelle und Projektumfang

**Kundin:** Nina Wolff\
**Unternehmen:** Wolff's Naturkosmetik\
**Kundengruppe:** K6\
**Interview:** Meeting 1 (12 Nachrichten) und Meeting 2 (8 Nachrichten)\
**Quelle:** Interview mit dem KI-Kunden Nina Wolff über insgesamt 20 Nachrichten.

Ziel ist eine Anwendung zur zentralen Verwaltung von Kunden, Bestellungen, Warenbeständen, Chargen und Wiederverkäuferkonditionen. Sie soll insbesondere die bisherige Verwaltung über Instagram-DMs und mehrere Excel-Tabellen verbessern.

## 2. Entitäten

### 2.1 Kunde

| Attribut       | Beschreibung                      |
| -------------- | --------------------------------- |
| kunden_id      | Eindeutige Kunden-ID              |
| name           | Name des Kunden                   |
| kundentyp      | Privatkunde oder Wiederverkäufer  |
| email          | E-Mail-Adresse                    |
| kontakt_handle | Instagram-Handle, falls vorhanden |

### 2.2 Produkt

| Attribut       | Beschreibung                               |
| -------------- | ------------------------------------------ |
| produkt_id     | Eindeutige Produkt-ID                      |
| name           | Produktname                                |
| b2c_preis      | Regulärer Verkaufspreis                    |
| allergen_info  | Informationen zu relevanten Inhaltsstoffen |
| mindestbestand | Individuell festgelegter Mindestbestand    |

### 2.3 Bestellung

| Attribut      | Beschreibung                            |
| ------------- | --------------------------------------- |
| bestellung_id | Eindeutige Bestell-ID                   |
| bestell_datum | Datum der Bestellung                    |
| verkaufskanal | Instagram, Website, E-Mail oder Messe   |
| bestellstatus | Aktueller Status der Bestellung         |
| bestaetigt_am | Zeitpunkt der verbindlichen Bestätigung |

### 2.4 Bestellposition

| Attribut     | Beschreibung                 |
| ------------ | ---------------------------- |
| position_id  | Eindeutige Positions-ID      |
| menge        | Bestellte Produktmenge       |
| einzelpreis  | Vereinbarter Preis pro Stück |
| preisart     | B2C, B2B oder Restposten     |
| offene_menge | Noch nicht erfüllte Menge    |

### 2.5 Charge

| Attribut                 | Beschreibung                    |
| ------------------------ | ------------------------------- |
| charge_id                | Eindeutige Chargen-ID           |
| herstellungsdatum        | Datum der Herstellung           |
| mindesthaltbarkeitsdatum | Mindesthaltbarkeitsdatum        |
| produzierte_menge        | Ursprünglich hergestellte Menge |
| verfuegbare_menge        | Aktuell verfügbare Menge        |

### 2.6 Konditionsvereinbarung

| Attribut             | Beschreibung             |
| -------------------- | ------------------------ |
| kondition_id         | Eindeutige Konditions-ID |
| mindestbestellmenge  | Vereinbarte Mindestmenge |
| rabatt               | Vereinbarter B2B-Rabatt  |
| zahlungsziel_tage    | Zahlungsfrist in Tagen   |
| gueltigkeitszeitraum | Zeitraum der Gültigkeit  |

### 2.7 Rechnung

| Attribut          | Beschreibung                          |
| ----------------- | ------------------------------------- |
| rechnung_id       | Eindeutige Rechnungs-ID               |
| rechnungsdatum    | Datum der Rechnungserstellung         |
| faelligkeitsdatum | Datum der Zahlungserwartung           |
| gesamtbetrag      | Gesamter Rechnungsbetrag              |
| zahlungsstatus    | Offen, teilweise bezahlt oder bezahlt |

### 2.8 Zahlung

| Attribut      | Beschreibung               |
| ------------- | -------------------------- |
| zahlung_id    | Eindeutige Zahlungs-ID     |
| eingangsdatum | Datum des Zahlungseingangs |
| betrag        | Eingegangener Betrag       |
| zahlungsart   | Art der Zahlung            |

## 3. Beziehungen und Kardinalitäten

| Nr. | Beziehung                           | Kardinalität | Beschreibung                                                                                                      |
| --- | ----------------------------------- | ------------ | ----------------------------------------------------------------------------------------------------------------- |
| 1   | Kunde – Bestellung                  | 1:n          | Ein Kunde kann mehrere Bestellungen aufgeben. Jede Bestellung gehört zu einem Kunden.                             |
| 2   | Bestellung – Bestellposition        | 1:n          | Eine Bestellung enthält eine oder mehrere Bestellpositionen.                                                      |
| 3   | Produkt – Bestellposition           | 1:n          | Ein Produkt kann in vielen Bestellpositionen vorkommen.                                                           |
| 4   | Produkt – Charge                    | 1:n          | Ein Produkt kann in mehreren Chargen produziert werden.                                                           |
| 5   | Bestellposition – Charge            | n:m          | Eine Bestellposition kann aus mehreren Chargen beliefert werden. Eine Charge kann mehrere Bestellungen beliefern. |
| 6   | Kunde – Konditionsvereinbarung      | 1:n          | Ein Wiederverkäufer kann mehrere zeitlich unterschiedliche Konditionsvereinbarungen besitzen.                     |
| 7   | Konditionsvereinbarung – Bestellung | 1:n          | Eine Konditionsvereinbarung kann für mehrere Bestellungen gelten.                                                 |
| 8   | Bestellung – Rechnung               | 1:1          | Eine Bestellung wird einer Rechnung zugeordnet. Dies ist eine vorläufige Modellierungsannahme.                    |
| 9   | Rechnung – Zahlung                  | 1:n          | Eine Rechnung kann durch mehrere Teilzahlungen beglichen werden.                                                  |

## 4. Business Rules

### BR-01: Verbindliche Bestellung und Reservierung

Bei einer Website-Bestellung wird die Bestellung durch den Klick auf „Bestellung abschicken“ verbindlich. Bei Instagram-DM-Bestellungen erhält die Kundin einen Bestätigungs-Link, der 24 Stunden gültig ist. Erst nach der Bestätigung wird die Bestellung verbindlich und die Ware reserviert. Bei relevanten Inhaltsstoffen muss vorher die Allergen-Information bestätigt werden. Für Privatkundinnen beginnt die Zahlungsfrist von 48 Stunden erst mit der verbindlichen Bestätigung. Erfolgt keine Bestätigung innerhalb von 24 Stunden, wird keine Ware reserviert.

**Quelle:** Meeting 1, Nachrichten 3–5.

### BR-02: Zentraler Warenbestand

Der Warenbestand muss über alle Verkaufskanäle zentral und unmittelbar aktualisiert werden. Wenn eine Bestellung die verfügbare Menge übersteigt, wird die vorhandene Menge reserviert. Die fehlende Menge wird als offener Bedarf angezeigt. Nina entscheidet über Nachproduktion und Liefertermin.

**Quelle:** Meeting 1, Nachrichten 7–8.

### BR-03: Chargenverwaltung und Mindesthaltbarkeit

Bei der Belieferung einer Bestellung wird automatisch zuerst die Charge mit dem frühesten Mindesthaltbarkeitsdatum verwendet (FEFO-Prinzip). Die Zuordnung zwischen Kundin, Bestellposition und Charge muss nachvollziehbar sein. Chargen mit weniger als 30 Tagen Restlaufzeit sollen als mögliche Restposten erkannt werden. Den Rabatt legt Nina selbst fest.

**Quelle:** Meeting 1, Nachrichten 9–11.

### BR-04: Individuelle Wiederverkäuferkonditionen

Für jeden Wiederverkäufer gelten individuell gespeicherte Konditionen. Die Mindestbestellmenge bezieht sich auf die Gesamtmenge aller Produkte einer Bestellung. Wird sie erreicht, gelten die vereinbarten B2B-Konditionen. Wird sie unterschritten, gilt automatisch der B2C-Preis. Konditionsänderungen gelten ab ihrem Gültigkeitsdatum nur für neue Bestellungen. Bereits eingegangene Bestellungen behalten die zuvor gültigen Konditionen.

**Quelle:** Meeting 2, Nachrichten 13–15.

### BR-05: Stammkundinnenstatus

Eine Privatkundin erhält den Stammkundinnenstatus, wenn sie mindestens sechs vollständig bezahlte und abgeschlossene Bestellungen innerhalb der letzten 365 Tage erreicht. Stornierte, unbezahlte und zurückgesendete Bestellungen werden nicht berücksichtigt. Der Status bleibt nach Unterschreiten der Schwelle noch 30 Tage bestehen und endet danach automatisch. Nina kann ihn in begründeten Ausnahmefällen manuell verlängern.

**Quelle:** Meeting 2, Nachrichten 18–20.

## 5. Widersprüche und Regelkonflikte

### Widerspruch 1: Verbindlichkeit einer Instagram-DM-Bestellung

**Ursprüngliche Aussage:** Nina betrachtet eine Nachricht wie „Ja, ich nehme zwei“ zunächst als verbindliche Bestellung und möchte die Ware direkt reservieren.

**Spätere Aussage:** Nina stellt fest, dass eine solche Nachricht nicht eindeutig verbindlich ist. Sie möchte stattdessen einen Bestätigungs-Link versenden, über den die Kundin ihre Bestellung ausdrücklich bestätigt.

**Auflösung:** Die spätere Konkretisierung wird übernommen. Eine Instagram-DM allein löst keine Reservierung aus. Erst die ausdrückliche Bestätigung über den Link macht die Bestellung verbindlich.

**Quelle:** Meeting 1, Nachrichten 2–4.

### Widerspruch 2: Zahlungsziel bei der Wiederverkäuferin „Grüne Ecke“

**Ursprüngliche Vereinbarung:** Für „Grüne Ecke“ wurde zunächst ein Zahlungsziel von 30 Tagen genannt.

**Spätere Vereinbarung:** Nina erklärte später, dass für „Grüne Ecke“ stattdessen ein Zahlungsziel von 14 Tagen gegen einen zusätzlichen Rabatt von 2 % vereinbart wurde.

**Auflösung:** Die spätere Vereinbarung ersetzt die frühere ab ihrem vereinbarten Gültigkeitsdatum. Das System muss Konditionsänderungen mit Gültigkeitszeitraum speichern. Bereits aufgegebene Bestellungen behalten die zum Bestellzeitpunkt gültigen Konditionen.

**Quelle:** Meeting 2, Nachrichten 13–15.

Hinweis zur Konditionshistorie: Für „Grüne Ecke“ ist die spätere Vereinbarung mit 14 Tagen Zahlungsziel und zusätzlichem Rabatt von 2 % maßgeblich. Frühere Konditionen bleiben für bereits aufgegebene Bestellungen nachvollziehbar.\n\n## 6. Prioritäten

### Version 1 – Kernfunktionen

| Funktion                        | Priorität | Begründung                                            |
| ------------------------------- | --------- | ----------------------------------------------------- |
| Kunden- und Bestellverwaltung   | Hoch      | Bestellungen gehen aktuell in Instagram-DMs verloren. |
| Verbindliche Bestellbestätigung | Hoch      | Klare Reservierung und Zahlungsfristen.               |
| Zentrale Bestandsverwaltung     | Hoch      | Vermeidung von Überverkäufen über mehrere Kanäle.     |
| Chargenverwaltung               | Hoch      | Mindesthaltbarkeit und Rückverfolgbarkeit.            |
| FEFO-Prinzip                    | Hoch      | Ältere Chargen zuerst verkaufen.                      |
| Wiederverkäuferkonditionen      | Hoch      | Automatische Preisberechnung und Vertragshistorie.    |
| Rechnungen und Teilzahlungen    | Mittel    | Kontrolle offener B2B-Zahlungen.                      |
| Stammkundinnenprogramm          | Mittel    | Automatische Erkennung und Treuerabatte.              |

### Spätere Erweiterungen

- Automatische Vorschläge für Mindestbestände anhand bisheriger Verkäufe.
- E-Mail-Vorlagen und Kontaktmöglichkeiten für abgelaufene Bestätigungs-Links.
- Automatisierte Benachrichtigungen für neue Chargen und Restposten an Stammkundinnen.
- Erweiterte Auswertungen zum Kundenverhalten.
- Vorschläge für Restpostenrabatte und individuelle Verkaufsaktionen.

## 7. Neue Anforderung aus Meeting 2

### Automatisches Stammkundinnenprogramm

Die App soll Stammkundinnen anhand ihrer Bestellhistorie automatisch erkennen. Ab sechs bezahlten und abgeschlossenen Bestellungen innerhalb der letzten 365 Tage erhält eine Kundin den Stammkundinnenstatus.

Damit verbunden sind:

- 10 % Treuerabatt.
- 24 Stunden früherer Zugang zu neuen Chargen.
- Vorrangige Information über Restposten.
- Eine Anzeige von Kundinnen, die bereits vier oder fünf Bestellungen innerhalb der letzten 365 Tage erreicht haben.

Der Status bleibt nach Unterschreiten der Bestellschwelle noch 30 Tage bestehen. Nina kann ihn in Ausnahmefällen manuell verlängern.

**Quelle:** Meeting 2, Nachrichten 17–20.

## 8. Offene Fragen

1. Wie sollen Rücksendungen und Erstattungen nach dem Versand im Detail abgewickelt werden?
2. Wie werden Instagram-DMs und E-Mails technisch in die Anwendung eingebunden?
3. Wie wird der tatsächliche Lagerbestand bei Verkäufen auf Messen unmittelbar aktualisiert?
4. Wird der 10-%-Treuerabatt zusätzlich zu einem Restpostenrabatt gewährt oder gilt bei Restposten ausschließlich der von Nina festgelegte Sonderpreis?
5. Wie werden die Kündigungsfristen der Wiederverkäuferverträge im Einzelnen gespeichert und überwacht?
6. Wie sollen Produkte mit mehreren relevanten Allergenen und deren Bestätigungen dokumentiert werden?
7. Wie werden Produktionsmengen und voraussichtliche Fertigstellungstermine für Nachproduktionen verwaltet?

---

**Ende der Spezifikation – Wolff's Naturkosmetik**