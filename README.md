# Gigachad - HackMyVM (Easy) - Writeup

![Gigachad Icon](Gigachad.png)

Dieses Repository enthält einen zusammengefassten Bericht über die Kompromittierung der HackMyVM-Maschine "Gigachad" (Schwierigkeitsgrad: Easy).

## Metadaten

*   **Maschine:** Gigachad (HackMyVM - Easy)
*   **Link zur VM:** [https://hackmyvm.eu/machines/machine.php?vm=Gigachad](https://hackmyvm.eu/machines/machine.php?vm=Gigachad)
*   **Autor des Writeups:** DarkSpirit
*   **Datum:** 10. Oktober 2022
*   **Original Writeup:** [https://alientec1908.github.io/Gigachad_HackMyVM_Easy/](https://alientec1908.github.io/Gigachad_HackMyVM_Easy/)

## Zusammenfassung des Angriffspfads

Die initiale Erkundung mit `nmap` offenbarte einen FTP-Dienst (Port 21, vsftpd) mit erlaubtem anonymen Login, einen SSH-Dienst (Port 22) und einen Apache-Webserver (Port 80). Auf dem FTP-Server wurde eine Datei `chadinfo` gefunden. Die Web-Enumeration mit `gobuster` fand unter anderem eine Bilddatei `/drippinchad.png`.

Die Analyse der `chadinfo`-Datei (via `lftp cat` oder Download/`file`) zeigte, dass es sich um ein ZIP-Archiv handelte (Implizit: Download und Entpacken). Die Steganographie-Analyse der `/drippinchad.png`-Datei mittels `stegsnow` extrahierte Passwort-Hinweise (`yyynbs`, `ynsdb0`). Diese Hinweise wurden genutzt, um mit `hydra` erfolgreich das SSH-Passwort (`maidenstower`) für den Benutzer `chad` zu knacken.

Dies ermöglichte den initialen Zugriff via SSH als `chad`. Die lokale Enumeration mittels `find` deckte ein SUID-gesetztes Binary `/usr/lib/s-nail/s-nail-privsep` auf. `searchsploit` identifizierte einen bekannten Local Privilege Escalation Exploit (CVE-2017-5899, `47172.sh`) für diese `s-nail`-Version.

Das Exploit-Skript wurde heruntergeladen, für die Nutzung im `/tmp`-Verzeichnis angepasst, auf das Zielsystem übertragen und ausführbar gemacht. Die Ausführung des Exploits in einer Schleife (`while true; do ./exploit.sh; done`) nutzte erfolgreich eine Race Condition aus und erzeugte eine Root-Shell.

Anschließend wurden die User-Flag (`/home/chad/user.txt`) und die Root-Flag (an einem ungewöhnlichen Ort: `/usr/include/root.txt`) ausgelesen.

## Verwendete Tools (Auswahl)

*   `arp-scan`
*   `nmap`
*   `lftp`
*   `cat`
*   `gobuster`
*   `stegsnow`
*   `hydra`
*   `ssh`
*   `find`
*   `searchsploit`
*   `wget`
*   `vi`
*   `chmod`
*   `bash` (while loop)
*   `id`
*   `pwd`
*   `cd`
*   `ls`
*   `./exploit.sh` (CVE-2017-5899 Exploit)
*   (Implied: `unzip` / `7z`)

## Angriffsschritte (Übersicht)

1.  **Reconnaissance:** Ziel-IP (`arp-scan`), Portscan (`nmap`) -> Port 21 (FTP, anon, `chadinfo`), Port 22 (SSH), Port 80 (HTTP).
2.  **Information Disclosure:**
    *   FTP (`lftp`): `chadinfo` ist ZIP-Datei (implizit: herunterladen, entpacken).
    *   Web (`gobuster`): `/drippinchad.png` gefunden.
    *   Stego (`stegsnow drippinchad.png`): Passwort-Hinweise `yyynbs`, `ynsdb0` extrahiert.
3.  **Password Crack:** Passwortliste aus Hinweisen erstellen. `hydra` gegen SSH-Benutzer `chad` -> Passwort `maidenstower` gefunden.
4.  **Initial Access:** Login via `ssh chad@gig.hmv` mit gefundenem Passwort.
5.  **Privilege Escalation Vector:** Lokale Enumeration (`find`) -> SUID-Binary `/usr/lib/s-nail/s-nail-privsep` gefunden.
6.  **Exploit Identification:** `searchsploit s-nail` -> Exploit `47172.sh` (CVE-2017-5899) gefunden.
7.  **Exploit Preparation:** Exploit herunterladen (`wget`), Pfade auf `/tmp` anpassen (`vi`), auf Ziel nach `/tmp` übertragen (`wget` von lokalem Server), ausführbar machen (`chmod +x`).
8.  **Privilege Escalation Execution:** Exploit in Schleife ausführen (`while true; do ./exploit.sh; done`) -> Race Condition erfolgreich -> Root-Shell.
9.  **Flags:** User-Flag (`cat /home/chad/user.txt`) und Root-Flag (`cat /usr/include/root.txt`) auslesen.

## Flags

*   **User Flag (`/home/chad/user.txt`):** `0FAD8F4B099A26E004376EAB42B6A56A`
*   **Root Flag (`/usr/include/root.txt`):** `832B123648707C6CD022DD9009AEF2FD`

---

## Disclaimer

Die hier dargestellten Informationen und Techniken dienen ausschließlich Bildungs- und Forschungszwecken im Bereich der Cybersicherheit. Die beschriebenen Methoden dürfen nur auf Systemen angewendet werden, für die eine ausdrückliche Genehmigung vorliegt (z.B. in CTF-Umgebungen wie HackMyVM, Penetrationstests mit schriftlicher Erlaubnis). Das unbefugte Eindringen in fremde Computersysteme ist illegal und strafbar. Die Autoren übernehmen keine Haftung für missbräuchliche Verwendung der bereitgestellten Informationen. Handeln Sie stets legal und ethisch verantwortlich.

---
