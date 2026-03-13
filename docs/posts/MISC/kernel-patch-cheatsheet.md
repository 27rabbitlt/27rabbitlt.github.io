# Kernel patch email cheat sheet

## 1. What you are building

You want this toolchain:

* **git** — commits
* **git send-email** — send patches
* **msmtp** — SMTP sender
* **neomutt** — plain-text email client
* **mbsync** — fetch Gmail mail via IMAP to local Maildir

Recommended workflow:

1. write code
2. `git commit -s`
3. `git format-patch`
4. `git send-email`
5. read/reply to review in `neomutt`

---

## 2. Install packages on Arch

```bash
sudo pacman -S git msmtp neomutt isync perl-authen-sasl perl-io-socket-ssl ca-certificates
```

Check:

```bash
git --version
git send-email --version
msmtp --version
neomutt -v | head
mbsync --version
```

---

## 3. Git identity

Use your **real name** and the **same email address** everywhere.

```bash
git config --global user.name "Teng Liu"
git config --global user.email "27rabbitlt@gmail.com"
git config --global core.editor "vim"
git config --global sendemail.annotate yes
git config --global format.subjectPrefix "PATCH"
```

Check:

```bash
git config --global --list
```

Important rule:

* `git user.email`
* `sendemail.from`
* `msmtp from`

should all match.

---

## 4. `~/.msmtprc`

Create it:

```bash
touch ~/.msmtprc
chmod 600 ~/.msmtprc
mkdir -p ~/.config/msmtp
chmod 700 ~/.config/msmtp
```

Example config:

```conf
defaults
auth           on
tls            on
tls_trust_file /etc/ssl/certs/ca-certificates.crt
logfile        ~/.msmtp.log

account        default
host           smtp.gmail.com
port           587
from           27rabbitlt@gmail.com
user           27rabbitlt@gmail.com
passwordeval   "cat ~/.config/msmtp/gmail_app_password"
```

Store Gmail app password:

```bash
printf '%s\n' 'YOUR_GMAIL_APP_PASSWORD' > ~/.config/msmtp/gmail_app_password
chmod 600 ~/.config/msmtp/gmail_app_password
```

---

## 5. Test `msmtp`

Direct test:

```bash
printf "Subject: msmtp test\n\nhello\n" | msmtp -a default 27rabbitlt@gmail.com
```

Check log:

```bash
tail -n 50 ~/.msmtp.log
```

Useful test:

```bash
msmtp --serverinfo -a default
```

If this works, `msmtp` is fine.

---

## 6. Correct `git send-email` config

Set it like this:

```bash
git config --global sendemail.smtpserver /usr/bin/msmtp
git config --global --unset-all sendemail.smtpserveroption
git config --global --add sendemail.smtpserveroption -a
git config --global --add sendemail.smtpserveroption default
git config --global sendemail.from "Teng Liu <27rabbitlt@gmail.com>"
```

Check:

```bash
git config --global --get-regexp '^sendemail\.'
```

Expected style:

```text
sendemail.annotate yes
sendemail.smtpserver /usr/bin/msmtp
sendemail.smtpserveroption -a
sendemail.smtpserveroption default
sendemail.from Teng Liu <27rabbitlt@gmail.com>
```

---

## 7. Minimal `neomutt` sending config

Create config dir:

```bash
mkdir -p ~/.config/neomutt
```

Example `~/.config/neomutt/neomuttrc`:

```conf
set realname = "Teng Liu"
set from = "27rabbitlt@gmail.com"
set use_from = yes
set envelope_from = yes

set editor = "vim"
set sendmail = "/usr/bin/msmtp -a default"
```

---

## 8. Why use NeoMutt instead of Gmail?

Main reason:

* **kernel patch review expects plain-text, inline replies**
* Gmail is okay for checking mail
* NeoMutt is better for replying correctly to review comments

Best practical split:

* **Gmail** for convenience/checking
* **NeoMutt** for serious mailing-list replies

---

## 9. Gmail IMAP settings: what to choose

In Gmail settings:

* **Enable IMAP**
* **Auto-Expunge off**
* **Archive the message**
* **Do not limit the number of messages**

Reason:

* safer
* less chance of deleting review mail by accident
* better for mailing-list workflow

---

## 10. `mbsync` config for reading mail in NeoMutt

Create:

