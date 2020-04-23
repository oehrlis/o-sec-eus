## Solution 1: Database Authentication

The following steps are performed in this exercise:

- Checking the current Password Verifier
- Adjusting the Password Verifier
- Optional tests with legacy Password Verifier.

<!-- Stuff between the <div class="notes"> will be rendered as pptx slide notes -->
<div class="notes">
</div>

<!-- Stuff between the <div class="no notes"> will not be rendered as pptx slide notes -->
<div class="no notes">

### Detailed Solution

#### Checking the current Password Verifier

1. Check what password hashes are currently available in the database. Which hashes are available? Why are there no entries in *password_versions* for certain users?

```SQL
set linesize 120 pagesize 200
col USERNAME for a25
SELECT username, password_versions FROM dba_users;
```

2. Check how the VIEW *dba_users* gets the information about *password_versions*. In the code for the view *dba_users* you will find corresponding *decode* functions where the columns *u.password* and *u.spare4* are accessed.

```SQL
set linesize 120 pagesize 200
set long 200000
SELECT text FROM dba_views WHERE view_name='DBA_USERS';
```

3. What kind of password hashes does the user SCOTT have effectively?

```SQL
set linesize 120 pagesize 200
col password for a16
col spare4 for a40
SELECT password, spare4 FROM user$ WHERE name='SCOTT';
```

4. Check what has been defined in the file ``sqlnet.ora`` for the parameters *ALLOWED_LOGON_VERSION_**. Use alternative ``cat``, ``less``, ``more`` or ``vi`` to display the contents of ``sqlnet.ora``.

```bash
less $cdn/admin/sqlnet.ora

cat $cdn/admin/sqlnet.ora|grep -i ALLOWED_LOGON_VERSION
```

5. Check what is effectively used by SQLNet.

* Switch on SQLNet Tracing on the client side. Setting *DIAG_ADR_ENABLED* and *TRACE_LEVEL_CLIENT*. Enclosed, replace manually with ``vi`` or alternatively directly with ``sed``.

```bash
vi $cdn/admin/sqlnet.ora
DIAG_ADR_ENABLED=OFF
TRACE_LEVEL_CLIENT=SUPPORT
```

```bash
sed -i "s|DIAG_ADR_ENABLED.*|DIAG_ADR_ENABLED=OFF|" $cdn/admin/sqlnet.ora
sed -i "s|TRACE_LEVEL_CLIENT.*|TRACE_LEVEL_CLIENT=SUPPORT|" $cdn/admin/sqlnet.ora
```

* Delete any old trace files.

```bash
rm $cdn/trc/sqlnet_client_*.trc
```

* Connect as user SCOTT

```bash
sqlplus scott/tiger

show user
```

* Check the trace file. What is set for ALLOWED_LOGON_VERSION? If nothing is set, what is the value?

```bash
ls -rtl $cdn/trc
less $cdn/trc/sqlnet_client_*.trc

grep -i ALLOWED_LOGON_VERSION $cdn/trc/sqlnet_client_*.trc
```

* Switch the tracing off again.

```bash
vi $cdn/admin/sqlnet.ora
DIAG_ADR_ENABLED=ON
TRACE_LEVEL_CLIENT=OFF
```

```bash
sed -i "s|DIAG_ADR_ENABLED.*|DIAG_ADR_ENABLED=ON|" $cdn/admin/sqlnet.ora
sed -i "s|TRACE_LEVEL_CLIENT.*|TRACE_LEVEL_CLIENT=OFF|" $cdn/admin/sqlnet.ora
```

#### Adjusting the Password Verifier

1. Delete the Oracle 12c password hash from user Scott. Respectfully set the 11g hashes explicitly.

```SQL
SELECT spare4 FROM user$ WHERE name='SCOTT';

set linesize 170
col 11G_HASH for a62

SELECT 
    REGEXP_SUBSTR(spare4,'(S:[[:alnum:]]+)') "11G_HASH"
FROM user$ WHERE name='SCOTT';

col 12C_HASH for a162
SELECT 
    REGEXP_SUBSTR(spare4,'(T:[[:alnum:]]+)') "12C_HASH"
FROM user$ WHERE name='SCOTT';

ALTER USER scott IDENTIFIED BY VALUES 'S:54A0B23AE639D4E0E22963A65A380DD496B8FCB65D1A5F9CC910EE625D8C';
```

2. Control the *password_versions* of the user *SCOTT*.

```SQL
col username for a30
SELECT username,password_versions FROM dba_users WHERE username='SCOTT';
```

3. Adjust the SQLNet parameter *ALLOWED_LOGON_VERSION_SERVER* and set the 12a authentication protocol

```bash
sed -i "s|#SQLNET.ALLOWED_LOGON_VERSION_SERVER.*|SQLNET.ALLOWED_LOGON_VERSION_SERVER=12a|" $cdn/admin/sqlnet.ora
sed -i "s|SQLNET.ALLOWED_LOGON_VERSION_SERVER.*|SQLNET.ALLOWED_LOGON_VERSION_SERVER=12a|" $cdn/admin/sqlnet.ora
```

4. Connect as User Scott. Is it even possible to connect?

```bash
sqlplus scott/tiger

show user
```

5. Adjust the SQLNet parameter *ALLOWED_LOGON_VERSION_CLIENT* and *ALLOWED_LOGON_VERSION_SERVER*, set the 11 authentication protocol

```bash
sed -i "s|#ALLOWED_LOGON_VERSION_CLIENT.*|SQLNET.ALLOWED_LOGON_VERSION_CLIENT=11|" $cdn/admin/sqlnet.ora
sed -i "s|SQLNET.ALLOWED_LOGON_VERSION_CLIENT.*|SQLNET.ALLOWED_LOGON_VERSION_CLIENT=11|" $cdn/admin/sqlnet.ora
sed -i "s|#SQLNET.ALLOWED_LOGON_VERSION_SERVER.*|SQLNET.ALLOWED_LOGON_VERSION_SERVER=11|" $cdn/admin/sqlnet.ora
sed -i "s|SQLNET.ALLOWED_LOGON_VERSION_SERVER.*|SQLNET.ALLOWED_LOGON_VERSION_SERVER=11|" $cdn/admin/sqlnet.ora
```

6. Connect as User Scott. Is it even possible to connect?

```bash
sqlplus scott/tiger

show user
```

7. What happens if you set the password of *SCOTT* as *SYS*? Which password hash has *SCOTT* now?

```SQL
connect / as sysdba

ALTER USER scott IDENTIFIED BY tiger;

set linesize 120 pagesize 200
col USERNAME for a25
col password for a16
col spare4 for a40
SELECT username, password_versions FROM dba_users WHERE username='SCOTT';
SELECT password, spare4 FROM user$ WHERE name='SCOTT';
```

#### Optional Optional Tasks: Tests with legacy Password Verifier.

If there is still time left, the following additional tasks may be useful:

- setting *ALLOWED_LOGON_VERSION_CLIENT* and *ALLOWED_LOGON_VERSION_SERVER* to values less than 11 e.g. 10, 9 or 8. What does the user *SCOTT* get for password hashes if you set the password with *ALTER USER* as *SYS*?
- Which password is used for login?

</div>
