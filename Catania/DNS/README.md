# DNS Security

* What is DNS? What problem does it solve?
* What is DNSsec? What problem does it solve?
* What is TSIG? What problem does it solve?

## Introduction

For this class we will be using a Virtual Machine from PUT, where we need to install the tools we will be using.

> [!TIP]
> Some useful tools to query name servers:
> * `nslookup` – should be available everywhere
> * `host`
> * `whois` – you might need to install it manually
>
> Use the manual or help for the commands before you run them.

Look up information about the following domains:

* facebook.pl
* gov.pl

You should find two things:

1. Can you tell if either of them uses DNSsec?
2. Can you observe something interesting in facebook's addresses?

> [!NOTE]
> The command `host` can be used to access more than just the addresses.
> Use `-a` to get the whole output or `-t` to get a specific type of record.
> Try:
> ```shell
> host -t mx megacorpone.com
> ```

### Installation

We need to install the Bind9 server we will use as our name server:
```console
student@zsk:~> sudo -i
zsk:~# zypper se whois
zsk:~# zypper in bind
Loading repository data...
Reading installed packages...
Resolving package dependencies...

The following package is going to be upgraded:
  bind-utils

The following 2 NEW packages are going to be installed:
  NetworkManager-dns-bind bind

1 package to upgrade, 2 new.

Package download size:     1.8 MiB

Package install size change:
              |       5.2 MiB  required by packages that will be installed
   775.1 KiB  |  -    4.4 MiB  released by packages that will be removed

Backend:  classic_rpmtrans
Continue? [y/n/v/...? shows all options] (y):
```

Type `y` to confirm and wait for the installation to complete.

> [!TIP]
> If you want to use `whois` later, repeat the step for a package called `whois`.


### Configuration files

* in `/etc/named.conf` find these lines and uncomment the _include_-expression:
    ```diff
      # Un-comment the following if you still need "/etc/named.conf.include" included.
    - # include "/etc/named.conf.include";
    + include "/etc/named.conf.include";
    ```
