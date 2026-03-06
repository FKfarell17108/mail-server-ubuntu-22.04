# Mail Server Installation Guide
Ubuntu 22.04 (Postfix + Dovecot + Roundcube)

This document explains the full process of installing and configuring a mail server in Ubuntu 22.04 inside VMware.

Local domain used:

mail.farell.local

---

# 1. Update Package

Update the system before installing packages.
```bash
sudo apt update && sudo apt upgrade -y
```

---

# 2. Configure Hostname

Set the hostname for the mail server.
```bash
sudo hostnamectl set-hostname mail.farell.local
```

Edit the hosts file.
```bash
sudo nano /etc/hosts
```

Add the following configuration.
```bash
127.0.0.1 localhost
127.0.1.1 mail.farell.local mail
```

Verify hostname.
```bash
hostname
```

Expected output: ```mail.farell.local```

---

# 3. Install Postfix (SMTP Server)

Install postfix.
```bash
sudo apt install postfix -y
```

During installation choose: ```Internet Site```

System mail name: ```farell.local```

Edit postfix configuration.
```bash
sudo nano /etc/postfix/main.cf
```

Make sure the configuration contains:
```bash
myhostname = mail.farell.local
mydomain = farell.local
myorigin = /etc/mailname

inet_interfaces = all
inet_protocols = ipv4

mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain

home_mailbox = Maildir/
```

Restart postfix.
```bash
sudo systemctl restart postfix
```

---

# 4. Install Dovecot (IMAP and POP3)

Install dovecot services.
```bash
sudo apt install dovecot-imapd dovecot-pop3d -y
```

Configure Maildir.
```bash
sudo nano /etc/dovecot/conf.d/10-mail.conf
```

Find: ```mail_location =```

Change to: ```mail_location = maildir:~/Maildir```

---

# Enable Authentication

Edit authentication configuration.
```bash
sudo nano /etc/dovecot/conf.d/10-auth.conf
```

Make sure these settings exist.
```bash
disable_plaintext_auth = no
auth_mechanisms = plain login
```

Restart dovecot.
```bash
sudo systemctl restart dovecot
```

Check service status.
```bash
sudo systemctl status dovecot
```

---

# 5. Create Email User

In this setup, Linux users are used as email accounts.

Create a user.
```bash
sudo adduser user1
sudo adduser user2
```

Email address will be: ```user1@farell.local``` & ```user2@farell.local```

---

# 6. Install Apache and PHP

Install Apache and required PHP modules.
```bash
sudo apt install apache2 php php-mysql php-intl php-mbstring php-xml php-zip php-curl -y
```

Restart apache.
```bash
sudo systemctl restart apache2
```

---

# 7. Install MariaDB

Install database server.
```bash
sudo apt install mariadb-server -y
```

Secure the installation.
```bash
sudo mysql_secure_installation
```

---

# 8. Create Roundcube Database

Login to MariaDB.
```bash
sudo mysql -u root -p
```

Create database.
```bash
CREATE DATABASE roundcube;
```

Create user.
```bash
CREATE USER 'roundcube'@'localhost' IDENTIFIED BY 'passwordku';
```

Grant privileges.
```bash
GRANT ALL PRIVILEGES ON roundcube.* TO 'roundcube'@'localhost';
```

Apply changes.
```bash
FLUSH PRIVILEGES;
EXIT;
```

---

# 9. Install Roundcube

Install Roundcube packages.
```bash
sudo apt install roundcube roundcube-mysql roundcube-core -y
```

When prompted: ```Configure database for roundcube → YES```

Enter database password.

---

# 10. Configure Roundcube

Edit configuration.
```bash
sudo nano /etc/roundcube/config.inc.php
```

Add or ensure the following configuration exists.
```bash
$config['default_host'] = 'localhost';

$config['smtp_server'] = 'localhost';
$config['smtp_port'] = 25;

$config['smtp_user'] = '%u';
$config['smtp_pass'] = '%p';
```

Restart apache.
```bash
sudo systemctl restart apache2
```

---

# 11. Enable Roundcube in Apache

Edit apache configuration.
```bash
sudo nano /etc/apache2/apache2.conf
```

Add at the bottom.
```bash
Include /etc/roundcube/apache.conf
```

Restart apache.
```bash
sudo systemctl restart apache2
```

---

# 12. Access Webmail

Open browser.
```bash
http://SERVER-IP/roundcube
```

Login example:

username
```bash
user1
```

password
```bash
(user1 password)
```

Email address automatically becomes: ```user1@farell.local```

---

# Troubleshooting

## Roundcube Not Found

Create symbolic link.
```bash
sudo ln -s /usr/share/roundcube /var/www/html/roundcube
```

Check directory.
```bash
ls /var/www/html
```

Expected result.
```bash
index.html
roundcube
```

Restart apache.
```bash
sudo systemctl restart apache2
```

Open again.
```bash
http://SERVER-IP/roundcube
```

---

## SMTP Authentication Failed

### Enable SASL Authentication in Postfix

Edit postfix configuration.
```bash
sudo nano /etc/postfix/main.cf
```

Ensure the following settings exist.
```bash
smtpd_sasl_auth_enable = yes
smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth

smtpd_sasl_security_options = noanonymous
broken_sasl_auth_clients = yes
```

Add recipient restrictions.
```bash
smtpd_recipient_restrictions =
permit_sasl_authenticated,
permit_mynetworks,
reject_unauth_destination
```

---

### Configure Dovecot Authentication Socket

Edit configuration.
```bash
sudo nano /etc/dovecot/conf.d/10-master.conf
```

Find: ```service auth {```

Add inside.
```bash
unix_listener /var/spool/postfix/private/auth {
mode = 0660
user = postfix
group = postfix
}
```

---

### Restart Services

Restart all mail services.
```bash
sudo systemctl restart postfix
sudo systemctl restart dovecot
sudo systemctl restart apache2
```

If successful, SMTP authentication should work correctly.
