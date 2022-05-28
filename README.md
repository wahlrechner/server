# Aufsetzen eines Wahlrechner-Servers

Dieses Repository dient als Hifestellung, um eine [Wahlrechner](https://github.com/wahlrechner/wahlrechner)-Instanz auf einem Server aufzusetzen.
Die folgende Anleitung funktioniert nur auf Debian-basierten Systemen, und wurde ausschließlich mit `Ubuntu 20.04` getestet.

## Vorraussetzungen

### Installation von Git

```
sudo apt-get update
sudo apt-get install git
```

### Repository klonen

```
mkdir wahlrechner-server/
git clone --recurse-submodules https://github.com/wahlrechner/server wahlrechner-server/
cd wahlrechner-server
```

### Installation von Docker

_Mehr Informationen zur Installation von Docker findest du [hier](https://docs.docker.com/engine/install/ubuntu/)._

```
sudo apt install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
```

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

```
sudo add-apt-repository \
"deb [arch=amd64] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) \
stable"
```

```
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

### Docker Compose

_Mehr Informationen zur Installation von Docker Compose findest du [hier](https://docs.docker.com/compose/install/)._

```
sudo curl -L "https://github.com/docker/compose/releases/download/1.28.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

```
sudo chmod +x /usr/local/bin/docker-compose
```

## Wahlrechner Theme als Submodul einbinden

Beispiel für [theme_buxtomat](https://github.com/wahlrechner/theme_buxtomat):

```
git submodule add https://github.com/wahlrechner/theme_buxtomat.git themes/theme_buxtomat
```


## Konfiguration des Wahlrechners

**Bevor du den Wahlrechner-Server das erste Mal startest,** musst du die Konfigurationsdatei `config.env` anpassen. Diese ist im Ordner `config/` zu finden.

**DJANGO_SECRET_KEY:** Ersetze `ChangeThisToRandomStringInProduction` durch eine [zufällig generierte](https://1password.com/de/password-generator/) (mind. 30 Zeichen lang, bestehend aus Zahlen, Buchstaben und Sonderzeichen) Zeichenkette. Teile den Secret Key niemals mit jemand anderem!

**DJANGO_DEFAULT_ADMIN_PASSWORD:** Beim erstmaligen Starten des Wahlrechners wird automatisch ein Admin-Account erstellt. Ersetze `adminpassword` durch ein erstes, sicheres Passwort. **Nach der erstmaligen Anmeldung in der Admin-Oberfläche solltest du dein Passwort zusätzlich nochmal ändern.**

**MYSQL_PASSWORD:** Ersetze `SetDatabaseUserPassword` durch ein zufällig generiertes Passwort. Du wirst es niemals von Hand eingeben müssen - also lass dir bitte ein sicheres Passwort mit einem [Passwortgenerator](https://1password.com/de/password-generator/) generieren.


## Installation eines SSL-Zertifikats

### Let's Encrypt

_Mehr Informationen zum Ausstellen eines SSL-Zertifikats mit certbot findest du [hier](https://certbot.eff.org/lets-encrypt/ubuntufocal-other)._

Installiere zuerst certbot:

```
sudo apt install snapd
```

```
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

Lasse dir anschließend von certbot ein Zertifikat ausstellen (Eventuell muss der Pfad zu den Skripten angepasst werden):

```
sudo certbot certonly --standalone --pre-hook "bash /root/wahlrechner-server/ServerStop.sh" --post-hook "bash /root/wahlrechner-server/ServerStart.sh"
```

Erstelle anschließend einen Symlink, damit die Zertifikate automatisch aktualisiert werden können. **Ersetze `example.com` durch deine Domain:**

```
mkdir web/cert/
ln -s /etc/letsencrypt/live/example.com/* web/cert/
```

### Eigenes Zertifikat

Du kannst auch ein eigenes Zertifikat verwenden. Dafür kopierst du den privaten Schlüssel in `/web/cert/privkey.pem` und den öffentlichen Schlüssel in `/web/cert/fullchain.pem`. Andere Dateinamen und Dateiendungen sind aktuell nicht möglich.

## Erstmaliges Starten des Wahlrechners

Für den **ersten** Start führe bitte das Skript `ServerUpdate.sh` aus. Dies lädt automatisch die neuste Version herunter und führt anschließend den Server aus:

```
bash ServerUpdate.sh
```

Nach dem ersten Starten melde dich bitte im Admin-Panel (`https://example.com/admin`) mit dem Benutzername `admin` und dem von dir festgelegten Passwort an. Klicke anschließend oben rechts auf `Passwort ändern` und ändere dein Passwort.

Mehr Informationen zur Bedienung der Admin-Oberfläche des Wahlrechners findest du im [Wiki](https://github.com/wahlrechner/wahlrechner/wiki).

## Starten, Stoppen und Aktualisieren des Wahlrechners

Du kannst den Server mit dem Skript `ServerStart.sh` **starten**:

```
bash ServerStart.sh
```

Du kannst den Server mit dem Skript `ServerStop.sh` wieder **stoppen**:

```
bash ServerStop.sh
```

Um die Wahlrechner-Instanz auf die neuste Version zu aktualisieren, führe das Skript `ServerUpdate.sh` aus. Anschließend wird der Server automatisch gestartet:

```
bash ServerUpdate.sh
```

### Automatisches Neustarten des Wahlrechners

Wenn du möchtest, kannst du folgenden cron job einrichten (`crontab -e`), um alle Docker Container jede Nacht neuzustarten:

```
0 3 * * * bash /root/wahlrechner-server/ServerStop.sh && bash /root/wahlrechner-server/ServerStart.sh >/dev/null 2>&1
```

_Der Pfad muss entsprechend angepasst werden._