* download the include file from [here](https://www.cs.put.poznan.pl/mszychowiak/security/tools/lab_DNS/named.conf.include):
    ```console
    zsk:~ # wget -O /etc/named.conf.include https://www.cs.put.poznan.pl/mszychowiak/security/tools/lab_DNS/named.conf.include
    ```
* download the zone file from [here](https://www.cs.put.poznan.pl/mszychowiak/security/tools/lab_DNS/lab.net.zone):
    ```console
    zsk:~ # wget -O /var/lib/named/lab.net.zone https://www.cs.put.poznan.pl/mszychowiak/security/tools/lab_DNS/lab.net.zone
    ```


Your own DNS
------------
We will set up our own DNS service using the master-slave architecture.
Each service needs a master and at least one slave.

<!-- 2. Użyjemy servera bind? See: [bind in docker](https://blog.welpnetwork.com/posts/docker/bind9/) -->

**Split into pairs.** Decide which one of you will be the _master_ and who will be the _slave_.

> [!TIP]
> I recommend setting your `hostname` to `master` or `slave`, respectively.
> Remember to reload the terminal, or activate a new one, for the change to take hold.

Once you decide who will take which role, add a new ip address to your machine, using the following pattern:

```sh
ip addr add 10.X.1.Y/16 dev Z
#              │   │        └─> interface on which the computer is connected to the internet
#              │   └──────────> 1 for master, 2 for slave
#              └──────────────> the same # as lab-sec-# of master
```

---

<details>
<summary><i>Example: IP addresses</i></summary>

> [!WARNING]
> The example below assumes the master is `lab-sec-0`,
> and both the _master_ and the _slave_ are connected to the lab network on interface `br0`.
> **Modify it accordingly!**

```console
master:~# ip a add 10.0.1.1/16 dev br0
```

```console
slave:~# ip a add 10.0.1.2/16 dev br0
```

</details>

---

### Bind9

We will be using a self-hosted name server called `bind9` for our exercises.

First, see if bind is installed in your system:

```shell
zypper se bind
```

If it's not, install it.
If it is—or once you've installed it—we can start the configuration process.

In `/etc/named.conf` check the existing configuration.
Set these two options (they might be commented and/or have different values):

```
dnssec-validation auto;
```

and

```
include "/etc/named.conf.include";
```

Once ready, download `/etc/named.conf.include` and `/var/lib/named/lab.net.zone` from course resources.

You need to modify `/var/lib/named/lab.net.zone`
to include the correct addresses of `dns1` (_master_) and `dns2` (_slave_),
according to your group's addresses set a moment ago.

> [!NOTE]
> Like we've already done before, you can set the configuration on one machine and copy it to the other.
> Please copy only those two files to avoid errors. If you do not share the file but modify it manually,
> verify that you have got the same configuration on both machines.

We should be able to start the name server daemon now. I suggest:

```shell
service named start
```

Finally, let's check if it works. First step will be to check your local name service:

```shell
dig @localhost mail.lab.net
```

**Then, you can check both your and your groupmate's services.**

---

<details>
<summary><i>Example: verify the first name services</i></summary>

> [!WARNING]
> The example below assumes the master is `lab-sec-0`. **Modify it accordingly!**

Both the machines should be able to execute:

```shell
dig @10.0.1.1 mail.lab.net
dig @10.0.1.2 mail.lab.net
```

The results should be matching for both.
</details>

---

### Setting up `slave`

For now, both machines run the same configuration as `type master` (see `/etc/named.conf.include`).
Now, we want the assigned _slave_ to actually listen to the _master_ service
(meaning: the _slave_ will synchronise with the _master_, whenever the _master_ has a newer zone definitions).

> [!IMPORTANT]
> Remember, for the _slaves_ to download newer zone definitions,
> the sequence number of that definition must be incremented (think: versioning).
> Otherwise the change will not be reflected and applied across the system.

**Modify the configuration of the _slave_** to reflect their role: change the `type` to `slave` and point to the
`master server`.


> [!WARNING]
> The example below assumes the master is `lab-sec-0`. **Modify it accordingly!**

```
# /etc/named.conf.include
zone "lab.net" in {
      type slave;
      masters { 10.0.1.1; };
      file "lab.net.zone";
      notify master-only;
};
```

Restart the slave's `named` service.

How do we check if it works correctly?
We modify the zone definition on the master's side.
If the slave replicates the change, it means the system is functioning correctly.

> [!TIP]
> You can modify the zone definition by incrementing both the `serial` and the last byte of `mail.lab.net`'s IP.
> **Remember to reload the master's `named` service afterward.**

---

<details>
<summary><i>Example: testing master-slave relationship</i></summary>

> [!WARNING]
> The example below assumes the master is `lab-sec-0`._ **Modify it accordingly!**

After modifying the zone definition and reloading the name daemon (both on _master_),
querying both dns servers must return the same information:

```shell
dig @10.0.1.1 mail.lab.net
dig @10.0.1.2 mail.lab.net
```

</details>

---

### Transaction Signature

#### On Master

Generate a key for TSIG:

```console
master:~# tsig-keygen -a hmac-sha512 transfer_key > /var/lib/named/transfer.key
```

**Copy the key to the slave (into the same location).**

Include the key file into your configuration file (the include file)
and allow transfer for our zone based on that key.

```diff
  # /etc/named.conf.include
+ include "/var/lib/named/transfer.key";

  zone "lab.net" in {
      type master;
      file "lab.net.zone";
      notify master-only;
+     allow-transfer { key transfer_key; };
  };
```

#### On Slave

Wait for the master to send you a key file into `/var/lib/named/transfer.key`
or secure-copy it from the master machine (agree on which to do with your groupmate).

Modify your config to include the key and assign the key to the `master_server`.

> [!WARNING]
> The example below assumes the master is `lab-sec-0`. **Modify it accordingly!**

```diff
 # /etc/named.conf.include
+ include "/var/lib/named/transfer.key";

+ masters master_server { 10.0.1.1 key transfer_key; };

  zone "lab.net" in {
        type slave;
-       masters { 10.0.1.1; };
+       masters { master_server; };
        file "lab.net.zone";
        notify master-only;
  };
```

#### On Both

Verify that the slave still reflects the changes from the master after the key is added, using:

```shell
dig @<dns_server> lab.net AXFR -k <transfer_key_file>
```

### Zone Signing

DNSsec introduces a minor complication in domain management, namely that any change to the zone file requires the zone
to be re-signed. The key (or keys) used for signing should be changed periodically (depending on the length of the key).
Unfortunately, any change to a key requires its signature to be sent to the parent DNS server. We can avoid this by
using two DNSKEY keys:

- ZSK (Zone-Signing Key) – we sign the entire zone with it; it is a shorter key that we can change frequently,
- KSK (Key-Signing Key) – this is used to sign DNSKEY records (ZSK keys) and its signature is passed on to the parent
  DNS server (→ DS record); it is less frequently changed.
  In this way, we can change the ZSK frequently and do not need to notify the master server.

However, if we need to change the KSK (when, for example, it is already nearing the end of its life) then:

1. we generate a new Key-Signing Key - KSK2,
2. we sign the ZSK with both keys (KSK and KSK2),
3. we inform the master server about this and ask it to update the DS record,
4. we wait until the DS record has been refreshed by all servers (i.e. we wait at least for the TTL lifetime of the
   entries on the master server, it is recommended to wait double the TTL time),
5. we sign the ZSK again, now only with the new KSK2.

> [!WARNING]
> The examples below include _exemplary_ values, especially in command outputs,
> as many are not idempotent.
> **Modify them accordingly!**

#### Master Server

> [!CAUTION]
> The keys must be created using `-L` for ttl.

Create ZSK:

```console
master:/var/lib/named/# dnssec-keygen -a RSASHA512 -b 2048 -n ZONE lab.net
Klab.net.+005+44901
```

Create KSK:

```console
master:/var/lib/named/# dnssec-keygen -f KSK -a RSASHA512 -b 2048 -n ZONE lab.net
Klab.net.+005+33553
```

Include the keys in your zone file (here lab.net.zone):
```diff
+ $INCLUDE /var/lib/named/Klab.net.+005+33553.key
+ $INCLUDE /var/lib/named/Klab.net.+005+44901.key
```

Manually sign your zone (`-k` specifies the KSK, don’t mismatch your keys!):

```console
master:/var/lib/named/# dnssec-signzone -o lab.net -k Klab.net.+005+33553 lab.net.zone Klab.net.+005+44901
zone_file_lab.net.signed
```

Replace the zone filename in your DNS server configuration:
```diff
  # /etc/named.conf.include
  include "/var/lib/named/transfer.key";

  zone "lab.net" in {
      type master;
-     file "lab.net.zone";
+     file "/var/lib/named/lab.net.zone.signed";
      notify master-only;
  };
```

And then check it:
```console
student@master:~$ dig @<dns_server> DNSKEY lab.net
; <<>> DiG 9.3.2 <<>> @serwer_dns DNSKEY lab.net
;; global options:  printcmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 35590
;; flags: aa rd ra ad; QUERY: 1, ANSWER: 2, AUTHORITY: 2, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags: do; udp: 4096

; QUESTION SECTION:
;lab.net.                       IN      DNSKEY

;; ANSWER SECTION:
lab.net.                86400   IN      DNSKEY  257 3 10 AwEAAdIXI2y/oSN8aPMYt6QcnK1G6X3wljFgA0N+0NcNX1eGxSMHbhNX ddvu5BbwPsgaTxsednnK/dheSilybGj9pEdBlGLeGTJfWwk6/Bvp74ny 7CfjVon3J46MuuxFV/FTnUvZC/eeoWx4Cf5SHlGeaXWCDShLnAnjK8dC JootXhKcIIchMxMLwbqmUEbvIDA1Y4jFvMFXvWLgIrFVQMes/AD4q1UR +ljB7ldepAphEB6rvmM8UcrJhE/y5X8Uez7noDJ7PWqlz4JmouFCYsup KT1xZ/id5Cc=
lab.net.                86400   IN      DNSKEY  256 3 10 AwEAAbFmIbrFFSzSuvsB2dxTLFXfZCJCEpJDpY74kCzhATU/zjJs/MjI R0c1WWscxmC8e3llkNfZsnaxQpV1Ycbget3myvCkkErgyGUkY6a4gvzI aHAr7HAzmZzVfTpoCZahUtvZcJd5rLjiyTSLyAeDzZjKA+EHOS+l9se5 nGiOMVe3PXbo0M8y0mwUYTOnTmC6oqi79uNslMUcv/P32F/sBAoABTb2 RmTHDnROC8l2GxdlHYXUt4ZT0ZVC1ZonEcMEDWo8v+qv0ZZ/KEaZnLPp gNoYlj47XXU=

student@master:~$ delv @<dns_server> -a <key-file> SOA +multiline +root=lab.net lab.net. 
; fully validated
lab.net.   86400 IN SOA dns1.lab.net. hostmaster.lab.net. (
				2014112007 ; serial
				1800       ; refresh (30 minutes)
				900        ; retry (15 minutes)
				2419200    ; expire (4 weeks)
				300        ; minimum (5 minutes)
				)
lab.net.   86400 IN RRSIG SOA 10 2 600 (
				20150107091559 20141208081559 17694 lab.net.
				LwG0rLOm9Q1Lu9bgIz1O+PTCwcCSs1Ev8Eqkqqd3gUJK
				qo0FSVv//axNVJFH2Lz8VLgFypD8xARWj1XQBD/9DIf6
				A3ncnrFymKdKze+2ghvTUxpwqctK4RF66mhu93e33+Ir
				QJVJgdHJVHudQWICA1AbIBYYzLGkKlp7JAJcgBM= )
```