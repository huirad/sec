# Mobile Datenträger verschlüsseln

Im folgenden werden einige Methoden vorgestellt, mit denen mobile Datenträger
wie *USB Sticks*, ggf. auch externe Festplatten oder CD-ROMs verschlüsselt werden können.

Manche der Methoden sich auch für eingebaute Festplatten etc geeeignet.


Also: worum geht es im Detail?

* Schutzziele: Vertraulichkeit, Integrität
* Verschlüsselung für Transport oder Speicherung
* Dateisystemverschlüsselung: direkter Zugriff ohne Umkopieren


Weitere Kriterien

* Betriebssystem(un)abhängigkeit
* Proprietär vs offen
* Hardwarebasierte Verschlüsselung?
  * Mögliche Vorteile. Speed, Schutz (Anzahl der fehlerhaften Passwort Eingaben ist begrenzt)
  * Mögliche Nachteile: Kosten, Portabilität



## [Kingston](https://www.kingston.com/de/usb/encrypted_security/dtvp30)
Kingston bietet verschiedene hardware-verschlüsselte USB Sticks an.

Übersicht: siehe https://www.kingston.com/de/usb/encrypted_security.

Vorteile:
* funktioniert für Windows, Linux, MAC
* keine SW-Installation nötig (SW für alle Betriebssystem ist auf RO-Partition)
* FIPS zertifiziert (was auch immer das heisst)
* reher robusteres Gehäuse

Nachteile
* Man sieht von aussen, dass es ein verschlüsselter Speicherstick ist
* proprietär (keine Ahnung ob es da Backdoors gibt?)
* teuer (80 EUR für 16GB)
* Kritik an AES-XTS: https://sockpuppet.org/blog/2014/04/30/you-dont-want-xts/

## [Nitrokey Storage](https://shop.nitrokey.com/de_DE/shop/product/nitrokey-storage-2-16gb-23)
Der Nitrokey Storage ist ein hardware-verschlüsselter USB Stick basierend auf Open-Source Hardware


