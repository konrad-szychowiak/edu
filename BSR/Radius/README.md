# Przygotowanie systemu do ćwiczeń
Do wykonania ćwiczeń niezbędne jest następujące oprogramowanie:
• serwer i klient FreeRADIUS,
• baza danych PostgreSQL.

Serwer FreeRADIUS należy zainstalować z odpowiedniego pakietu z repozytorium dla bieżącej dystrybucji systemu Linux. Gotowe pakiety dostępne są też na stronie projektu FreeRADIUS. W przypadku dystrybucji OpenSuSE potrzebne będą pakiety `freeradius-server`, `freeradius-server-utils`, `freeradius-server-postgresql` oraz
`freeradius-client` (wraz z niezbędnymi bibliotekami, które należą do zależności powyższych pakietów i zapewne zostaną automatycznie doinstalowane przez systemowego zarządcę pakietów).

```console
# zypper se freeradius
i | freeradius-client
i | freeradius-server
i | freeradius-server-utils
...
# zypper in freeradius-server freeradius-server-utils freeradius-server-postgresql freeradius-client
...
```

Po zainstalowaniu można sprawdzić jakie pliki zostały dodade:
```console
# rpm -ql freeradius-server | grep conf$
/etc/raddb/clients.conf
/etc/raddb/radiusd.conf
...
# rpm -ql freeradius-server | grep -w users
/etc/raddb/users
```

Usługą można sterować jak wszystkimi usługami systemowymi, np.:
```console
# service freeradius start
```

Alternatywnie można też uruchomić serwer „ręcznie” w trybie diagnostycznym (debug):
```console
# /usr/sbin/radiusd -X
```

# Zadania wstępne
W poniższych zadaniach wykorzystamy tylko Radiusa z plikiem definiującym użytkowników, jeszcze bez bazy danych.

## Zadanie 1. Skonfiguruj klienta FreeRADIUS
![[Pasted image 20260316155214.png]]

## Zadanie 2. Zdefiniuj użytkownika w bazie plikowej `/etc/raddb/users`
```
bob	Cleartext-Password := "bob"
	Reply-Message := "Hello, %{User-Name}"
```

## Zadanie (3) Uruchom serwer w trybie diagnostycznym i narzędziem `radtest` przetestuj uwierzytelnianie użytkownika lokalnie.

EAP certs:
```console
# cd /etc/raddb/certs/
# ./bootstrap
```
debug mode:
```console
# /usr/sbin/radiusd -X
```
test:
```console
$ radtest -x bob bob 127.0.0.1 10000 testing123
```

# PostgreSQL

> [!WARNING]
> Zanim zaczniesz zadania, wykomentuj `bob`-a z `/etc/raddb/users`.

## Zadanie: (4) Skonfiguruj współpracę serwera FreeRADIUS z bazą danych PostgreSQL.
```console
# zypper in postgresql postgresql-server
# service postgresql start
# ss -ltp
```

```console
# ln -s /etc/raddb/mods-available/sql /.../mods-enabled/
# vi /etc/raddb/mods-enabled/sql
dialect = "postgresql"
radius_db = "dbname=radius host=localhost user=radius password=raddpass"
```

```console
# su - postgres
$ psql -c "CREATE USER radius WITH PASSWORD 'raddpass';"
$ psql -c "CREATE DATABASE radius OWNER radius;"
```

```console
$ rpm -ql freeradius-server-postgresql
...
/etc/raddb/mods-config/sql/main/postgresql/schema.sql
/etc/raddb/mods-config/sql/main/postgresql/setup.sql
...
$ psql -d radius -a -f /etc/raddb/mods-config/sql/main/postgresql/schema.sql
```


```console
# vi /var/lib/pgsql/data/pg_hba.conf
local	all	all					password
host	all	all	127.0.0.1/32	password
# service postgresql reload
# psql -U radius -a -f /.../schema.sql
# psql -U radius -a -f /.../setup.sql
```

```console
$ psql -U radius
radius=> \d
Lista relacji
Schemat  | Nazwa                 | Typ       | Właściciel
---------+-----------------------+-----------+------------
public   | nas			         | tabela 	 | radius
public	 | nas_id_seq	         | sekwencja | radius
public	 | radacct		         | tabela 	 | radius
```

## Zadanie: (5) Zdefiniuj użytkownika w bazie danych i przetestuj uwierzytelnianie narzędziem radtest.

```sql
radius=> INSERT INTO radcheck (username, attribute, op, value)
VALUES ('jbond','Cleartext-Password',':=','SecretService');

radius=> INSERT INTO radreply (username, attribute, op, value)
VALUES ('jbond','Reply-Message',':=','Hello jbond!');
```

```console
$ radtest -x jbond SecretService 127.0.0.1 10000 testing123
```

```sql
radius=> \d+
public | radpostauth | tabela | radius | 16 kB |
radius=> SELECT * FROM radpostauth;
```