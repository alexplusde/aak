# aak - yak mit Anpassungen an alex+ Digitales Marketing

Für die, die ganz frisch auf einem lokalen Rechner starten, ist hier eine [Anleitung](https://getgrav.org/blog/macos-sonoma-apache-multiple-php-versions) um ein Apache Setup mit mehreren PHP-Versionen aufzusetzen, eine Umgebung mit MySQL, virtuellen Hosts, APC-Caching, YAML und Xdebug einzurichten sowie SSL für virtuelle Apache-Hosts. 
Diese Anleitung richtet sich an erfahrene Webentwickler. Wenn man Anfänger ist, ist man mit MAMP oder MAMP Pro besser bedient.

Ansonsten wird 
- eine `bash` benötigt und
- [`yarn`](https://yarnpkg.com) sollte installiert sein

### Installation

1. Laragon / WAMP / MAMP installieren und konfigurieren.
2. Dieses Repository-Template in ein neues Repository kopieren.
3. REDAXO wird mit Ausführung des nächsten Befehls automatisch installiert und eine vorhandene Instanz wird überschrieben.

    ```
    $ php setup/presetup.php
    ```

    **Nach dem presetup sollte die neue Redaxo Struktur wie folgt aussehen**

    ```
    - /assets/
        - fonts/
        - images/
        - scripts/
        - styles/
        - svgs/
    - /bin/
    - /public/
        - assets/
            - addons/
            - core/
        - media/
        - redaxo/
    - /src/
        - addons/
        - core/
        - module/
        - templates/
        - AppPathProvider.php
    - /var/
        - cache/
        - data/
        - log/
    - .env
    - .gitignore
    - deploy.php
    - LICENSE
    - package.json
    - postcss.config.js
    - README.md
    - webpack.config.js
    ```
   
1. Datei `.env` kopieren und als `.env.local` auf selbige Ebene speichern. Die Datei `.env.local` öffnen und wie folgt anpassen
    ```
    APP_HOST=project-name.test
    APP_ENV=dev
    ```
1. Im Anschluss `yarn` ausführen

1. Browser öffnen und Redaxo Setup via https://project-name.test/redaxo starten

## Nach dem Setup

1. `developer` AddOn installieren und [Einstellungen](#einstellungen-developer-addon) für die lokale Instanz vornehmen. Die Einstellungen für die Live-Instanz können erst nach dem ersten deployen vorgenommen werden.

2. `ydeploy` AddOn installieren

### Einstellungen Developer AddOn

#### Lokal

- [x] Templates synchronisieren
- [x] Module synchronisieren
- [x] Actions synchronisieren
- [ ] YForm: E-Mail-Templates synchronisieren
- [x] Im Frontend synchronisieren (nur wenn als Admin im Backend eingeloggt)
- [x] Im Backend synchronisieren (nur wenn als Admin eingeloggt)
- [x] Datei- und Ordnernamen aktuell halten
- [x] Ordnernamen mit ID als Suffix
- [ ] Präfix für Dateinamen (enthält ID und Name)
- [ ] Umlaute in Namen beibehalten (Deprecated; die Option wird in der nächsten Major-Version wegfallen und somit immer deaktiviert sein)
- [x] Item-Ordner löschen nach dem Löschen eines Items über das Backend


#### Staging/Prod

- [x] Templates synchronisieren
- [x] Module synchronisieren
- [x] Actions synchronisieren
- [ ] YForm: E-Mail-Templates synchronisieren
- [ ] Im Frontend synchronisieren (nur wenn als Admin im Backend eingeloggt)
- [ ] Im Backend synchronisieren (nur wenn als Admin eingeloggt)
- [ ] Datei- und Ordnernamen aktuell halten
- [ ] Ordnernamen mit ID als Suffix
- [ ] Präfix für Dateinamen (enthält ID und Name)
- [ ] Umlaute in Namen beibehalten (Deprecated; die Option wird in der nächsten Major-Version wegfallen und somit immer deaktiviert sein)
- [ ] Item-Ordner löschen nach dem Löschen eines Items über das Backend

### zusätzliche Datenbank-Tabellen synchronisieren

Da man lokal am Anfang zumeist die Struktur wie vom Kunden gewünscht aufsetzt und ggf. auch die ersten Slices als Beispiele in der Live-Instanz bereitstellen möchte, kann man mit folgendem Skript diese Tabellen synchronisieren.

**!** Sobald jedoch an der Live-Instanz redaktionell gearbeitet wird, sollte das Skript wieder entfernt werden. Ansonsten gehen die Daten der Live-Instanz verloren.

Das Skript in die `boot.php`des `project` AddOns der **lokalen Instanz** einfügen

```php
// YDeploy
// - - - - - - - - - - - - - - - - - - - - - - - - - -
if (\rex::isBackend() && \rex_addon::get('ydeploy')->isAvailable()) {

    rex_extension::register('PACKAGES_INCLUDED', function () {
        $config = \rex_addon::get('ydeploy')->getProperty('config');

        // zusätzliche Tabellen synchronisieren
        // nie action, module, module_action, template definieren
        // werden über developer AddOn synchronisiert
        $config['fixtures']['tables'] = array_merge(
            [
                'article' => null,
                'article_slice' => null,
                'clang' => null,
                'media' => null,
                'media_category' => null,
                'sprog_wildcard' => null,
            ],
            $config['fixtures']['tables']
        );

        \rex_addon::get('ydeploy')->setProperty('config', $config);
    });
}
```

## Erstes deployen auf Staging/Prod (bei alex+ Digitales Marketing: next/live)

**alle Befehle gehen direkt vom Projektordner aus.**  
`~/Sites/localhost.project-name`

- Datenbankdump der lokalen Instanz erstellen und auf Live via Adminer oder Datenbanktool einspielen
- `deploy.php` in der lokalen Instanz für Server anpassen
- Ausführen `$ bin/console ydeploy:diff`
- lokalen Stand auf GitHub pushen
- Ausführen `$ dep deploy` (hier kommt es absichtlich noch zu einem Fehler, aber die Grundstruktur ist schon mal auf dem Server)

**auf dem Live Server via FTP Client**

- `/releases/1/src/core/default.config.yml` nach `/shared/var/data/core/config.yml` kopieren
- `data/core/config.yml` öffnen und
    - `setup` auf `false`
    - Datenbankverbindung der Live-Instanz eingetragen

- Ausführen `$ dep deploy:unlock`
- Ausführen `$ dep deploy` (diesmal sollte kein Fehler mehr kommen)
- Domain der Live-Instanz auf `current/public` zeigen lassen (Der Pfad muss zumeist per Hand notiert werden, da es ein Symlink ist)

## Allgemeines Arbeiten

1. Vor dem Deployen, zuvor alle geänderte Dateien committen und in der Konsole `$ bin/console ydeploy:diff` aufrufen. Die geänderten bzw. neu angelegte Dateien (fixtures, migration, schema) ebenfalls committen.
1. In der Konsole dann `$ dep deploy` ausführen oder die Pipeline anstoßen.

## Bekannte Probleme

Siehe Original-Repository <https://github.com/yakamara/yak>

## Lizenz

Dieses Projekt ist unter der MIT-Lizenz lizenziert. Weitere Informationen finden man in der Datei [LICENSE](LICENSE).