```bash
touch ~/.mbsyncrc
chmod 600 ~/.mbsyncrc
mkdir -p ~/Mail/gmail
```

Example `~/.mbsyncrc`:

```conf
IMAPAccount gmail
Host imap.gmail.com
Port 993
User 27rabbitlt@gmail.com
PassCmd "cat ~/.config/msmtp/gmail_app_password"
TLSType IMAPS
CertificateFile /etc/ssl/certs/ca-certificates.crt

IMAPStore gmail-remote
Account gmail

MaildirStore gmail-local
SubFolders Verbatim
Path ~/Mail/gmail/
Inbox ~/Mail/gmail/INBOX

Channel gmail-inbox
Far :gmail-remote:
Near :gmail-local:
Patterns INBOX "[Gmail]/Sent Mail" "[Gmail]/Drafts" "[Gmail]/Trash"
Create Near
SyncState *
Sync All
```

First sync:

```bash
mbsync -a
```

Check downloaded folders:

```bash
find ~/Mail/gmail -maxdepth 2 -type d | sort
```

---

## 11. What `mbsync -a` progress means

Example:

```text
C: 0/1  B: 0/3  F: +0/0 *0/0 #0/0 -0/0  N: +343/11280 *0/0 #0/0 -0/0
```

Meaning:

* **C** = channels
* **B** = mailboxes
* **F** = far = remote Gmail side
* **N** = near = local Maildir side

Symbols:

* `+` added messages
* `*` changed messages
* `#` flag changes
* `-` removed messages

So `N: +343/11280` means it has fetched 343 local messages out of 11280 total planned.

That is normal on first sync.

Verbose mode:

```bash
mbsync -a -V
```

---

## 12. Point NeoMutt at synced Gmail mail

Update `~/.config/neomutt/neomuttrc`:

```conf
set realname = "Teng Liu"
set from = "27rabbitlt@gmail.com"
set use_from = yes
set envelope_from = yes

set sendmail = "/usr/bin/msmtp -a default"
set editor = "vim"

set folder = "~/Mail/gmail"
set spoolfile = "+INBOX"
set record = "+[Gmail]/Sent Mail"
set postponed = "+[Gmail]/Drafts"
set trash = "+[Gmail]/Trash"

set mbox_type = Maildir
set copy = yes
set move = no
set wait_key = no
set edit_headers = yes
set text_flowed = no
set charset = "utf-8"

mailboxes +INBOX
```

Open mail:

```bash
neomutt
```

Useful keys:

* `Enter` open mail
* `i` index
* `r` reply
* `m` compose
* `g` jump folder
* `c` change mailbox
* `q` quit

Refresh mail:

```bash
mbsync -a
```

---

## 13. Test `git send-email`

Make a tiny test repo:

```bash
mkdir ~/tmp/git-mail-test
cd ~/tmp/git-mail-test
git init
printf 'hello\n' > README
git add README
git commit -s -m "test: add README"
git format-patch -1
```

Dry run:

```bash
git send-email --dry-run --to 27rabbitlt@gmail.com 0001-test-add-README.patch
```

Real send:

```bash
git send-email --to 27rabbitlt@gmail.com 0001-test-add-README.patch
```

---

## 14. Commands you will use all the time

### Git identity

```bash
git config --global user.name "Teng Liu"
git config --global user.email "27rabbitlt@gmail.com"
```

### Check send-email config

```bash
git config --global --get-regexp '^sendemail\.'
```

### Test SMTP

```bash
printf "Subject: test\n\nhello\n" | msmtp -a default 27rabbitlt@gmail.com
```

### Sync Gmail

```bash
mbsync -a
```

### Open mail

```bash
neomutt
```

### Send patch

```bash
git send-email --to SOMEONE@example.com 0001-something.patch
```

### Commit with signoff

```bash
git commit -s
```

---

## 15. Common mistakes

* using different email addresses in Git and SMTP
* using normal Gmail password instead of app password
* setting `sendemail.smtpserveroption` as one combined string
* trying to reply to patch review in HTML mail
* deleting mailing-list mail instead of archiving
* forgetting `git commit -s`

---

## 16. Best current state for you

You already have:

* Arch Linux
* `msmtp` working
* `neomutt` able to send mail

Next likely steps are:

1. finish `mbsync` sync
2. open synced mail in `neomutt`
3. make a test repo
4. send a test patch to yourself
5. move to kernel repo setup

