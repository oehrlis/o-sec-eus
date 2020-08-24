## Solution 3: Kerberos Authentication

Configuration of Kerberos Authentication and successful login locally (Linux VM):

- Review AD configuration and service principle
- Setup DB Server for Kerberos
- Creating a user with Kerberos authentication and successful login locally (Linux VM)
- Everybody can use kerberos authentication.

<!-- Stuff between the <div class="notes"> will be rendered as pptx slide notes -->
<div class="notes">
</div>

<!-- Stuff between the <div class="no notes"> will not be rendered as pptx slide notes -->
<div class="no notes">

### Detailed Solution

The following steps have been performed on the *ol7docker00* host. If necessary, adjust the commands, filenames or the host name according to your environment.
**Übungsziele:** 

## Service Principle und Keytab Datei

Arbeitsumgebung für die Übung

* **Server:** ad.trivadislabs.com
* **Benutzer:** Administrator

Für die Kerberos Authentifizierung wird ein Service Principle benötigt. Der Entsprechende Benutzer Account wurde vorbereitet. Kontrollieren Sie in auf dem Server *ad.trivadislabs.com* mit dem Tool *Active Directory User and Computers* ob der Benutzer *db.trivadislabs.com* existiert. Falls ja, was hat der Benutzer für Einstellungen bezüglich Login Name und Account optionen? Passen Sie ggf noch die Account Optionen an uns setzen *Kerberos AES 128* und *Kerberos AES 256*. Die folgende Abbildung zeigt ein Beispiel. Optional können Sie den Benutzer auch löschen und neu anlegen.

!["Benutzereigenschaften"](images/User_Account_Preferences.png)  

Nachdem die Account Optionen angepasst wurden, ist für diesen Benutzer eine Keytab Datei zu erstellen. Öffnen Sie dazu ein Command Prompt (``cmd.exe``) und führen ``ktpass.exe`` aus.

```bash
ktpass.exe -princ oracle/db.trivadislabs.com@TRIVADISLABS.COM -mapuser db.trivadislabs.com -pass LAB01schulung -crypto ALL -ptype KRB5_NT_PRINCIPAL  -out C:\oracle\network\admin\db.trivadislabs.com.keytab
```

Überprüfen Sie anschliessend den Service Principle Names (SPN) mit ``setspn``

```bash
setspn -L db.trivadislabs.com
```

Kopieren Sie die Keytab Datei mit WinSCP auf den Datenbank Sever in das Verzeichnis ``$cdn/admin``. Achten Sie darauf, dass die Datei als Binärdatei kopiert wird. Alternativ können Sie auch das unten aufgeführte Putty SCP Kommando verwenden.

```bash
"C:\Program Files\PuTTY\pscp.exe" C:\u00\app\oracle\network\admin\db.trivadislabs.com.keytab db.trivadislabs.com:/u00/app/oracle/network/admin
```

## SQLNet Konfiguration

Arbeitsumgebung für die Übung:

* **Server:** db.trivadislabs.com
* **Benutzer:** oracle

Ergänzen Sie die ``sqlnet.ora`` Datei mit folgenden Parametern. Eine Beispiel Konfigurationsdatei ist im Verzeichnis ``$cdl/doag2018/lab/03_krb`` vorhanden.

```bash
cp $cdl/doag2018/lab/03_krb/sqlnet.ora $cdn/admin/sqlnet.ora
vi $cdn/admin/sqlnet.ora
##########################################################################
# Kerberos Configuration
##########################################################################
SQLNET.AUTHENTICATION_SERVICES = (BEQ,KERBEROS5)
SQLNET.FALLBACK_AUTHENTICATION = TRUE
SQLNET.KERBEROS5_KEYTAB = /u00/app/oracle/network/admin/db.trivadislabs.com.keytab
SQLNET.KERBEROS5_REALMS = /u00/app/oracle/network/admin/krb.realms
SQLNET.KERBEROS5_CC_NAME = /u00/app/oracle/network/admin/krbcache
SQLNET.KERBEROS5_CONF = /u00/app/oracle/network/admin/krb5.conf
SQLNET.KERBEROS5_CONF_MIT=TRUE
SQLNET.AUTHENTICATION_KERBEROS5_SERVICE = oracle
```

