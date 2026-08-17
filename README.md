# Sparkasse Android App: Backup entschlüsseln

Dieses Repo dokumentiert die Entschlüsselung und den CSV-Export von Umsatzdaten aus Backups der Sparkassen App.

Hinweis: 95% KI-Generiert. Keine Eigenleistung außer Prompts. 

---

## Sicherheit

- **100% OFFLINE**: Das gesamte Vorgehen läuft ausschließlich lokal auf dem eigenen Rechner. Es werden keine Daten irgendwohin gesendet.
- **Kein fertiges Skript**: Bei sensiblen Bankdaten sollte man keine fertigen Skripte aus dem Internet blind ausführen. Stattdessen habe ich absichtlich nur einen KI-Prompt gebaut, den man einfach an einen KI Agent geben kann und sich ein kurzes, transparentes Python-Skript für das eigene System lässt.
- **Voller Datenschutz**: Der KI müssen weder Passwörter noch echte Backup-Dateien übergeben werden – die reine Spezifikation der Krypto-Struktur reicht völlig aus.

---

## Hintergrund

In meinen jungen Jahren war ich leider dum und hatte keine anderen Backups oder woanders die Historie gespeichert. comdirect löschte damals Finanzreporte nach einer gewissen Zeit regelmäßig aus der Postbox. Also war die Sparkassen-App leider der einzige Ort mit kompletter History.
Ich hatte über Jahre hinweg regelmäßig bei Star Finanz per Mail und sogar per Brief gebeten, eine Exportfunktion einzubauen, am Ende hatte ich sogar 1000 € (Peanuts lol) für deren Unterstützung geboten. Kam nichtmal eine Antwort außer wir schauen uns das an. Tja.
Naja, jedenfalls hat Gemini die Krypto-Struktur der Sicherung analysiert und jetzt endlich den Export ermöglicht. 

Dieses Projekt ist KI-generiert **ohne nennenswerte Eigenleistung** und wird hier geteilt, falls jemand anderes in derselben Situation feststeckt und an seine eigenen Daten gelangen möchte.

---

## Was das Ganze macht

Die offizielle Sparkassen-App speichert eingebundene Fremdbankkonten lokal in einer verschlüsselten SQLCipher-Datenbank. Während für Sparkassen-eigene Konten Exportfunktionen existieren, fehlt für Fremdbanken in der App jegliche Export-Möglichkeit.

Über die in der App integrierte Funktion **„Manuelle Sicherung erstellen“** erzeugt die App ein ZIP-Archiv. Aus diesem Archiv lässt sich mit dem vergebenen Passwort der Datenbankschlüssel mathematisch ableiten und der gesamte Buchungsverlauf als CSV exportieren.

---

## Voraussetzungen

- Smartphone: Jedes herkömmliche Android-Smartphone mit der Sparkassen-App. Kein Root-Zugriff und keine Modifikationen nötig.
- Sicherungsdatei: Die manuelle Sicherungsdatei aus der App (`sfinanzstatus...zip`).
- Passwort: Das Passwort, das beim Erstellen der Sicherung in der App vergeben wurde.
- PC / Mac: Ein Rechner mit Python (Windows, macOS, Linux).

---

## Schritte

1. **Sicherung in der App erstellen**:  
   In der Sparkassen-App auf **Profil -> Einstellungen -> Datensicherung -> Manuelle Sicherung erstellen** gehen, Passwort vergeben und die `.zip`-Datei speichern.
2. **Datei auf den PC kopieren**:  
   Die ZIP-Datei auf den Rechner übertragen.
3. **Prompt in die KI einfügen**:  
   Claude oder AGY im Ordner öffnen und den folgenden Prompt reinpasten. Die KI generiert daraus ein sauberes, lokales Python-Skript.
4. **Skript starten**:  
   Das generierte Skript ausführen, Sicherungsdatei und Passwort eingeben und die fertigen CSV-Tabellen erhalten.

---

## Prompt-Vorlage für die KI

Diesen Textblock einfach kopieren und an die KI übergeben:

```text
Ich möchte meine eigenen historischen Multibanking-Umsätze aus einer manuellen Sicherungsdatei der Android-App "Sparkasse Ihre mobile Filiale" (Dateiname: sfinanzstatus*.zip) lokal auf meinem Rechner als CSV exportieren.

Bitte erstelle mir ein eigenständiges Python-Skript, das 100 % offline arbeitet und folgende Spezifikation umsetzt:

1. Archiv-Struktur:
   Die ZIP-Datei enthält die Einstellungsdatei 'StarMoneyPrefs' sowie die Datenbank 'data.db'.

2. Parameter-Extraktion:
   In 'StarMoneyPrefs' befinden sich zwei Base64-Strings:
   - 'sf1': 16 Bytes Base64 (dient als Salt und AES-IV).
   - 'sf3': 48 Bytes Base64 (der mit AES-256 verschlüsselte Datenbankschlüssel).

3. Schlüsselableitung (2-stufig):
   - Stufe 1: Aus meinem Benutzer-Passwort und dem Salt 'sf1' wird via PBKDF2-HMAC-SHA1 mit 100.001 Iterationen ein 32-Byte AES-Schlüssel abgeleitet.
   - Stufe 2: Mit diesem abgeleiteten Schlüssel und dem IV 'sf1' wird 'sf3' per AES-256-CBC (PKCS7-Padding) entschlüsselt. Das Ergebnis ist ein Klartext-String ('dbAccess').

4. Datenbank-Entschlüsselung (SQLCipher v4):
   - Die Datei 'data.db' ist eine SQLCipher v4-Datenbank (Page Size: 4096 Bytes, Reserve: 80 Bytes).
   - Aus der Passphrase 'dbAccess' und den ersten 16 Bytes der Datei (Salt) wird via PBKDF2-HMAC-SHA512 mit 256.000 Iterationen der Seiten-Verschlüsselungsschlüssel berechnet.
   - Jede 4096-Byte-Seite wird per AES-256-CBC entschlüsselt (IV steht jeweils in den ersten 16 Bytes der 80 Reserve-Bytes).
   - Das Ergebnis ist eine standardkonforme SQLite3-Datenbank.

5. Export:
   - Das Skript soll die Tabelle 'giro_umsatz' (Girokonto) und 'kkumsatz' (Kreditkarte) aus der entschlüsselten Datenbank auslesen und als saubere, Semikolon-getrennte CSV-Dateien mit UTF-8-Encoding abspeichern.
   - Das Skript soll nach dem Pfad zur ZIP-Datei und nach dem Passwort fragen (z. B. via getpass maskiert), keine harten Pfade/Passwörter enthalten und keine Daten über das Netzwerk senden.
```
