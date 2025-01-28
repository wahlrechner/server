# Aufsetzen eines Wahlrechner-Servers

Dieses Repository dient als Hifestellung, um eine [Wahlrechner](https://github.com/wahlrechner/wahlrechner)-Instanz auf einem Server aufzusetzen.
Die folgende Anleitung funktioniert nur auf Debian-basierten Systemen, und wurde ausschließlich mit `Ubuntu 24.04 LTS`
getestet.

## Vorraussetzungen

### Installation von Git

```
sudo apt update && sudo apt install git -y
```

### Repository klonen

```
mkdir /opt/wahlrechner-server/
git clone --recurse-submodules https://github.com/wahlrechner/server /opt/wahlrechner-server/
cd /opt/wahlrechner-server
```

### Installation von Docker

_Mehr Informationen zur Installation von Docker findest du [hier](https://docs.docker.com/engine/install/ubuntu/)._

```
sudo apt install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
```

```
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

```
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```


## Wahlrechner Theme als Submodul einbinden

Beispiel für [theme_buxtomat](https://github.com/wahlrechner/theme_buxtomat):

```
git submodule add https://github.com/wahlrechner/theme_buxtomat.git /opt/wahlrechner-server/themes/theme_buxtomat
```


## Konfiguration des Wahlrechners

**Bevor du den Wahlrechner-Server das erste Mal startest,** musst du die globale Konfigurationsdatei `global.env`
erstellen. Eine Vorlage ist unter `config/global.env.example` zu finden.

```
cd /opt/wahlrechner
cp config/config.env.example config/config.env
```

**DJANGO_SECRET_KEY:** Ersetze `ChangeThisToRandomStringInProduction` durch eine [zufällig generierte](https://1password.com/de/password-generator/) (mind. 30 Zeichen lang, bestehend aus Zahlen, Buchstaben und Sonderzeichen) Zeichenkette. Teile den Secret Key niemals mit jemand anderem!

**DJANGO_DEFAULT_ADMIN_PASSWORD:** Beim erstmaligen Starten des Wahlrechners wird automatisch ein Admin-Account erstellt. Ersetze `adminpassword` durch ein erstes, sicheres Passwort. **Nach der erstmaligen Anmeldung in der Admin-Oberfläche solltest du dein Passwort zusätzlich nochmal ändern.**

**MYSQL_PASSWORD:** Ersetze `SetDatabaseUserPassword` durch ein zufällig generiertes Passwort. Du wirst es niemals von Hand eingeben müssen - also lass dir bitte ein sicheres Passwort mit einem [Passwortgenerator](https://1password.com/de/password-generator/) generieren.

Zusätzlich musst du für deine Wahlrechner-Instanzen ebenfalls eine eigene Konfigurationsdatei erstellen. Eine Vorlage
ist unter `config/wahlrechner-eins.env.example` zu finden.

```
cd /opt/wahlrechner
cp config/wahlrechner-eins.env.example config/wahlrechner-eins.env
```

Falls du mehr als eine Wahlrechner-Instanz aufsetzen möchtest, erstelle eine Konfigurationsdatei für jede Instanz. Achte
darauf, dass jede Instanz einen einzigartigen Datenbank-Namen erhält.

## Konfiguration des Webservers

Für die Konfiguration des Webservers erstelle eine Datei `web/nginx.conf`. Du findest dafür eine Vorlage unter
`web/nginx.conf.example` bzw. `web/nginx-ssl.conf.example` für eine Verwendung mit SSL.

Bitte setze den `server_name` auf deine Domain, damit das Routing richtig funktioniert. Bei einer Verwendung von
mehreren Instanzen sind weitere Änderungen notwendig.

## Anpassung der Konfiguration für mehrere Instanzen

Wenn du mehrere Wahlrechner-Instanzen aufsetzen möchtest, musst du einiges anpassen. Die wichtigsten Stellen in den
Konfigurationsdateien sind mit einem `TODO` markiert.

Zu den Dateien, die angepasst werden müssen, gehören unter anderem:

- `web/nginx.conf`
- `docker-compose.yml`
- Anlegen einer neuen Konfigurationsdatei unter `config/`

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

Lasse dir anschließend von certbot ein Zertifikat pro Wahlrechner-Instanz ausstellen (eventuell muss der Pfad zu den
Skripten angepasst werden):

```

sudo certbot certonly --standalone --pre-hook "bash /opt/wahlrechner-server/ServerStop.sh" --post-hook "bash
/opt/wahlrechner-server/ServerStart.sh"
```

Erstelle anschließend einen Symlink, damit die Zertifikate automatisch aktualisiert werden können.
```

mkdir ./web/cert/
ln -s /etc/letsencrypt/live/* ./web/cert/
```

### Eigenes Zertifikat

Du kannst auch ein eigenes Zertifikat verwenden. Dafür kopierst du den privaten Schlüssel in `/web/cert/privkey.pem` und den öffentlichen Schlüssel in `/web/cert/fullchain.pem`. Andere Dateinamen und Dateiendungen sind aktuell nicht möglich.

### Anpassen der `nginx.conf`

Bitte passe anschließend in der `web/nginx.conf` den Pfad für das `ssl_certificate` und `ssl_certificate_key` zu deinem generierten Zertifikat an.

## Erstmaliges Starten eines Wahlrechners

Für den **ersten** Start führe bitte das Skript `ServerUpdate.sh` aus. Dies lädt automatisch die neuste Version herunter und führt anschließend den Server aus:

```
bash ServerUpdate.sh
```

Nach dem ersten Starten melde dich bitte bei jeder Wahlrechner-Instanz im Admin-Panel (`https://example.com/admin`) mit
dem Benutzername `admin` und dem von dir festgelegten Passwort an. Klicke anschließend oben rechts auf `Passwort ändern`
und ändere dein Passwort.

Mehr Informationen zur Bedienung der Admin-Oberfläche des Wahlrechners findest du im [Wiki](https://github.com/wahlrechner/wahlrechner/wiki).

## Starten, Stoppen und Aktualisieren aller Wahlrechner

Du kannst den Server mit dem Skript `ServerStart.sh` **starten**:

```
bash ServerStart.sh
```

Du kannst den Server mit dem Skript `ServerStop.sh` wieder **stoppen**:

```
bash ServerStop.sh
```

Um alle Wahlrechner-Instanzen auf die neuste Version zu aktualisieren, führe das Skript `ServerUpdate.sh` aus.
Anschließend wird der Server automatisch gestartet:

```
bash ServerUpdate.sh
```

### Automatisches Neustarten der Wahlrechners

Wenn du möchtest, kannst du folgenden cron job einrichten (`crontab -e`), um alle Docker Container jede Nacht neuzustarten:

```

0 3 * * * bash /opt/wahlrechner-server/ServerStop.sh && bash /opt/wahlrechner-server/ServerStart.sh >/dev/null 2>&1
```

_Der Pfad muss entsprechend angepasst werden._
