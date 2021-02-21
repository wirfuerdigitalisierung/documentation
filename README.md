# Implementierungsleitfaden zur sicheren Übertragung von Kontaktnachverfolgungsdaten externer Dienstleister in die Infrastruktur der Darfichrein GmbH
----
## Abstract
Die Darfichrein GmbH hält alle personenbezogenen Daten der Gäste asymmetrisch verschlüsselt im BSI-zertifizierten Rechenzentrum der Anstalt für kommunale Datenverarbeitung in Bayern (AKDB). 
Die Darfichrein GmbH (darfichrein) möchte diesen Baustein nutzen, um ihn auch weiteren Kontaktdatenerfassungssystemen im Austausch mit den Gesundheitsbehörden zur Verfügung zu stellen.
darfichrein stellt hierfür eine RESTful API, welche den Austausch entsprechend konkretisiert und kapselt.
Die Arbeit erfolgt im Rahmen „wirfuerdigitalisierung“.

## Dokumenthistorie



## Code of conduct


## Ablauf
### Onboarding
Die Darfichrein GmbH sorgt für das Onboarding der deutschen Gesundheitsbehörden. Jede Behörde bekommt ein eigens für Sie innerhalb des Browsers generiertes Schlüsselpaar (RSA, 4096 Bit). Der öffentliche Schlüssel wird innerhalb der Telemetrie der Darfichrein GmbH persistiert.
### Daten anfordern
#### Szenario „Providerwahl“
Möchte eine Gesundheitsbehörde Kontaktdaten einer Destination erfragen, so wählt diese den entsprechenden Provider aus, welchen die Destination nutzt. 
#### Szenario „Volltext“
In diesem Szenario ist der Behörde der Provider von Kontaktdaten egal. Es werden alle angeschlossenen Systeme angefragt und durch Standortinformationen (Anschrift) kann der Mitarbeiter die entsprechende Destination wählen.
#### Szenariounabhängig
__unabhängig des Szenarios „Providerwahl“ oder „Volltext“__
Die Abfrage wird durch einen Zeitraum weiter spezifiziert. Anschließend wird ein HTTP-Request (POST) mit diesen Informationen, dem öffentlichen Schlüssel, sowie  dem Zertifikat der anfragenden Behörde und weiteren angereicherten Daten ausgelöst.
### Daten liefern
Ziel ist jetzt, den Kunden des Providers die Kontaktdaten zur Verfügung zu stellen - falls diese durch Public Key-Infrastruktur verschlüsselt sind. Andernfalls können die geforderten Daten direkt mit dem öffentlichen Schlüssel der Behörde verschlüsselt und an einen definierten HTTP-Endpunkt der Darfichrein GmbH übertragen werden.
### Daten exportieren und verarbeiten
Die Behörde als solche kann die Datensätze nun mit dem privaten Schlüssel entschlüsseln, exportieren und ins jeweilige System der Gesundheitsbehörde importieren.Exportformate können jederzeit erweitert werden.

## Definition Daten und Schnittstellen
### Grundsätzliches
Da diese Anfrage nur innerhalb der Anwendung der Gesundheitsbehörden stattfindet, werden keine Request bekannt werden.
Wir bitten dennoch, einen HTTP-Header (z. B. `x-wfd-api-key`) für einen API-Key für alle HTTP-Anfragen zu implementieren. Der API-Key wird vom Provider selbst gewählt und sollte nur Zugriff auf die nachfolgenden Ressourcen gewähren.
Die Spezifikation liegt im OpenAPI-Format unter [https://github.com/darfichrein-de/wfd-ga-digital] vor.

### Ressource: Standortanfrage
Die Standortanfrage ist von allen beteiligten Providern selbst zu implementieren und bereit zu stellen.
Wir plädieren für einen HTTP/GET-Request.

#### Use-Case
Als Mitarbeiter des Gesundheitsamtes möchte bei Eingabe von mindestens 4 Zeichen eine Suche nach Standorten auslösen. Als Ergebnis erwarte ich, dass ich alle an Provider angeschlossene Destinationen erhalte, die mit meinem Suchbegriff starten - unabhängig der Majuskeln bzw. Minuskeln (case-insensitive).

#### Definition der HTTP-Response
Die Definition der erwarteten Response ist dem OpenAPI-Dokument im Repository(fn) zu entnehmen.

### Ressource: Kontaktdatenanfrage
Die Kontaktdatenanfrage wird seitens „Gesundheitsbehörde digital“-Portal ausgelöst. Im Zuge dessen ist es wünschenswert, dass jeder Provider eine HTTP/POST-Anfrage zulässt, um die Anfrage innerhalb seines Systems aufzunehmen und zu verarbeiten.
#### Use-Case
Als Mitarbeiter des Gesundheitsamtes möchte ich eine Anfrage zur Kontaktdatenanforderung an den vorher festgelegten Standort senden können. Dieser muss die Informationen wie Zeitraum und ggf. Kontext der Kontaktdatenanfrage annehmen und entsprechend verarbeiten. Dabei ist es elementar, die Daten entsprechend des Zeitraums einzuschränken. Als Sicherheitskriterium möchte ich, dass das Zertifikat meines Gesundheitsamtes mitgesendet wird, um die Anfrage zu verifizieren.
#### Definition des HTTP-Requests
Die Definition des Request ist dem OpenAPI-Dokument im Repository(fn) zu entnehmen.

### Ressource: Kontaktdaten-Upload
Herzstück von Gesundheitsbehörde digital ist die Entgegennahme von Ende-zu-Ende-verschlüsselten Kontaktdateninformationen unterschiedlichster Provider.
#### Use-Case
Als Mitarbeiter des Gesundheitsamtes möchte ich, dass angefragte Kontaktdaten mit dem öffentlichen Schlüssel meiner Behörde verschlüsselt sind und einem Fall zugeordnet hochgeladen werden.
Dabei ist mir wichtig, dass alle Kontaktdatenanbieter die Struktur der persönlichen Daten homogen halten, um ein automatisches Einordnen in bestimmte Formate (z. B. CSV-Export) erlauben.
#### Definition des HTTP-Requests
Die Definition des Request ist dem OpenAPI-Dokument im Repository(fn) zu entnehmen.
