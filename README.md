# DNS-Server-mit-Firewall-Marke-Eigenbau

# DNS-Server + Firewall im Heimnetz (2 PCs) – Lernprojekt 

## Kurz erklärt
In diesem Projekt richte ich auf einem **zweiten PC** (Linux) einen **DNS-Server** ein und sichere ihn mit einer **Firewall** ab.  
Ziel: Grundlagen verstehen, sauber dokumentieren und testen – ohne Profi-Fachchinesisch.



---

## Setup (2 PCs)
- **PC 1 = dein normaler PC** (Windows oder Linux) → damit testest du später
- **PC 2 = Server-PC** (Ubuntu Linux empfohlen) → hier läuft DNS + Firewall

### Was du brauchst
- 2 PCs/Laptops im selben Netzwerk (z. B. über Router/WLAN/LAN)
- Auf **PC 2**: Ubuntu (oder ein anderes Debian/Ubuntu-Linux)
- Admin-Rechte (sudo)
- Internet (für Installation)

---

## Schritt 1: IP-Adresse vom Server-PC festhalten
Auf **PC 2** (Server-PC) Terminal öffnen:

```bash
ip a
Merke dir die IPv4-Adresse (z. B. 192.168.178.50).
Am besten bekommt PC 2 im Router eine feste IP (oder du stellst sie später statisch ein).


Schritt 2: System updaten
bash
Code kopieren
sudo apt update && sudo apt upgrade -y


Schritt 3: DNS-Server installieren (Bind9)
bash
Code kopieren
sudo apt install bind9 dnsutils -y
Bind9 starten + Autostart aktivieren:

bash
Code kopieren
sudo systemctl enable --now bind9
Status checken:

bash
Code kopieren
systemctl status bind9 --no-pager
Schritt 4: DNS so einstellen, dass er im Heimnetz lauscht
Wir sorgen dafür, dass Bind im lokalen Netzwerk Anfragen annimmt.


Datei öffnen:

bash
Code kopieren
sudo nano /etc/bind/named.conf.options
Ersetze den Inhalt zwischen den { } von options ungefähr so (du kannst das 1:1 kopieren):


c
Code kopieren
options {
    directory "/var/cache/bind";

    recursion yes;
    allow-recursion { localhost; 192.168.0.0/16; };

    listen-on { 127.0.0.1; 192.168.0.0/16; };
    listen-on-v6 { none; };

    dnssec-validation auto;

    forwarders {
        1.1.1.1;
        8.8.8.8;
    };
};
Speichern & schließen:

CTRL + O → Enter

CTRL + X

Konfiguration prüfen:
bash
Code kopieren
sudo named-checkconf
Bind neu starten:

bash
Code kopieren
sudo systemctl restart bind9
Schritt 5: Firewall einrichten (UFW)
UFW installieren (falls nicht vorhanden):

bash
Code kopieren
sudo apt install ufw -y
Optional: SSH erlauben (nur falls du dich später remote verbinden willst):

bash
Code kopieren
sudo ufw allow OpenSSH
DNS erlauben (Port 53 UDP/TCP):

bash
Code kopieren
sudo ufw allow 53/udp
sudo ufw allow 53/tcp
Firewall aktivieren:

bash
Code kopieren
sudo ufw enable
Status prüfen:

bash
Code kopieren
sudo ufw status verbose
Schritt 6: DNS testen (auf dem Server-PC)
Erst lokal testen, ob Bind antwortet:

bash
Code kopieren
dig google.com @127.0.0.1
Wenn du eine Antwort bekommst (SECTION „ANSWER“), läuft es.

Jetzt mit der Server-IP testen (ersetze SERVER_IP):

bash
Code kopieren
dig google.com @SERVER_IP



Schritt 7: PC 1 so einstellen, dass er den Server als DNS nutzt
Windows (PC 1)
Einstellungen → Netzwerk & Internet → Adapteroptionen


Rechtsklick auf WLAN/LAN → Eigenschaften

„Internetprotokoll Version 4 (TCP/IPv4)“ → Eigenschaften

„Folgende DNS-Serveradressen verwenden“

Bevorzugter DNS-Server = IP von PC 2 (z. B. 192.168.178.50)

Dann testen (CMD):

bat
Code kopieren
nslookup google.com
Linux (PC 1)
Du kannst im Network-Manager den DNS auf die IP von PC 2 setzen, dann testen:

bash
Code kopieren
dig google.com
Troubleshooting (wenn’s nicht klappt)





1) Bind läuft?
bash
Code kopieren
systemctl status bind9 --no-pager


2) Hört Bind auf Port 53?
bash
Code kopieren
sudo ss -lntup | grep :53


3) Firewall blockt was?
bash
Code kopieren
sudo ufw status verbose


4) Logs ansehen
bash
Code kopieren
journalctl -u bind9 --no-pager -n 50