Eine Übersicht über die Funktionen liefert das [Datenblatt](https://www.nitrokey.com/files/doc/Nitrokey_Storage_Infoblatt.pdf)

Vorteile:
* funktioniert für Windows, Linux, MAC
* Ab Version 2 keine SW-Installation nötig (SW für alle Betriebssystem ist auf RO-Partition)
* Open Source Hardware und Software
* Firmware Updates möglich
* Unterstützt sogar versteckte Volumes
* Zusätzliche Funktionen 
  * zusätzliche unverschlüsselte Partition (default: Read-Only)
  * SmartCard (Schlüsselspeicher und Signaturen für PGP)
  * Passwortmanager (16 Einträge)
  * Einmalpasswörter (HOTP, TOTP)
  * Physikalischer Zufallszahlengenerator (TRNG)

Nachteile
* Man sieht von aussen, dass es ein verschlüsselter Speicherstick ist
* teuer (noch etwas teurer als der Kingston Stick: 110 EUR für 16GB)
* Bei der älteren Version 1 muss die SW noch manuell heruntergeladen werden (Windows) bzw über den Paketmanager (Windows) installiert werden
* Aufgrund der vielen Funktionen erscheint die Bedienung eher kompliziert
* Auch die Dokumentation ist eher schwer zu lesen

Die Firma Nitrokey bietet noch andere USB Sticks mit Krypto-Funktionen an. Siehe https://www.nitrokey.com/de


## Bitlocker
Software-Verschlüsselung von Microsoft, verfügbar in den Pro/Enterprise Versionen von MS-Windows.

Einfachste Wahl für Anwender in Firmen.

Vorteile
* ist einfach dabei, wenn man die passende Windows version hat

Nachteile
* funktioniert nur mit den teureren Windows Versionen


## [SecurStick](http://www.withopf.com/tools/securstick/)
Eine vordergündig recht elegante Software-Lösung, basierend auf einem Projekt der Zeitschrift c't.
Die verschlüsselten Daten werden per WebDAV zugänglich gemacht.


Links:
  * Homepage: http://www.withopf.com/tools/securstick/
  * C't Artikel: https://www.heise.de/ct/artikel/Sperrgebiet-1901289.html
  * Noch eine Anleitung: https://www.lgs-dieburg.de/images/media/Anleitung_SecurStick_PDF_07.pdf

Vorteile:
* funktioniert für Windows, Linux, MAC

Nachteile
* Nicht wirklich Open Source --> Ausschlusskriterium!

Tipp für Linux
* Programm per Kommandozeile öffnen
* Dann den angegebenen Link im Browser öffnen: http://127.0.0.2:2000/login
* Verschlüsseltes Laufwerk per WebDAV in Dolphin Dateimanager einbinden: webdav://127.0.0.2:2000/X

## dm-crypt/LUKS

dm-crypt mit LUKS ist weit verbreitet unter Linux.
Es arbeitet unterhalb des Dateisystemebene und eignet sich sogar zur Verschlüsselung von Boot Partitionen.
Eine Übersicht gibt der [Wikipedia](https://en.wikipedia.org/wiki/Dm-crypt) Artikel.

Mit dm-crypt kann man verschlüsselte Dateisysteme erstellen für
* ganze Laufwerke
* Partitionen
* Containerdateien (macht weniger Sinn bei mobilen Datenträgern weil es beim Anstecken nicht automatisch erkannt wird)

Vorteil
* wird von jeder standard Linux Distribution unterstützt
* (fast) beliebige Dateisysteme möglich: ext4, ntfs, FAT, ....

Nachteil
* geht standardmässig nur mit Linux
  * die Windows Variante [LibreCrypt](https://github.com/t-d-k/LibreCrypt) wird offenbar nicht aktive gepflegt
* die Größe des verschlüsselten Bereichs muss vorab festgelegt werden

### How-to
Verschlüsselte Partition anlegen in 3 Schritten - werden unten im Detail beschrieben
1. Platz machen: Partition oder Containerdatei erstellen
2. Verschlüsselung aktivieren
  * lukSformat
3. Dateisystem anlegen   
  * luksOpen,  mkfs, optional chown luksClose
4. täglicher Gebrauch
   * wird meist ganz einfach von Desktop und dem zugehörigen Dateimanager unterstützt


#### Einrichten, Schritt 1: Platz machen

Referenzen: 
[kuketz](https://www.kuketz-blog.de/dm-crypt-luks-daten-unter-linux-sicher-verschluesseln/), 
[miguelmenendez](https://miguelmenendez.pro/en/blog/2014/10/encrypt-usb-storage-device-linux-unified-key-setup-luks/)

Entweder USB-Stick umpartitionieren, am einfachsten mit dem vorhandenen graphischenen Partition-Editor Deines Linux, z.B. partitioner (openSUSE KDE) der gparted (*buntu gnome).
   Dann eine Partition raussuchen, die verschlüsselt werden soll, z.B. /dev/sdc2.

Oder: Eine bestehende Partition raussuchen, die verschlüsselt werden soll, z.B. /dev/sdc1.

Oder: Eine Containerdatei anlegen (wird im folgenden nicht weiter beschrieben)

#### Einrichten, Schritt 2: Verschlüsselung für eine Partition aktivieren

Vorab: ein sicheres Passwort überlegen!

Sicherstellen, dass die Partion nicht gemountet ist. Check mit 
    
    lsblk #CHECK: zeigt alle Blockdevices an und wo sie ggf gemountet sind

Verschlüsselung aktivieren - löscht den Inhalt der Partition

    cryptsetup luksFormat /dev/sdb2 #hier musst Du das Passwort vergeben


#### Einrichten, Schritt 3: Dateisystem anlegen

Vorab: überlegen welches Dateisystem verwendet werden soll!
Am portabelsten für mobile Datenträger ist vermutlich FAT.
Filesysteme mit UNIX Berechtigungen wie ext4 machen eigentlich nur dann Sinn,
wenn sie immer am gleichen Rechner verwendet werden
Grund: die user-ids werden im Filesystem numerisch abgespeichert.
Die Zuordung zu den usern erfolgt lokal (/etc/passwd) und ist in der Regel nicht syncchronisiert zwschen verschiedenen Rechnern.


Erst öffnen wir den verschlüsselten Bereich (hier im beispiel /dev/sdb2) und geben an, wo er gemappt werden soll

    cryptsetup luksOpen /dev/sdb2 usb_crypt # mappt die verschlüsselte Partition auf /dev/mapper/usb_crypt
    lsblk # CHECK: zeigt alle Blockdevices an und wo sie ggf gemountet sind
    ls -l /dev/mapper/usb_crypt # CHECK: wirklich unter /dev/mapper verfügbar

Dann legen wir eine Dateisystempartition an: entweder im FAT Format

    mkfs.vfat /dev/mapper/usb_crypt -n CRYPT_VFAT # -n setzt den Namen mit dem das Dateisystem später gemountet wird

Oder im NTFS Format

    mkfs.ntfs /dev/mapper/usb_crypt -L CRYPT_NTFS # -L setzt den Namen mit dem das Dateisystem später gemountet wird

Oder ext4

    mkfs.etf4 /dev/mapper/usb_crypt -L CRYPT_EXT4 # -L setzt den Namen mit dem das Dateisystem später gemountet wird
    chown -R <USER>:<GROUP> /dev/mapper/usb_crypt # Zugriffsberechtigung setzen!

Dann schliessen wir das Device wieder

    cryptsetup luksClose usb_crypt


#### Täglicher Gebrauch

Normalerweise ganz einfach unterstützt vom Desktop und dem zugehörigen Dateimanager:

* Nach Anstecken eines verschlüsselten USB Sticks wird nach den Passwort gefragt.
* Nach korrekter Eingabe des Passworts ist die verschlüsselte Partition zugänglich.
* Wie bei unverschlüsselten USB-Sticks sollte man sie vor dem Abstecken freigeben.


Notfalls kann man den verschlüsselten Datenträger auch per Shell mit den Befehlen luksOpen und luksClose nutzen.



### Referenzen
Allgemein

* cryptsetup: [man-page](https://linux.die.net/man/8/cryptsetup), [gitlab](https://gitlab.com/cryptsetup/cryptsetup)
* Wikipedia dm-crypt: [EN](https://en.wikipedia.org/wiki/Dm-crypt), [DE](https://de.wikipedia.org/wiki/Dm-crypt)
* Hintergrund [linux-magazin](http://www.linux-magazin.de/ausgaben/2005/08/geheime-niederschrift/)
* Weitere Artikel aus dem Linux Magazin [1](http://www.linux-magazin.de/ausgaben/2006/10/beklaut-und-ausspioniert/)
  [2](http://www.linux-magazin.de/ausgaben/2006/10/loechriger-kaese/)
  [3](http://www.linux-magazin.de/ausgaben/2011/06/bitparade/)
* [New Methods in Hard Disk Encryption](http://clemens.endorphin.org/nmihde/nmihde-A4-os.pdf)

Spezifische Anleitungen
* Anleitung für Ubuntu: [LUKS](https://wiki.ubuntuusers.de/LUKS/):
  [Partitionen versclüsseln](https://wiki.ubuntuusers.de/LUKS/Partitionen_verschl%C3%BCsseln/)
  [Containerdatei](https://wiki.ubuntuusers.de/LUKS/Containerdatei/)
  [noch eine Anleitung] https://www.pcwelt.de/ratgeber/Luks-Verschluesselung-auf-USB-Sticks-uebertragen-Mobiles-Luks-9601788.html
* Bestehende Partition verschlüsseln [kuketz](https://www.kuketz-blog.de/dm-crypt-luks-daten-unter-linux-sicher-verschluesseln/)
* Neue Partition anlegen zum verschlüsseln [miguelmenendez](https://miguelmenendez.pro/en/blog/2014/10/encrypt-usb-storage-device-linux-unified-key-setup-luks/), [suse](https://doc.opensuse.org/documentation/leap/security/html/book.security/cha.security.cryptofs.html), [linuxconfig](https://linuxconfig.org/usb-stick-encryption-using-linux), [grund-wissen](https://www.grund-wissen.de/linux/datensicherung/verschluesselung.html)
* Containerdatei: [opensuse](https://doc.opensuse.org/documentation/leap/security/html/book.security/cha.security.cryptofs.html#sec.security.cryptofs.y2.vdisk), [elephply](https://elephly.net/posts/2013-10-01-dm-crypt.html)

## eCryptFS oder verschlüsseltes ext4

eCryptFS ist ein verschlüsseltes Dateisystem für Linux

Referenzen
* http://ecryptfs.org/
* https://en.wikipedia.org/wiki/ECryptfs
* https://www.admin-magazin.de/Das-Heft/2011/02/Verschluesselte-Festplatten-per-eCryptfs
* https://lwn.net/Articles/156921/; https://lwn.net/Articles/305931/
* https://wiki.archlinux.org/index.php/ECryptfs

Seit ca 2015 (Kernel 4.1) bietet auch das ext4 Dateisystem eine Verschlüsselung an: https://lwn.net/Articles/639427/

## Veracrypt

Referenz: https://www.kuketz-blog.de/veracrypt-daten-auf-usb-stick-sicher-verschluesseln/

Irgendwie ist mir das eher dubios. Hab's noch nie gemocht. 
Vor allem beim Vorgängerprojekt TrueCrypt gab's das ganze Lizenzschlamassel und die Unklarheit wer eigentlich der Author ist.

## Alternativen

### Einfache Dateiverschlüsselung
Einfache Dateiverschlüsselung gibt es natürlich auch noch, z.B.:
* gpg https://www.gnupg.org
  * [Anleitung](https://www.gnupg.org/howtos/de/GPGMiniHowto-4.html)
  * Sicherste und flexibelste Methode
  * Ist bei Linux immer dabei
  * Installer für Windows: https://www.gpg4win.de/
* [openssl enc](https://www.openssl.org/docs/man1.0.2/man1/openssl-enc.html) 
  * Anleitung: [Wiki](https://wiki.openssl.org/index.php/Enc)
  * Ist bei Linux immer dabei
* zip (WinZip / 7-zip)
  * geht für Windows irgendwie immer, dort die einfachste Methode

Vorteil
* alles schön einfach

Nachteil
* die entschlüsselten Daten müssen ausgepackt und zwischengespeichert werden

### Dein Smartphone
Dein Smartphone ist zwar kein USB-Stick aber natürlich auch ein mobiler Datenspeicher.
Und bei aktuellen Smartphones kannst Du einstellen, dass der Flash Speicher und ggf die Micro-SD-Karte verschlüsselt wird.

Vorteile
* Eh da, kostet also aucht nicht extra
* Irgendwie immer dabei, fällt daher nicht auf

Nachteile
* Wer weiss schon, wie die Verschlüsselung wirklich funktioniert?
* Kennt wirklich niemand Deine PIN oder kann das Smartphone anderweitig entsperren?
* Alle möglichen Apps haben Zugriff auf die Daten
* Irgendwie immer dabei, kannst Du also nicht separat wegsperren, liegt auch mal irgendwo rum

Um wirklich sicher zu gehen, könnte man die wichtigen Dateien auf dem Smartphone noch mal extra verschlüsseln.


### Die Cloud
Oder Du speicherst Deine Sachen auf Dropbox etc. Einfach so nochmal per zip/gpg/openssl-enc verschlüsselt.

Vorteil
* immer verfügbar, wenn Du Internetzugang hast und der Cloud Provider online ist
* was Du nicht physisch dabei hast, kann Dir keiner wegnehmen
* was Du nicht physisch dabei hast, sieht auch keiner dass Du es hast

Nachteil
* nicht verfügbar wenn Du keinen Internetzugang hast oder der Cloud Provider nicht zugänglich ist
* wer weiss, wo Deine Daten wirklich gespeichert sind und wer da mitliest (und wenn es nur Metadaten sind)
* wer weiss, wie gut die Sicherheitsvorkehrungen des Cloud Providers sind
* wer weiss, warum das so oft kostenlos ist






