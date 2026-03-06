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