Erstellen Sie die Kerberos Konfigurationsdatei ``krb5.conf`` mit folgendem Inhalt. Eine Beispiel Konfigurationsdatei ist im Verzeichnis ``$cdl/doag2018/lab/03_krb`` vorhanden.

```bash
cp $cdl/doag2018/lab/03_krb/krb5.conf $cdn/admin/krb5.conf
vi $cdn/admin/krb5.conf 
[libdefaults]
 default_realm = TRIVADISLABS.COM
 clockskew=300
 ticket_lifetime = 24h
 renew_lifetime = 7d
 forwardable = true

[realms]
 TRIVADISLABS.COM = {
   kdc = ad.trivadislabs.com
   admin_server = ad.trivadislabs.com
}

[domain_realm]
.trivadislabs.com = TRIVADISLABS.COM
trivadislabs.com = TRIVADISLABS.COM
```

Nachdem die Keytab Datei auf dem Datenbank Server kopiert worden ist, kann mit ``oklist`` überprüft werden was die Datei für Crypto Algorithmen untersützt. Somit wird zudem indirekt geprüft ob die Keytab Datei verwendet werden kann.

```bash
oklist -e -k $cdn/admin/db.trivadislabs.com.keytab
```

Kontrollieren Sie ob die Namensauflösung wie gewünscht funktioniert.

```bash
nslookup ad.trivadislabs.com
nslookup 10.0.0.4
nslookup db.trivadislabs.com
nslookup 10.0.0.5
```

Erstellen Sie anschliessend mit``okinit`` manuell ein Session Ticket.

```Bash
okinit king@TRIVADISLABS.COM

oklist
```

## Kerberos Authentifizierung

Arbeitsumgebung für die Übung:

* **Server:** db.trivadislabs.com
* **Benutzer:** oracle
  
Passen Sie den init.ora Parameter OS Prefix an. Für die Kerberos Authentifizierung muss dieser leer sein.

```SQL
sqh
Show parameter os_authent_prefix
ALTER SYSTEM SET os_authent_prefix='' SCOPE=spfile;
STARTUP FORCE;
```

Erstellen Sie einen Kerberos Benutzers für den Mitarbeiter King. Verwenden Sie dazu die Variante mit dem Kerberos Principal Name.

```SQL
CREATE USER king IDENTIFIED EXTERNALLY AS 'king@TRIVADISLABS.COM';
GRANT CONNECT TO king;
GRANT SELECT ON v_$session TO king;
```

Login als Benutzer King mit dem zuvor generierten Session Ticket und anzeigen der Informationen zur aktuellen Session.

```SQL

connect /@TDB184A

show user

@sousrinf

SELECT sys_context('USERENV','AUTHENTICATION_TYPE') FROM DUAL;
SELECT sys_context('USERENV','AUTHENTICATION_METHOD') FROM DUAL;
SELECT sys_context('USERENV','AUTHENTICATED_IDENTITY') FROM DUAL;
```

## Zusatzaufgaben

Falls noch Zeit übrig ist, bieten sich folgende Zusatzaufgaben an:

* Versuchen Sie einen weiteren Kerberos Benutzer in der Datebank zu erstellen (z.B. *adams*). Dabei nutzen Sie aber den UPN als Benutzernamen. Wie setzt sich dieser genau zusammen? Wie wird dieser bei einem ``CREATE USER`` geschreiben, damit die Authentifizierung klappt?
* Kombinieren Sie Kerberos Authentifizierung mit Proxy Authentifizierung.
* Locken Sie im AD den Benutzer und versuchen erneut mit der Datenbank zu verbinden.
* Versuchen Sie sich als Benutzer *king* am Active directory Server anzumelden und eine SQLPlus Verbindung auf den Datenbank Server zu öffnen. Hierzu müssen Sie vorgänging noch die Kerberos Client Konfiguration erstellen (sqlnet.ora sowie die krb5.conf Datei erstellen).

</div>
