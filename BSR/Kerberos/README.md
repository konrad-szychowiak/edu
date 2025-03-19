# Kerberos

Kerberos to rozproszony system kryptograficznego uwierzytelniania z zaufaną stroną trzecią. Rolę tej strony,
której ufają wszystkie podmioty systemu (principals), pełni serwer KDC (Key Distribution Center) realizujący
usługi uwierzytelniania oraz – potencjalnie (choć w praktyce rzadko w pełni tego znaczenia) – autoryzacji. System
używa centralnego rejestru danych uwierzytelniających (credentials). Podmioty (użytkownicy, komputery,
aplikacje) otrzymują z centrum KDC bilet (ticket) zawierający m.in. klucz sesji (session key) wykorzystywany
w dostępie do poszczególnych usług, np. zdalnego logowania, drukowania itd. Każdy komputer kliencki i każdy
użytkownik w domenie uwierzytelniania (realm) musi zostać do niej indywidualnie dołączony.

## Ćwiczenia

W tym ćwiczeniu będziemy pracować w grupach minimum 3 osób. Role będą następujące:

* **KDC** (ang. _key distribution center_) – serwer Kerberosa pod adresem, `kdc.lan.lab`,
* **Server** – serwer usługi sieciowej, do której będziemy chcieli uzyskać dostęp, `server.lan.lab`, oraz
* **Client** – maszyna klienta, z której będziemy próbować się dostać do usługi, `client.lan.lab`.

> [!TIP]
> Jeśli w grupie wypadnie więcej osób, to kolejne osoby powinny przygotować kolejne serwery **usług**.

Pierwszym krokiem na każdej maszynie będzie sprawdzenie, czy zainstalowane są wymagane pakiety kerberosa:

```sh
# warto wykonać polecenia jako root, żeby zaktualizować od razu indeksy pakietów
student> sudo -i

# sprawdzamy, co już jest zainstalowane
root# zypper se krb5

#instalujemy pakiety (jeśli jakiś brakuje)
root# zypper in krb5 krb5-server krb5-client
```

## KDC

Trzeba przygotować konfigurację kerberosa w pliku `/etc/krb5.conf`:

```toml
[libdefaults]
    default_realm = LAB.LAN
    dns_lookup_kdc = false
    dns_lookup_realm = false
[realms]
    LAB.LAN = {
        kdc = kdc.lab.lan
        admin_server = kdc.lab.lan
    }
[domain_realm]
    .lab.lan = LAB.LAN
```

