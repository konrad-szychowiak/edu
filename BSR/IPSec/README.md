# IPSec VPN

This class is about using IPSec to set up a _virtual private network_.

First, a few questions:
1. IPSec?
2. IKE?

## Listening in

A quick reminder, before we begin.

You can use either _Wireshark_ or `tcpdump` to listen in on network traffic.
Let's start with simple ICMP traffic as sent by ping. 
If you choose to use `tcpdump` you can do it as follows:

```console
lab-sec-0:~# tcpdump -nAX -i br0 icmp
```

## IPSec Tooling

Check if you've got StrongSWAN installed:
```console
lab-sec-0:~# zypper se strongswan
lab-sec-0:~# zypper in strongswan
```

* `/etc/ipsec.conf`
* `/etc/ipsec.secrets`
* `/etc/strongswan.conf`

