# Progetto D&D - Drupal

Guida rapida per avviare il sito Drupal in locale e in produzione.

## 📋 Prerequisiti

- PHP 8.1 o superiore
- MySQL 5.7+ o MariaDB 10.3+
- Composer
- Server web (per produzione: Apache/Nginx, per sviluppo: Drush)

## 🔧 Configurazione Database

Il database configurato è:
- **Nome database**: `dnd-drupal`
- **Host**: `localhost`
- **Porta**: `3306`
- **Username**: `admin`
- **Password**: `admin`

## 🚀 Avvio in Locale (Sviluppo)

### Metodo 1: Drush Server (Raccomandato per sviluppo)

```bash
# Dalla directory del progetto
# Su Windows PowerShell:
php vendor\bin\drush.php serve

# Su Git Bash / Linux / Mac:
php vendor/bin/drush.php serve
```

Il sito sarà disponibile su: **http://127.0.0.1:8888**

### Opzioni aggiuntive:

```bash
# Specificare host e porta personalizzati
php vendor/bin/drush.php serve --host=127.0.0.1 --port=8080

# Con output dettagliato
php vendor/bin/drush.php serve -vvv
```

### Metodo 2: PHP Built-in Server

```bash
cd web
php -S localhost:8000
```

Il sito sarà disponibile su: **http://localhost:8000**

## 🌐 Deploy in Produzione

**⚠️ IMPORTANTE**: NON usare `drush serve` in produzione!

### Configurazione Apache

1. **Creare un Virtual Host** (`/etc/apache2/sites-available/dnd-drupal.conf`):

```apache
<VirtualHost *:80>
    ServerName tuodominio.com
    ServerAlias www.tuodominio.com
    
    DocumentRoot /path/to/my-project/web
    
    <Directory /path/to/my-project/web>
        AllowOverride All
        Require all granted
    </Directory>
    
    ErrorLog ${APACHE_LOG_DIR}/dnd-drupal-error.log
    CustomLog ${APACHE_LOG_DIR}/dnd-drupal-access.log combined
</VirtualHost>
```

2. **Abilitare il sito**:
```bash
sudo a2ensite dnd-drupal
sudo a2enmod rewrite
sudo systemctl restart apache2
```

### Configurazione Nginx

1. **Creare un Server Block** (`/etc/nginx/sites-available/dnd-drupal`):

```nginx
server {
    listen 80;
    server_name tuodominio.com www.tuodominio.com;
    root /path/to/my-project/web;
    
    index index.php index.html;
    
    location / {
        try_files $uri /index.php?$query_string;
    }
    
    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
    
    location ~ /\.ht {
        deny all;
    }
}
```

2. **Abilitare il sito**:
```bash
sudo ln -s /etc/nginx/sites-available/dnd-drupal /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

### Configurazione Database Produzione

1. **Creare file `settings.local.php`** in `web/sites/default/`:

```php
<?php
// Settings per ambiente di produzione
$databases['default']['default'] = [
  'database' => 'dnd_drupal_prod',
  'username' => 'username_produzione',
  'password' => 'password_sicura',
  'host' => 'localhost',
  'port' => '3306',
  'driver' => 'mysql',
  'prefix' => '',
  'collation' => 'utf8mb4_general_ci',
];

// Disabilitare il debug in produzione
$config['system.logging']['error_level'] = 'hide';
$config['system.performance']['css']['preprocess'] = TRUE;
$config['system.performance']['js']['preprocess'] = TRUE;
```

2. **Abilitare `settings.local.php` in `settings.php`** (decommenta le ultime righe):

```php
if (file_exists($app_root . '/' . $site_path . '/settings.local.php')) {
  include $app_root . '/' . $site_path . '/settings.local.php';
}
```

## 🛠️ Comandi Utili Drush

**Nota**: Su Git Bash/Linux/Mac usa `/` invece di `\` nei percorsi.

### Verifica stato del sito
```bash
php vendor/bin/drush.php status
```

### Pulire cache
```bash
php vendor/bin/drush.php cache:rebuild
# oppure versione breve
php vendor/bin/drush.php cr
```

### Aggiornare database dopo modifiche
```bash
php vendor/bin/drush.php updatedb
# oppure
php vendor/bin/drush.php updb
```

### Esportare/Importare configurazione
```bash
# Esportare configurazione
php vendor/bin/drush.php config:export
# oppure
php vendor/bin/drush.php cex

# Importare configurazione
php vendor/bin/drush.php config:import
# oppure
php vendor/bin/drush.php cim
```

### Login rapido come admin
```bash
# Con URI corretto (quando usi drush serve)
php vendor/bin/drush.php uli --uri=http://127.0.0.1:8888

# Versione completa
php vendor/bin/drush.php user:login --uri=http://127.0.0.1:8888
```

## 📦 Gestione Dipendenze

### Installare nuove dipendenze
```bash
composer require drupal/module_name
```

### Aggiornare Drupal Core e moduli
```bash
composer update drupal/core "drupal/core-*" --with-all-dependencies
```

### Aggiornare tutti i pacchetti
```bash
composer update
```

## 🔒 Sicurezza in Produzione

### Checklist:
- [ ] Cambiare password database da `admin/admin` a credenziali sicure
- [ ] Rimuovere o proteggere `install.php` e `update.php`
- [ ] Impostare permessi corretti sui file (755 per directory, 644 per file)
- [ ] Configurare HTTPS con certificato SSL
- [ ] Impostare `settings.php` in sola lettura: `chmod 444 settings.php`
- [ ] Configurare trusted host patterns in `settings.php`:

```php
$settings['trusted_host_patterns'] = [
  '^tuodominio\.com$',
  '^www\.tuodominio\.com$',
];
```

## 🐛 Troubleshooting

### Il sito mostra una pagina bianca
```bash
# Controlla i log di errore
php vendor/bin/drush.php watchdog:show

# Pulisci la cache
php vendor/bin/drush.php cr
```

### Errori di permessi
```bash
# Linux/Mac
chmod -R 755 web/sites/default/files
chown -R www-data:www-data web/sites/default/files

# Verificare proprietà
ls -la web/sites/default/
```

### Database connection error
- Verificare che MySQL/MariaDB sia in esecuzione
- Controllare credenziali in `settings.php`
- Verificare che il database esista

```bash
# Testare connessione MySQL
mysql -u admin -p -h localhost dnd-drupal
```

## 📚 Risorse Utili

- [Documentazione Drupal](https://www.drupal.org/docs)
- [Drush Commands](https://www.drush.org/latest/commands/)
- [Drupal Security](https://www.drupal.org/security)

## 📝 Note

- **Sviluppo**: Usa sempre `php vendor/bin/drush.php serve` per test locali
- **Produzione**: Configura un web server appropriato (Apache/Nginx)
- **Backup**: Esegui backup regolari del database e dei file
- **Aggiornamenti**: Mantieni Drupal Core e i moduli aggiornati per sicurezza
- **Windows**: Su Git Bash usa `/`, su PowerShell usa `\` nei percorsi

---

**Versione Drupal**: 10.x  
**Ultima modifica**: 26 Novembre 2025
