# IPSec VPN

This class is about using IPSec to set up a _virtual private network_.

> https://www.youtube.com/watch?v=xTH1ZA_qUvA

> [!IMPORTANT]
> **Co nas dzisiaj czeka?**
> — stoi przed nami przygotowanie bezpiecznego tunelu między komputerami
> wykorzystując protokół (a właściwie zestaw protokołów pod nazwą) IPsec.
> Jest to oczywiście rozszerzenie bezpieczeństwa dla protokołu IP.

## Listening in

A quick reminder, before we begin.

You can use either _Wireshark_ or `tcpdump` to listen in on network traffic.
Let's start with simple ICMP traffic as sent by ping. 
If you choose to use `tcpdump` you can do it as follows:

```console
# tcpdump -nAX -i br0 icmp
```

## IPSec Tooling

Check if you've got StrongSWAN installed:
```console
# zypper se strongswan
# zypper in strongswan
```

## Krok 1. IPsec host-to-host PSK

Skonfigurujemy połączenie między pierwszymi dwoma komputerami korzystając 
z predefiniowanego klucza.  

> [!TIP]
> Poniżej są przykłady takiej konfiguracji dla komputerów o adresach
> 150.254.32.75 oraz 150.254.32.76. Jak się domyślacie, 
> należy je dostosować do _waszych_ adresów IP.

Pełne pliki znajdziecie [na stronie opiekuna przedmiotu](https://www.cs.put.poznan.pl/mszychowiak/security/tools/lab_IPsec/).

```bash
# /etc/ipsec.conf:
conn host-to-host
    ikelifetime=60m
    keylife=20m
    rekeymargin=3m
    keyingtries=1
    left=150.254.32.75
    right=150.254.32.76    
    type=transport
    authby=psk
    auto=route
```

```bash
# /etc/ipsec.secrets:
# istotna jest spacja po dwukropku !
: PSK "B$R%lab!" 
```

### Przydatne polecenia
```bash
ipsec start
ipsec status
ipsec statusall

ipsec listall
ip -s xfrm policy
ip -s xfrm state

ipsec down
ipsec stop
ipsec restart

tcpdump -n -i br0 udp port isakmp or esp or icmp
tcpdump -n -i br0 net 150.254.32
tcpdump -n -i br0 host 150.254.32.76 and \( udp port isakmp or esp or icmp \)

ipsec status
ipsec statusall
```

## Krok 2. Zbieranie logów
```bash
# /etc/strongswan.conf:
charon {
    ...
    filelog {
        charon {
            path = /var/log/ike.log
            time_format = %b %e %T
            ike_name = yes
            append = no
            default = 2
            flush_line = yes
        }
    }
    ...
```

## Krok 3. Dodanie kolejnego komputera

```shell
# /etc/ipsec.conf
conn host-to-host
    ikelifetime=60m
    keylife=20m
    rekeymargin=3m
    keyingtries=1

conn test-vpn
    also=host-to-host
    left=...
    ...
```

## Krok 4. Generic parameters
Przykład:
```bash
...
conn test-vpn
    ...
    left=150.254.32.0/24
    right=%any
    ...
```

## Krok 5. Pod-sieci
```bash
...
conn test-vpn
    ...
    rightsubnet=150.254.32.0/24
    ...
```

## Krok 6. Tylko konkretne protokoły
```bash
...
conn test-vpn
    ...    
    rightsubnet=150.254.32.0/24[icmp]
    ...
```

## Krok 7. Polityka passthrough
```bash
...
conn pass-lo
    left=127.0.0.1
    right=%any
    type=passthrough
    auto=add
```

Poniżej również przykład dla DNS:
```bash
# passthrough policy for DNS servers:
conn pass-servers
    left=150.254.32.0/24
    rightsubnet=150.254.5.0/24[udp/53]
    type=passthrough
    auto=route
```

> [!IMPORTANT]
> Dodajcie politykę passthrough dla połączeń SSH.
